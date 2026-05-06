# Project: "Cute ESP" -- Karim's Mystery Love Device

**Date:** April 20, 2026
**Built by:** MHD (developer) for Karim (the romantic)
**Status:** Final plan, ready to build

---

## The Story

Karim wants to marry his dream girl. Her parents approved, but she hasn't met him and
isn't convinced yet. The plan: gift her a mysterious device with a cute pixel art
character that Karim secretly controls from his phone. Over time, the character charms
her -- talking, showing emotions, sending messages, photos, music, videos. She doesn't
know who's behind it. The goal: she goes to her dad and says "I'm interested."

---

## How It Works (Big Picture)

```
Karim's Phone (React Native Expo app)
        |
        v
   Remote Server (REST API)
        |
        v
   Server queues content, waits for device to poll
        |
        v
   ESP32 wakes every 5 min, checks server via HTTP over 4G
        |
        v
   New content? Download to SD card, display + play
        |
        v
   Done? Modem naps (DTR sleep), ESP32 deep sleeps for 5 min
```

**No SMS gateway. No SMS at all. Just a simple REST API.**
The device polls the server every 5 minutes over 4G HTTP.
Between polls, the modem naps (DTR sleep, ~3mA) and the ESP32 deep sleeps (~0.157mA).

---

## The Device Behavior

### Default State: DEEP SLEEP + DTR NAP
- ESP32 in deep sleep (~0.157mA)
- A7670 modem in DTR sleep (~3mA) -- still registered to network, UART off
- Display OFF, backlight OFF (GPIO21 LOW, held by 10K pull-down resistor)
- Speaker OFF (MAX98357A auto-shuts down when no I2S clock)
- GPS OFF
- Total sleep current: **~3.2mA**

### Every 5 Minutes (Poll Cycle -- no new content):
1. ESP32 wakes from timer
2. Sets `BOARD_POWERON_PIN` (GPIO12) HIGH
3. Releases DTR hold, pulls DTR (GPIO25) LOW -- modem wakes from nap (~2 sec)
4. HTTP GET to server: `GET /api/device/{id}/pending`
5. Server responds: `{"pending": false}`
6. Pulls DTR HIGH -- modem goes back to nap
7. ESP32 enters deep sleep for another 5 minutes
8. **Total time awake: ~5 seconds. Cost: ~0.28mAh.**

### Every 5 Minutes (Poll Cycle -- NEW CONTENT):
1. ESP32 wakes from timer
2. Wakes modem from DTR nap (~2 sec)
3. HTTP GET to server: `GET /api/device/{id}/pending`
4. Server responds with content queue (could be multiple messages)
5. Downloads media files (TTS audio, photos, videos) to **SD card**
6. Sends display wake command (0x11), turns on backlight (GPIO21 HIGH)
7. Pixel art character appears, delivers each message in sequence:
   - Text: character speaks via TTS audio through speaker
   - Photo: displayed on screen with character reaction
   - Video: played from SD card with audio
   - Animation: character does gestures/emotions
8. After all content played: sends display sleep command (0x10)
9. Turns off backlight (GPIO21 LOW)
10. Reports battery level to server: `POST /api/device/{id}/status`
11. Pulls DTR HIGH -- modem naps
12. ESP32 deep sleeps for 5 minutes

### GPS Location (On-Demand, during any poll):
1. During normal poll, server response includes `"gps_requested": true`
2. ESP32 enables GPS via AT command: `AT+CGPS=1`
3. Waits for fix (30 sec to 2 min): `modem.getGPS(...)`
4. Sends coordinates to server: `POST /api/device/{id}/location`
5. Disables GPS: `AT+CGPS=0`
6. Continues normal sleep cycle
7. **Official example code exists:** `docs/examples/WakeupByRingOrSMS_SendLocation/`

### First Boot Experience:
- Device powers on for the first time
- Character appears, greets her with a cute animation + TTS message
- Then display turns off
- She's left wondering "what was that?"
- 5 minutes later (or whenever Karim sends the first message), it comes alive again

### Low Battery Behavior:
- Firmware monitors battery via GPIO35 ADC every poll cycle
- Reports level to server (Karim sees it in his app)
- Below ~20%: character shows "I'm sleepy" animation, yawns
- Below ~10%: character says "feed me" (charge me)
- Below 3.3V: forced deep sleep, no more polling, protects LiPo from damage
- She plugs in USB-C to charge, device wakes up and character shows happy animation

---

## Communication Rules

- **One-way only**: device receives, never sends back to her
- She cannot respond through the device
- This continues **until she tells her dad she's interested in the marriage**
- Only then does two-way communication begin (future phase, not part of this build)

---

## Content Types

| Type | How It Works |
|------|-------------|
| **Text messages** | Character displays text on screen, reads it via TTS |
| **Animations with text** | Pixel art character does gestures/emotions + text overlay |
| **Audio** | TTS generated on server, sent as audio file, played via speaker |
| **Photos** | Downloaded over 4G, stored on SD card, displayed on screen |
| **Videos** | Downloaded over 4G, stored on SD card, played on screen with audio |
| **Music** | Audio files played through speaker |

### Text-to-Speech
- TTS is processed **on the server** (not on ESP32)
- Server generates audio file from text
- ESP32 downloads and plays the audio file through MAX98357A + speaker
- This gives much better voice quality than on-device TTS

---

## Physical Design

| Aspect | Decision |
|--------|----------|
| **Look** | DIY green PCB aesthetic, retro, raw electronic look |
| **Case** | Transparent cover (she can see all the components inside) |
| **Size** | Portable, small enough to carry around |
| **Display** | 4.0" IPS screen (only on when content is playing) |
| **Speaker** | Small mono speaker for TTS and audio |
| **Power** | 8000mAh LiPo pouch cell (PL985525), rechargeable via USB-C, always-on but sleeping |
| **Charging** | USB-C (she plugs it in like a phone) |

---

## Hardware Parts (All Purchased)

| # | Component | Price | Details |
|---|-----------|:-----:|---------|
| 1 | LILYGO T-SIM A7670E R2 | AED 141.70 | ESP32 + 4G LTE + Built-in GPS + SD card slot |
| 2 | SOTAMIA Mini Speakers (2pcs) | AED 9.46 | 4 Ohm, 3W (using 1 for mono, 1 spare) |
| 3 | MAX98357A I2S Amplifier (5pcs) | AED 3.75 | I2S Class D amp, 3W (using 1, rest are spares) |
| 4 | 4.0" IPS SPI Display | AED 55.92 | ILI9488 + FT6236 capacitive touch, 480x320, 14-pin |
| 5 | PL985525 LiPo Battery | AED 45.00 | 3.7V 8000mAh lithium polymer pouch cell |

**Total spent: ~AED 255.83**

### Additional Hardware Needed (from parts bin)
- 1x 10K resistor (for display RST circuit)
- 1x 100nF ceramic capacitor (for display RST circuit)
- 1x 10K resistor (pull-down on GPIO21/BL -- CRITICAL for sleep)
- 1x 1000-2200uF electrolytic capacitor (recommended, for modem power stability)

### Battery Mod
The stock 18650 holder will be **desoldered** from the LILYGO board. The PL985525 LiPo
pouch cell (8000mAh) is soldered to the B+ and B- pads via wires. Same 3.7V voltage,
the board's built-in USB-C charger works as-is. More than 2x the capacity of a standard
18650 (3500mAh), and flat form factor fits better in the transparent enclosure.

### Board Specs (LILYGO T-SIM A7670E R2)
- **MCU:** ESP32 (240MHz, 4MB Flash, 8MB PSRAM)
- **Cellular:** A7670E 4G LTE Cat1
- **GPS:** Built-in (BeiDou/GPS/GLONASS) -- accessed via modem AT commands, no extra pins
- **Connectivity:** Wi-Fi 802.11 b/g/n, Bluetooth v4.2
- **Storage:** SD card slot (built-in, HSPI bus)
- **Power:** 8000mAh LiPo (modded) + USB-C charging
- **Interfaces:** I2C, SPI, UART, SD, I2S, CAN

---

## Key Decisions Made

| Decision | Reasoning |
|----------|-----------|
| **Mono audio** | One speaker is enough for TTS and audio playback |
| **No touch** | She doesn't interact with the device, one-way only |
| **Timer polling (5 min)** | Simpler than SMS, no gateway costs, ~29 day battery life |
| **DTR sleep between polls** | Modem naps at ~3mA instead of full awake at ~20mA |
| **No SMS gateway** | Server is just a REST API, simpler and free |
| **TTS on server** | Much better audio quality than on-device TTS |
| **SD card for media** | ESP32 has limited RAM; download media to SD then play from it |
| **Deep sleep default** | Screen and everything off unless content arrives |
| **GPS on-demand only** | Server flags when GPS is needed, checked during normal poll |

---

## Pin Assignment (VERIFIED from official repo)

**Source:** `docs/examples/ATdebug/utilities.h` lines 88-145 (LILYGO_T_A7670 block)

### Internally Used by Board (DO NOT USE)

| GPIO | Used By | Notes |
|:----:|---------|-------|
| 4 | BOARD_PWRKEY_PIN | Pulse to power modem on/off |
| 5 | MODEM_RESET_PIN | Active HIGH reset (MODEM_RESET_LEVEL = HIGH) |
| 12 | BOARD_POWERON_PIN | **MUST be HIGH for battery power** |
| 25 | MODEM_DTR_PIN | HIGH = modem nap, LOW = modem wake |
| 26 | MODEM_TX_PIN | UART to modem |
| 27 | MODEM_RX_PIN | UART from modem |
| 33 | MODEM_RING_PIN | RI pin (not used for wake-up in timer mode, but reserved) |
| 35 | BOARD_BAT_ADC_PIN | Battery voltage sensing (input-only) |
| 36 | BOARD_SOLAR_ADC_PIN | Solar panel ADC, v1.4 only (input-only) |
| 2 | BOARD_MISO_PIN | SD card MISO |
| 13 | BOARD_SD_CS_PIN | SD card chip select |
| 14 | BOARD_SCK_PIN | SD card clock |
| 15 | BOARD_MOSI_PIN | SD card MOSI |

### Final GPIO Assignments (all 8 output-capable pins used)

| GPIO | Assigned To | Notes |
|:----:|------------|-------|
| 16 | Display DC | Data/Command select |
| 17 | Display CS | Chip select |
| 18 | Display SCK | VSPI hardware SPI clock |
| 23 | Display MOSI | VSPI hardware SPI data |
| 21 | **Display BL** | Backlight on/off + **10K pull-down to GND** |
| 19 | I2S BCLK | Audio bit clock |
| 0 | **I2S LRC/WS** | Audio word select (boot-sensitive -- see safety notes) |
| 22 | I2S DIN | Audio data |

### Hardwired (no GPIO needed)

| Signal | Wiring | Notes |
|--------|--------|-------|
| Display RST | 3.3V via 10K resistor + 100nF cap to GND | Auto power-on reset |
| MAX98357A SD | Tied to 3.3V | L+R mono mix, 15dB gain |

### Not Connected (touch -- skipped)

Display pins 9-14: MISO, T_CLK, T_CS, T_DIN, T_DO, T_IRQ -- all left unconnected.

### Free GPIOs (input-only, spare)

| GPIO | Notes |
|:----:|-------|
| 34 | Input-only, usable for button |
| 39 | Input-only, usable for sensor |

### GPIO0 Safety Notes (I2S LRC)

GPIO0 must be HIGH during boot or ESP32 enters flash mode. Safe because:

1. **During boot:** I2S not started, internal pull-up holds GPIO0 HIGH. Normal boot.
2. **During audio:** I2S toggles LRC. Boot is long done. No issue.
3. **Before deep sleep:** Firmware stops I2S, sets GPIO0 HIGH, holds it:
   ```c
   i2s_driver_uninstall(I2S_NUM_0);
   pinMode(0, OUTPUT);
   digitalWrite(0, HIGH);
   gpio_hold_en((gpio_num_t)0);    // GPIO0 IS an RTC GPIO -- hold persists
   ```
4. **Wake from sleep:** GPIO0 held HIGH. Normal boot. Firmware re-inits I2S.

### GPIO21 Backlight Note (NOT an RTC GPIO)

GPIO21 is NOT an RTC GPIO. `gpio_hold_en()` does NOT persist through deep sleep.
When ESP32 sleeps, GPIO21 floats. Without a fix, the backlight turns on (~100mA drain).

**Fix: 10K pull-down resistor** soldered between GPIO21 and GND.
- During sleep: GPIO21 floats, resistor pulls it LOW = backlight OFF
- During active: firmware drives GPIO21 HIGH, easily overrides the weak pull-down

---

## Complete Wiring Diagram

```
╔══════════════════════════════════════════════════════════════════════════╗
║                        FULL WIRING SCHEMATIC                           ║
║                     Cute ESP -- Karim's Love Device                    ║
╚══════════════════════════════════════════════════════════════════════════╝


    ┌─────────────────────────────────────────────────────────────────┐
    │                  LILYGO T-A7670E R2 (ESP32)                     │
    │                                                                 │
    │  Internal (DO NOT TOUCH):                                       │
    │    GPIO4  = PWRKEY        GPIO26 = MODEM TX                     │
    │    GPIO5  = MODEM RESET   GPIO27 = MODEM RX                     │
    │    GPIO12 = POWER ON      GPIO33 = RING (reserved)              │
    │    GPIO25 = MODEM DTR     GPIO35 = BATTERY ADC                  │
    │    GPIO2  = SD MISO       GPIO36 = SOLAR ADC                    │
    │    GPIO13 = SD CS         GPIO14 = SD CLK                       │
    │    GPIO15 = SD MOSI                                             │
    │                                                                 │
    │  Our pins:                                                      │
    │    GPIO16 ──── DC                                               │
    │    GPIO17 ──── CS                                               │
    │    GPIO18 ──── SCK             ┌──────────────────────────┐     │
    │    GPIO23 ──── MOSI            │   4.0" IPS DISPLAY       │     │
    │    GPIO21 ──── BL              │   (ILI9488, 480x320)     │     │
    │    GPIO19 ──── BCLK            │                          │     │
    │    GPIO0  ──── LRC             │   14-pin connector:      │     │
    │    GPIO22 ──── DIN             │                          │     │
    │                                │   1  VCC ◄── 3.3V        │     │
    │    3.3V   ──── VCC, RST*,      │   2  GND ◄── GND         │     │
    │                SD pin          │   3  CS  ◄── GPIO17      │     │
    │    GND    ──── GND, RST cap,   │   4  RST ◄── (hardwired) │     │
    │                BL pull-down    │   5  DC  ◄── GPIO16      │     │
    └────────────────────────────────│   6  MOSI◄── GPIO23      │     │
                                     │   7  SCK ◄── GPIO18      │     │
                                     │   8  BL  ◄── GPIO21      │     │
                                     │   9  MISO    (NC)        │     │
                                     │  10  T_CLK   (NC)        │     │
                                     │  11  T_CS    (NC)        │     │
                                     │  12  T_DIN   (NC)        │     │
                                     │  13  T_DO    (NC)        │     │
                                     │  14  T_IRQ   (NC)        │     │
                                     └──────────────────────────┘


    DISPLAY RST CIRCUIT (hardwired, no GPIO):

         3.3V
          │
        [10K Ω]
          │
          ├──────────── Display RST (pin 4)
          │
       [100nF]
          │
         GND


    BACKLIGHT PULL-DOWN (CRITICAL for sleep battery life):

         GPIO21 ────┬──── Display BL (pin 8)
                    │
                 [10K Ω]
                    │
                   GND


    MAX98357A AMPLIFIER:                         SPEAKER:

    ┌──────────────────┐                    ┌──────────────┐
    │   MAX98357A       │                    │  SOTAMIA     │
    │                   │                    │  4Ω 3W       │
    │   VIN ◄── 3.3V   │                    │              │
    │   GND ◄── GND    │    Speaker out     │              │
    │                   │   ┌────────────►   │  + terminal  │
    │   BCLK◄── GPIO19 │   │                │              │
    │   LRC ◄── GPIO0  │   │   ┌─────────►  │  - terminal  │
    │   DIN ◄── GPIO22 │   │   │            │              │
    │                   │   │   │            └──────────────┘
    │   SD  ◄── 3.3V   │   │   │
    │  (L+R mono mix)  │   │   │
    │                   │   │   │
    │   Speaker+ ───────┘   │
    │   Speaker- ───────────┘
    │                   │
    └──────────────────┘


    BATTERY (LiPo mod):

    ┌──────────────────┐
    │  PL985525        │        Solder wires to LILYGO board
    │  3.7V 8000mAh   │        B+ and B- pads (after removing
    │  LiPo pouch      │        the 18650 holder)
    │                  │
    │  Red (+) ─────────────►  B+ pad on LILYGO board
    │  Black (-) ───────────►  B- pad on LILYGO board
    │                  │
    └──────────────────┘
    Charges via USB-C on the LILYGO board (built-in charger IC)


    BULK CAPACITOR (recommended for modem stability):

         B+ ────┬──────────── to board
                │
          [1000-2200 µF]      (electrolytic, rated 6.3V or higher)
                │
         B- ────┴──────────── to board


╔══════════════════════════════════════════════════════════════════════════╗
║                        CONNECTION SUMMARY                              ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  LILYGO pin    Wire to              Color suggestion                   ║
║  ─────────     ───────              ────────────────                   ║
║  3.3V          Display VCC          Red                                ║
║  3.3V          MAX98357A VIN        Red                                ║
║  3.3V          MAX98357A SD pin     Red (short jumper)                 ║
║  3.3V          RST circuit (10K)    Red                                ║
║  GND           Display GND          Black                              ║
║  GND           MAX98357A GND        Black                              ║
║  GND           RST circuit (cap)    Black                              ║
║  GND           BL pull-down (10K)   Black                              ║
║  GPIO16        Display DC           Yellow                             ║
║  GPIO17        Display CS           Orange                             ║
║  GPIO18        Display SCK          Blue                               ║
║  GPIO23        Display MOSI         Green                              ║
║  GPIO21        Display BL           White                              ║
║  GPIO19        MAX98357A BCLK       Purple                             ║
║  GPIO0         MAX98357A LRC        Gray                               ║
║  GPIO22        MAX98357A DIN        Brown                              ║
║  MAX Speaker+  Speaker + terminal   Red (speaker wire)                 ║
║  MAX Speaker-  Speaker - terminal   Black (speaker wire)               ║
║  LiPo Red      Board B+ pad         Red (battery wire)                 ║
║  LiPo Black    Board B- pad         Black (battery wire)               ║
║                                                                        ║
║  NOT CONNECTED: Display pins 9-14 (MISO + all touch)                   ║
║                                                                        ║
╠══════════════════════════════════════════════════════════════════════════╣
║  Total wires: 17 signal + 2 battery + 2 speaker = 21 connections       ║
║  Total GPIOs used: 8 of 8 available (0 spare outputs)                  ║
║  Extra components: 2x 10K resistor, 1x 100nF cap, 1x bulk cap         ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## Power Architecture

### Sleep Mode (~3.2mA total)

| Component | State | Current |
|-----------|-------|:-------:|
| ESP32 | Deep sleep | 0.157mA |
| A7670 modem | DTR sleep (registered to network) | ~3mA |
| Display | Sleep mode (cmd 0x10 sent) | ~0.01mA |
| Backlight | OFF (GPIO21 floating, 10K pull-down to GND) | 0mA |
| MAX98357A | Auto-shutdown (no I2S clock) | ~0.003mA |
| SD card | Idle (powered but inactive) | ~0.1mA |
| GPS | OFF | 0mA |
| **Total** | | **~3.3mA** |

### Poll Mode -- No Content (~5 sec, ~200mA avg)

| Step | Duration | Current |
|------|:--------:|:-------:|
| ESP32 wake + DTR LOW | ~2 sec | ~100mA |
| HTTP GET request | ~3 sec | ~300mA |
| DTR HIGH + deep sleep | instant | -- |
| **Per poll cost** | **~5 sec** | **~0.28mAh** |

### Active Mode -- Content Delivery (~500mA avg, 2A peak)

| Component | Current |
|-----------|:-------:|
| ESP32 (240MHz, processing) | ~100mA |
| A7670 4G data (downloading) | 500mA avg, 2A peak TX |
| Display + backlight | 80-150mA |
| MAX98357A + speaker | 100-300mA |
| GPS (if requested) | 30-50mA |

### Battery Life Calculation (8000mAh LiPo, 5 min poll, DTR sleep)

**Daily power budget:**

| Source | Calculation | mAh/day |
|--------|-------------|:-------:|
| Sleep drain | 3.3mA × 24h | 79.2 |
| Empty polls (worst case: 288/day) | 288 × 0.28mAh | 80.6 |
| Content delivery (5 msgs, ~3 min each) | 15 min × 500mA / 60 | 125.0 |
| GPS check (1x/day, 2 min) | 2 min × 200mA / 60 | 6.7 |
| **Total daily drain** | | **~291mAh** |

**Battery life: 8000 / 291 = ~27.5 days**

| Scenario | Daily Drain | Battery Life |
|----------|:-----------:|:------------:|
| 2-3 msgs/day, no GPS | ~210mAh | **~38 days** |
| 5 msgs/day + GPS 1x/day | ~291mAh | **~27 days** |
| 10 msgs/day + GPS 3x/day | ~430mAh | **~18 days** |
| 15+ msgs/day (Karim is spamming) | ~570mAh | **~14 days** |

### Power Optimization Summary

| Optimization | Saves | Status |
|-------------|:-----:|:------:|
| DTR sleep between polls | ~17mA constant | PLANNED |
| 10K pull-down on BL (GPIO21) | ~100mA during sleep | PLANNED |
| ILI9488 sleep command before ESP32 sleep | ~2mA during sleep | PLANNED |
| MAX98357A auto-shutdown | ~0 (automatic) | FREE |
| Bulk capacitor for modem spikes | prevents crashes | RECOMMENDED |
| Don't 4G TX + audio simultaneously | prevents crashes | FIRMWARE |
| Battery monitoring + forced shutdown <3.3V | protects LiPo | FIRMWARE |

---

## Firmware Deep Sleep Sequence (pseudocode)

### Going to Sleep

```c
void enterDeepSleep(int seconds) {
    // 1. Stop audio
    i2s_driver_uninstall(I2S_NUM_0);

    // 2. GPIO0 (I2S LRC) -- set HIGH for safe boot, hold through sleep
    pinMode(0, OUTPUT);
    digitalWrite(0, HIGH);
    gpio_hold_en((gpio_num_t)0);           // GPIO0 IS RTC -- holds in deep sleep

    // 3. GPIO21 (backlight) -- set LOW
    //    NOT an RTC GPIO, so gpio_hold won't persist
    //    The 10K pull-down resistor keeps it LOW when it floats
    digitalWrite(21, LOW);

    // 4. Display sleep
    sendDisplayCommand(0x10);               // ILI9488 Sleep In

    // 5. Modem DTR sleep
    digitalWrite(MODEM_DTR_PIN, HIGH);      // GPIO25 -- modem naps
    gpio_hold_en((gpio_num_t)MODEM_DTR_PIN); // GPIO25 IS RTC -- holds

    // 6. Hold power and reset pins
    gpio_hold_en((gpio_num_t)BOARD_POWERON_PIN);  // GPIO12 HIGH -- keep power
    gpio_hold_en((gpio_num_t)MODEM_RESET_PIN);    // GPIO5 LOW -- don't reset
    gpio_deep_sleep_hold_en();                     // persist all RTC holds

    // 7. Sleep for N seconds
    esp_sleep_enable_timer_wakeup(seconds * 1000000ULL);
    esp_deep_sleep_start();
}
```

### Waking Up

```c
void setup() {
    // 1. CRITICAL: set power pin HIGH immediately (prevents reset on battery)
    pinMode(BOARD_POWERON_PIN, OUTPUT);
    digitalWrite(BOARD_POWERON_PIN, HIGH);

    // 2. Release all GPIO holds
    gpio_hold_dis((gpio_num_t)MODEM_DTR_PIN);
    gpio_hold_dis((gpio_num_t)BOARD_POWERON_PIN);
    gpio_hold_dis((gpio_num_t)MODEM_RESET_PIN);
    gpio_hold_dis((gpio_num_t)0);

    // 3. Wake modem from DTR nap
    pinMode(MODEM_DTR_PIN, OUTPUT);
    digitalWrite(MODEM_DTR_PIN, LOW);
    delay(2000);                            // modem wakes up (~2 sec)

    // 4. Check server for content
    if (serverHasPendingContent()) {

        // 5. Download content to SD card
        downloadContent();

        // 6. Wake display
        sendDisplayCommand(0x11);           // ILI9488 Sleep Out
        delay(120);                         // display needs 120ms after wake
        pinMode(21, OUTPUT);
        digitalWrite(21, HIGH);             // backlight ON

        // 7. Init audio
        i2s_driver_install(...);
        i2s_set_pin(...);                   // BCLK=19, LRC=0, DIN=22

        // 8. Play content (character animations, TTS, media)
        playContent();

        // 9. Check if GPS requested
        if (gpsRequested()) {
            getAndReportGPS();
        }

        // 10. Report battery level
        reportBatteryLevel();
    }

    // 11. Check battery -- force shutdown if too low
    float voltage = readBatteryVoltage();   // GPIO35 ADC
    if (voltage < 3.3) {
        // Show "I'm dying" animation if display is up
        enterDeepSleep(3600);               // sleep 1 hour, hope she charges
    }

    // 12. Back to sleep
    enterDeepSleep(300);                    // 5 minutes = 300 seconds
}
```

---

## Software Architecture Overview

### ESP32 Firmware (Arduino + ESP-IDF APIs)
- Timer-based deep sleep (5 min intervals)
- DTR modem sleep control (GPIO25)
- HTTP/HTTPS client via TinyGSM + A7670 AT commands
- SD card file management (HSPI: GPIO 14, 2, 15, 13)
- ILI9488 display driver over VSPI (GPIO 18, 23, 17, 16)
- I2S audio output (GPIO 19, 0, 22) via MAX98357A
- GPS via modem AT commands (no extra pins, same UART)
- Battery level monitoring via ADC (GPIO35)
- Display sleep/wake commands (0x10/0x11)

### Server (Backend -- simple REST API)
- `POST /api/content` -- Karim's app pushes content
- `GET /api/device/{id}/pending` -- device checks for new content
- `GET /api/device/{id}/content/{contentId}` -- device downloads specific content
- `POST /api/device/{id}/status` -- device reports battery level
- `POST /api/device/{id}/location` -- device reports GPS coordinates
- `GET /api/device/{id}/flags` -- server tells device if GPS is requested
- TTS engine (converts text to audio files, serves them for download)
- Media storage (photos, videos, TTS audio files)
- Content queue (multiple messages wait for next poll)
- GPS location storage and history

### Karim's App (React Native Expo)
- Send text messages
- Trigger character animations/emotions
- Send photos from gallery
- Send/record audio
- Send video
- Request device GPS location
- View location history (is she carrying it?)
- See device battery level
- Character emotion/animation picker

### SPI Bus Configuration (two separate instances)

```c
// SD card -- default SPI object (HSPI)
SPI.begin(14, 2, 15);          // SCK=14, MISO=2, MOSI=15
SD.begin(13);                   // CS=13

// Display -- separate VSPI instance
SPIClass displaySPI(VSPI);
displaySPI.begin(18, -1, 23, -1);  // SCK=18, no MISO, MOSI=23, no CS
// CS (GPIO17) and DC (GPIO16) managed manually as regular GPIO outputs
```

---

## Problems & Solutions Summary

| # | Problem | Status | Solution | Cost |
|---|---------|:------:|----------|:----:|
| 1 | Pin shortage | SOLVED | No touch (-4 pins) + hardwire RST (-1) + use all 8 GPIOs | AED 0 |
| 2 | Boot pin conflicts | SOLVED | All assigned pins are boot-safe (verified from repo) | AED 0 |
| 3 | SPI bus conflict | SOLVED | Display on VSPI, SD card on HSPI, separate SPI instances | AED 0 |
| 4 | Battery life | SOLVED | 8000mAh LiPo + DTR sleep + timer polling = ~27 days | AED 45 |
| 5 | Modem current spikes | PLANNED | 1000uF+ bulk capacitor across battery pads | AED 0-15 |
| 6 | 18650 holder removal | PLANNED | Desolder holder, wire LiPo to B+/B- pads | AED 0 |
| 7 | Display wiring | SOLVED | Connect 7 of 14 pins, skip touch + MISO | AED 0 |
| 8 | Mono audio | SOLVED | MAX98357A SD pin to 3.3V (L+R mix) | AED 0 |
| 9 | Wake-up method | DECIDED | Timer every 5 min + DTR sleep (no SMS, no gateway) | AED 0 |
| 10 | Backlight sleep drain | SOLVED | 10K pull-down resistor on GPIO21 | AED 0 |
| 11 | GPIO21 not RTC | SOLVED | Pull-down resistor handles it (gpio_hold not needed) | AED 0 |
| 12 | GPIO0 boot safety | SOLVED | Set HIGH + gpio_hold before sleep, internal pull-up as backup | AED 0 |
| 13 | Display idle power | PLANNED | Send ILI9488 sleep command (0x10) before deep sleep | AED 0 |
| 14 | LiPo protection | PLANNED | Monitor GPIO35 ADC, force sleep below 3.3V | AED 0 |

**Total cost: AED 255.83** (all parts) + AED 0-15 optional (bulk capacitor)

---

## Pre-Build Verification Checklist

When the board arrives, check these BEFORE wiring anything:

- [ ] **Verify A7670E chip** -- look at modem markings. Must say A7670E, NOT A7670G.
      If A7670G: there's a separate L76K GPS chip using GPIO19/21/22/23. Everything breaks.
- [ ] **Check for I2C pull-ups on GPIO21/22** -- measure voltage on GPIO21 and GPIO22
      with nothing connected. If ~3.3V, there are pull-ups that fight our BL pull-down.
      If ~0V or floating, we're good.
- [ ] **Verify display pinout** -- check the silkscreen on the actual 14-pin connector
      against our assumed pinout (VCC, GND, CS, RST, DC, MOSI, SCK, BL, MISO, T_*)
- [ ] **Test SIM card** -- insert SIM, power on, send SMS to verify the operator
      delivers SMS over 4G (needed as fallback even though we use timer polling)
- [ ] **Test SD card slot** -- insert a formatted micro SD, run the official SDCard example

---

## What's NOT Decided Yet

- [ ] Pixel art character design (Karim's choice)
- [ ] Server tech stack and hosting
- [ ] TTS engine/service choice
- [ ] Enclosure exact dimensions and manufacturing (transparent + DIY green look)
- [ ] Video playback format and codec (ESP32 limitations -- likely MJPEG or frame-by-frame)
- [ ] Low battery animations ("I'm sleepy", "feed me")
- [ ] First boot experience exact sequence and greeting message
- [ ] Charging animation (character happy when USB-C plugged in?)

---

## Reference Files in Repo (docs/)

| File | What It Contains |
|------|------------------|
| `docs/examples/ATdebug/utilities.h` | **Pin definitions** for all board variants (lines 88-145 for T-A7670) |
| `docs/examples/WakeupByRingOrSMS/` | SMS wake-up example (reference for RI pin + gpio_hold patterns) |
| `docs/examples/WakeupByRingOrSMS_SendLocation/` | **GPS location via SMS** -- starter code for GPS tracking feature |
| `docs/examples/DeepSleep/` | Timer-based deep sleep example (157uA measured, modem-off mode) |
| `docs/examples/ModemSleep/` | **DTR sleep control** -- exact code for modem nap mode |
| `docs/examples/ModemPlayMusic/` | A7670 built-in audio (fallback option, not primary) |
| `docs/examples/SDCard/` | SD card read/write example with HSPI pin config |
| `docs/datasheet/A76XX/` | AT command manual PDF |
