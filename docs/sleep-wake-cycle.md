# Sleep/Wake Cycle

> Code-level walkthrough of going to deep sleep and waking up. Order matters.

## Going to Sleep (Order Matters)

```c
void enterDeepSleep(int seconds) {
    // 1. Stop audio peripheral cleanly
    i2s_driver_uninstall(I2S_NUM_0);

    // 2. Set GPIO0 (I2S LRC) HIGH for boot safety, hold through sleep
    pinMode(0, OUTPUT);
    digitalWrite(0, HIGH);
    gpio_hold_en((gpio_num_t)0);          // GPIO0 IS RTC -- hold persists

    // 3. Backlight LOW. Pull-down resistor ensures it stays LOW even though
    //    GPIO21 isn't an RTC GPIO and gpio_hold won't persist for it
    digitalWrite(21, LOW);

    // 4. Display sleep mode -- turns off display controller
    sendDisplayCommand(0x10);              // ILI9488 "Sleep In" (CS is hardwired LOW)

    // 5. Park display DC HIGH (data mode default), hold through sleep
    digitalWrite(32, HIGH);
    gpio_hold_en((gpio_num_t)32);          // GPIO32 IS RTC -- hold persists

    // 6. Modem DTR sleep -- modem keeps network registration but naps
    digitalWrite(MODEM_DTR_PIN, HIGH);     // GPIO25
    gpio_hold_en((gpio_num_t)MODEM_DTR_PIN);

    // 7. Hold critical control pins
    gpio_hold_en((gpio_num_t)BOARD_POWERON_PIN);  // GPIO12 stays HIGH
    gpio_hold_en((gpio_num_t)MODEM_RESET_PIN);    // GPIO5 stays LOW (don't reset)
    gpio_deep_sleep_hold_en();             // persist all RTC holds through deep sleep

    // 8. Configure timer wakeup
    esp_sleep_enable_timer_wakeup((uint64_t)seconds * 1000000ULL);

    // 9. Goodbye
    esp_deep_sleep_start();
}
```

## Waking Up (Boot Sequence)

```c
void setup() {
    Serial.begin(115200);

    // 1. CRITICAL FIRST: set power pin HIGH
    //    On battery, if BOARD_POWERON_PIN floats, the board resets in a loop
    pinMode(BOARD_POWERON_PIN, OUTPUT);
    digitalWrite(BOARD_POWERON_PIN, HIGH);

    // 2. Release all GPIO holds from previous sleep
    gpio_hold_dis((gpio_num_t)MODEM_DTR_PIN);
    gpio_hold_dis((gpio_num_t)BOARD_POWERON_PIN);
    gpio_hold_dis((gpio_num_t)MODEM_RESET_PIN);
    gpio_hold_dis((gpio_num_t)0);
    gpio_hold_dis((gpio_num_t)32);     // display DC

    // 3. Init UART to modem
    SerialAT.begin(115200, SERIAL_8N1, MODEM_RX_PIN, MODEM_TX_PIN);

    // 4. Wake modem from DTR sleep
    pinMode(MODEM_DTR_PIN, OUTPUT);
    digitalWrite(MODEM_DTR_PIN, LOW);
    delay(2000);                           // give modem time to wake
    modem.sleepEnable(false);

    // 5. Verify modem is responsive
    while (!modem.testAT(1000)) {
        Serial.print(".");
    }

    // 6. Check if 4G data is available
    if (!modem.isGprsConnected()) {
        modem.gprsConnect("internet", "", "");  // APN depends on operator
    }

    // 7. Poll the server
    String response = httpGet("https://yourserver.com/api/device/" + deviceId + "/pending");

    if (response.contains("\"pending\":true")) {

        // 8. Download content to SD card
        downloadContent(response);

        // 9. Wake display
        sendDisplayCommand(0x11);          // ILI9488 "Sleep Out"
        delay(120);                        // controller needs 120ms after wake
        pinMode(21, OUTPUT);
        digitalWrite(21, HIGH);            // backlight ON

        // 10. Init I2S audio
        i2s_config_t cfg = { ... };
        i2s_driver_install(I2S_NUM_0, &cfg, 0, NULL);
        i2s_pin_config_t pins = {
            .bck_io_num = 19,
            .ws_io_num = 0,
            .data_out_num = 22,
            .data_in_num = -1
        };
        i2s_set_pin(I2S_NUM_0, &pins);

        // 11. Play content (pixel art, TTS audio, photos, video)
        playAllContent();

        // 12. Check if GPS was requested
        if (response.contains("\"gps_requested\":true")) {
            modem.enableGPS();
            // Wait up to 2 minutes for fix
            float lat, lon;
            if (modem.getGPS(&lat, &lon)) {
                httpPost("/api/device/" + deviceId + "/location",
                         {"lat": lat, "lon": lon});
            }
            modem.disableGPS();
        }

        // 13. Report battery level
        float voltage = readBatteryVoltage();   // GPIO35 ADC
        int percent = mapVoltageToPercent(voltage);
        httpPost("/api/device/" + deviceId + "/status", {"battery": percent});
    }

    // 14. Battery protection check
    float voltage = readBatteryVoltage();
    if (voltage < 3.3) {
        // Critically low -- show "I'm dying" briefly if power allows, then sleep long
        showLowBatteryAnimation();
        enterDeepSleep(3600);              // wait 1 hour, hope she charges
    }

    // 15. Back to deep sleep
    enterDeepSleep(300);                   // 5 minutes
}

void loop() {
    // Never reached -- everything happens in setup() then deep sleep
}
```

## Why setup() and Not loop()

ESP32 deep sleep is essentially a reset. After waking, execution starts from the top of `setup()`, not where `loop()` left off. Therefore all logic lives in `setup()`, and `loop()` is unused.

## Initial Cold Boot (First Power-On)

The first time the device is powered (after she opens the package), there's no prior sleep state. The ESP32 boots normally and:

1. Sets BOARD_POWERON_PIN HIGH
2. Resets the modem (PWRKEY pulse)
3. Waits for "SMS DONE" (network registration complete, ~10–30 seconds)
4. **Plays the "first boot greeting"** — a special pre-loaded animation + TTS message that introduces the character ("Hello! I'm [name]!" or similar)
5. Polls server immediately for any waiting content
6. Goes to sleep, polling cycle begins

The first-boot greeting can be hardcoded in firmware or downloaded on first poll.

## Related pages

- [Pin assignment](pin-assignment.md) — why GPIO0 and GPIO21 need the special handling above
- [Power and battery life](power-and-battery-life.md) — current cost of each step
- [Behavior states](behavior-states.md) — high-level state diagram
- [Firmware patterns](firmware-patterns.md) — SPI / I2S / battery / display init code
- [Server API](server-api.md) — what `httpGet` and `httpPost` are talking to
- [Gotchas](gotchas.md) — `BOARD_POWERON_PIN` reset loop, display 120ms wait
