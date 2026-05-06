# Firmware Patterns

> Reusable code snippets for SPI, I2S, modem, display, battery, and LiPo protection.

## SPI Bus Configuration (Two Separate Instances)

The SD card and display BOTH need SPI. They use different buses on the ESP32:

```c
// SD card on HSPI (default SPI object)
SPI.begin(BOARD_SCK_PIN, BOARD_MISO_PIN, BOARD_MOSI_PIN);  // 14, 2, 15
if (!SD.begin(BOARD_SD_CS_PIN)) {                           // 13
    Serial.println("SD card failed");
}

// Display on VSPI (separate instance)
SPIClass *displaySPI = new SPIClass(VSPI);
displaySPI->begin(18, -1, 23, -1);                          // SCK, MISO, MOSI, CS
// We pass -1 for MISO (display is write-only) and -1 for CS (we manage CS manually)

// Initialize display driver with the VSPI instance
TFT_eSPI tft = TFT_eSPI();  // configure User_Setup.h to use VSPI pins
// or use Adafruit_ILI9488 with custom SPI: tft.begin(displaySPI);
```

## I2S Setup for MAX98357A

```c
i2s_config_t i2s_config = {
    .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_TX),
    .sample_rate = 16000,                       // TTS quality
    .bits_per_sample = I2S_BITS_PER_SAMPLE_16BIT,
    .channel_format = I2S_CHANNEL_FMT_ONLY_LEFT,  // mono
    .communication_format = I2S_COMM_FORMAT_I2S_MSB,
    .intr_alloc_flags = ESP_INTR_FLAG_LEVEL1,
    .dma_buf_count = 8,
    .dma_buf_len = 64,
    .use_apll = false
};

i2s_pin_config_t pin_config = {
    .bck_io_num = 19,    // BCLK
    .ws_io_num  = 0,     // LRC (this is GPIO0, careful with deep sleep)
    .data_out_num = 22,  // DIN
    .data_in_num = -1
};

i2s_driver_install(I2S_NUM_0, &i2s_config, 0, NULL);
i2s_set_pin(I2S_NUM_0, &pin_config);
```

## Battery Monitoring (GPIO35 ADC)

The LILYGO board has a voltage divider on GPIO35 that scales the battery voltage to fit the ESP32 ADC range (0–3.3V). Typical divider: 100K + 100K (ratio 1:2).

```c
// GPIO35 reads half the actual battery voltage
analogReadResolution(12);                       // 0-4095
analogSetPinAttenuation(35, ADC_11db);          // full 0-3.3V range

int raw = analogRead(35);
float adc_voltage = (raw / 4095.0) * 3.3;
float battery_voltage = adc_voltage * 2.0;      // multiply by divider ratio

int percent = constrain(map(battery_voltage * 100, 320, 420, 0, 100), 0, 100);
// 3.20V = 0%, 4.20V = 100%
```

## Modem AT Command Patterns

```c
// Wake from DTR sleep
gpio_hold_dis((gpio_num_t)MODEM_DTR_PIN);
digitalWrite(MODEM_DTR_PIN, LOW);
delay(2000);
modem.sleepEnable(false);

// Connect to 4G data
if (!modem.isGprsConnected()) {
    modem.gprsConnect(APN, USERNAME, PASSWORD);  // APN depends on operator
}

// Get GPS fix (built-in GPS via modem)
modem.enableGPS();
float lat, lon, speed, alt, accuracy;
int year, month, day, hour, min, sec;
uint8_t fixMode;
int vsat, usat;
while (!modem.getGPS(&fixMode, &lat, &lon, &speed, &alt, &vsat, &usat,
                      &accuracy, &year, &month, &day, &hour, &min, &sec)) {
    delay(5000);  // wait for fix
}
modem.disableGPS();

// HTTP request via modem
TinyGsmClient client(modem);
HttpClient http(client, "yourserver.com", 443);
http.get("/api/device/abc/pending");
int statusCode = http.responseStatusCode();
String response = http.responseBody();
http.stop();

// Back to DTR sleep
digitalWrite(MODEM_DTR_PIN, HIGH);
gpio_hold_en((gpio_num_t)MODEM_DTR_PIN);
modem.sleepEnable(true);
```

## Display Sleep/Wake (ILI9488)

```c
// Send a command to ILI9488 (4-wire SPI mode)
void sendDisplayCommand(uint8_t cmd) {
    digitalWrite(16, LOW);                   // DC LOW = command mode
    digitalWrite(17, LOW);                   // CS LOW = select chip
    displaySPI->transfer(cmd);
    digitalWrite(17, HIGH);                  // CS HIGH = release
    digitalWrite(16, HIGH);                  // DC back to data mode
}

// Sleep In (before deep sleep)
sendDisplayCommand(0x10);
delay(5);                                    // allow controller to settle

// Sleep Out (after wake)
sendDisplayCommand(0x11);
delay(120);                                  // controller needs 120ms before drawing
```

## LiPo Over-Discharge Protection (Critical)

LiPo cells without protection can be permanently damaged below ~3.0V. Even though the LILYGO board may have basic protection, we should be conservative in firmware:

```c
const float CRITICAL_VOLTAGE = 3.3;          // emergency cutoff
const float LOW_VOLTAGE = 3.5;               // start showing "feed me"
const float SLEEPY_VOLTAGE = 3.7;            // start "sleepy" animations

float voltage = readBatteryVoltage();

if (voltage < CRITICAL_VOLTAGE) {
    // Skip everything, just sleep long to reduce drain
    enterDeepSleep(3600);                    // 1 hour
}
else if (voltage < LOW_VOLTAGE) {
    showAnimation("feed_me");
}
else if (voltage < SLEEPY_VOLTAGE) {
    showAnimation("sleepy");
}
```

## Related pages

- [Sleep/wake cycle](sleep-wake-cycle.md) — full setup() flow that uses these snippets
- [Pin assignment](pin-assignment.md) — pin numbers above are board-verified
- [LILYGO examples reference](lilygo-examples.md) — official examples to crib from
- [Gotchas](gotchas.md) — pitfalls in SPI init, display sleep timing, GPIO0 boot
