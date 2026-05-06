# Cute ESP -- Complete Project Knowledge Base

> Single source of truth for the entire project. Everything we know, every decision made,
> every problem found and solved, all the wiring, firmware patterns, math behind the numbers,
> and the stuff still left to decide.

---

## Table of Contents

1. [The Story & Purpose](#1-the-story--purpose)
2. [Project Goals & Success Criteria](#2-project-goals--success-criteria)
3. [System Architecture](#3-system-architecture)
4. [Hardware Bill of Materials](#4-hardware-bill-of-materials)
5. [Hardware Modifications Required](#5-hardware-modifications-required)
6. [Pin Assignment (Verified)](#6-pin-assignment-verified)
7. [Complete Wiring Diagram](#7-complete-wiring-diagram)
8. [Power Architecture & Battery Math](#8-power-architecture--battery-math)
9. [Sleep/Wake Cycle (Full Detail)](#9-sleepwake-cycle-full-detail)
10. [Behavior States](#10-behavior-states)
11. [Content Types & Delivery](#11-content-types--delivery)
12. [Firmware Implementation Patterns](#12-firmware-implementation-patterns)
13. [Server API Specification](#13-server-api-specification)
14. [Karim's React Native App](#14-karims-react-native-app)
15. [Problems Found & Solutions](#15-problems-found--solutions)
16. [Critical Gotchas & Pitfalls](#16-critical-gotchas--pitfalls)
17. [Pre-Build Verification Checklist](#17-pre-build-verification-checklist)
18. [What's Still Not Decided](#18-whats-still-not-decided)
19. [Reference Files in Repo](#19-reference-files-in-repo)
20. [Glossary](#20-glossary)

---

## 1. The Story & Purpose

**Karim is a friend** of the developer (MHD). Karim wants to marry his dream girl. The
situation is an arranged marriage:

- Her parents have **already approved** the match
- **She has NOT met him** in person, never even seen him
- **She is not yet convinced** about marrying him

The plan is to **gift her a mysterious device** -- a small portable gadget she receives
with no explanation of what it does or who sent it. Inside lives a cute pixel art
character. Karim secretly controls this character from his phone over 4G LTE.

The character will:
- **Talk to her** via TTS (text-to-speech)
- **Show emotions** through pixel art animations and gestures
- **Send her messages** as text + audio
- **Send her photos** Karim picks
- **Send her videos** Karim records
- **Send her music** Karim chooses
- **Greet her** when first powered on
- **Show its mood** (sleepy when battery low, happy when charging)

She doesn't know any of this is from Karim. The device is meant to **charm her over time**.
The character becomes a friend, then something more, then she starts wondering who's
behind it. Eventually her curiosity and affection grow until she **tells her dad she's
interested in the marriage**. That's the win condition.

### The Subtle Genius

Karim also wants to know if she's **carrying the device with her** -- to uni, to the mall,
to friends. If she is, that means she's emotionally attached. The device has on-demand
**GPS location reporting** so Karim can quietly verify her interest without ever asking.

### Communication Rules

- **One-way only** until the win condition is met
- She receives, never sends back
- She has **no idea** the device is connected to a person
- After she says yes to marriage: future versions could enable two-way (out of scope)

---

## 2. Project Goals & Success Criteria

### Primary Goals

1. **Mystery & Charm** -- the device must feel magical, not technical
2. **Reliability** -- it must "just work" with no maintenance from her
3. **Battery life** -- ideally a month between charges, like a phone
4. **Real-time feel** -- when Karim sends something, she gets it within 5 minutes
5. **Aesthetic** -- DIY green PCB look, retro, transparent case (intentional raw style)

### Non-Goals

- Native interactivity from her side (one-way only)
- Multi-user support (single device, single recipient)
- Voice calls (she could just call him -- this is mystery, not chat)
- Real video conferencing (she shouldn't see Karim before she chooses)

---

## 3. System Architecture

### High-Level Flow

```
┌────────────────────────────────────────────────────────────────────────┐
│                                                                        │
│   Karim's phone (React Native Expo app)                                │
│                  │                                                     │
│                  │ HTTPS POST                                          │
│                  ▼                                                     │
│   Remote Server (REST API)                                             │
│       - Stores content queue                                           │
│       - Generates TTS audio                                            │
│       - Stores media files                                             │
│       - Tracks GPS history & battery level                             │
│                  ▲                                                     │
│                  │ HTTPS GET / POST                                    │
│                  │ (every 5 min over 4G LTE)                           │
│                  │                                                     │
│   ESP32 Device (LILYGO T-SIM A7670E R2)                                │
│       - Polls server every 5 min while in deep sleep cycle             │
│       - Downloads new content when present                             │
│       - Plays content via display + speaker                            │
│       - Reports battery + GPS when asked                               │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### Why Timer Polling Over SMS Wake-up

We considered SMS wake-up (modem RI pin triggers ESP32 from deep sleep on incoming SMS).
We **rejected it** because:

| Concern | SMS Wake-up | Timer Polling |
|---------|:-----------:|:-------------:|
| Latency | Instant (seconds) | 0-5 min |
| Server complexity | REST + SMS gateway | REST only |
| Per-message cost | SMS gateway fees | Free |
| Operator dependency | Needs "SMS over SGs" | Any 4G works |
| Battery life @ 5 msgs/day | ~40 days | ~27 days |
| Code complexity | High (RI pin + AT cmds) | Low (HTTP polling) |

**Tradeoff:** ~5 min latency for ~13 days less battery, but **vastly simpler** server and no
gateway costs. For a romantic message, 5 minutes is invisible. We chose simplicity.

### Why DTR Sleep is Mandatory

Without DTR sleep (modem fully awake between polls): ~20mA idle = **6-12 days battery**.
With DTR sleep (modem napping at ~3mA): **~27-38 days battery**.

DTR sleep is non-negotiable.

---

## 4. Hardware Bill of Materials

### Already Purchased (Total: AED 255.83)

| # | Component | Specs | Price |
|---|-----------|-------|:-----:|
| 1 | LILYGO T-SIM A7670E R2 | ESP32 + 4G LTE Cat1 + Built-in GPS + SD slot + 18650 holder | AED 141.70 |
| 2 | SOTAMIA Mini Speakers (2pcs) | 4Ω, 3W, full-range, 20×30mm | AED 9.46 |
| 3 | MAX98357A I2S Amplifier (5pcs) | Class D, 3W output, mono | AED 3.75 |
| 4 | 4.0" IPS SPI Display | ILI9488 + FT6236 capacitive touch, 480×320, 14-pin connector | AED 55.92 |
| 5 | PL985525 LiPo Battery | 3.7V, 8000mAh, lithium polymer pouch | AED 45.00 |

### Additional Components Needed (from parts bin)

| Part | Purpose | Quantity |
|------|---------|:--------:|
| 10K Ω resistor (1/4W) | Display RST pull-up + GPIO21 backlight pull-down | 2 |
| 100nF ceramic capacitor | Display RST RC reset circuit | 1 |
| 1000-2200 µF electrolytic cap (≥6.3V) | Bulk decoupling for modem current spikes | 1 |
| Jumper wires / hookup wire | Connections | ~25 |
| Solder + flux | Assembly | -- |

### LILYGO Board Specs (Detailed)

- **MCU:** ESP32 (Tensilica LX6 dual-core, 240MHz)
- **Flash:** 4MB
- **PSRAM:** 8MB
- **Modem:** A7670E (Asia/Europe LTE Cat1: B1/B3/B5/B7/B8/B20)
- **GPS:** Built into A7670E modem (BeiDou/GPS/GLONASS) -- accessed via AT commands, no extra pins
- **Connectivity:** Wi-Fi 802.11 b/g/n, BT v4.2 (BR/EDR + BLE)
- **Storage:** Built-in SD card slot (HSPI: GPIO 14, 2, 15, 13)
- **Power:** 18650 holder + USB-C charging (single-cell lithium charger IC)
- **SIM:** Nano SIM, 1.8V/3.0V
- **USB:** USB-C for power/programming + Micro USB for A7670 firmware updates (separate)
- **Voltage range:** 3.4-4.2V (battery direct)

### Display Specs (4.0" IPS)

- **Controller:** ILI9488
- **Touch:** FT6236 capacitive (NOT USED in this project)
- **Resolution:** 480×320
- **Interface:** SPI (4-wire, write-only mode)
- **Connector:** 14-pin FPC ribbon cable
- **Color:** "Yellow" PCB variant
- **Power:** 3.3V logic, 3.3V backlight

### Speaker & Amp

- **MAX98357A:** I2S input, Class D output, 3W into 4Ω, integrated DAC
- **Speaker:** SOTAMIA 4Ω 3W full-range, ~20×30mm flat speaker

### Battery (LiPo replacement)

- **PL985525** -- standard naming convention: 9mm × 55mm × 25mm pouch cell
- **3.7V nominal, 4.2V max charge, 3.0V cutoff**
- **8000mAh capacity** (vs ~3500mAh for typical 18650)
- **Single cell** -- compatible with onboard charger as-is

---

## 5. Hardware Modifications Required

### Modification 1: Battery Swap

1. **Desolder the 18650 battery holder** from the LILYGO board (hot air or iron + braid)
2. Identify the **B+** and **B-** pads on the PCB underneath
3. Solder the LiPo's **red wire to B+**, **black wire to B-**
4. Insulate exposed solder joints with heat shrink or kapton tape
5. The board's USB-C charger IC will charge the LiPo same as it would a 18650 (same voltage)

### Modification 2: Display RST RC Circuit (saves 1 GPIO)

Build a tiny power-on reset circuit so we don't need a GPIO for RST:

```
    3.3V ────┬───[10KΩ]─── Display RST (pin 4)
             │
          [100nF]
             │
            GND
```

When power applies, the cap holds RST low briefly (~1ms), then the resistor pulls it
HIGH. Display resets itself on every power-on. No firmware control needed.

### Modification 3: Backlight Pull-Down on GPIO21 (CRITICAL)

GPIO21 is **not an RTC GPIO**. `gpio_hold_en()` does NOT persist its state through deep
sleep. When ESP32 sleeps, GPIO21 floats. The display backlight has a pull-up internally,
so floating = backlight stays on = ~100mA constant drain = battery dies in days.

**Fix:** Solder a 10K resistor from GPIO21 to GND.

```
    GPIO21 ────┬──── Display BL (pin 8)
                │
             [10KΩ]
                │
               GND
```

- During sleep: GPIO21 floats, pull-down keeps it LOW = backlight off
- During active: ESP32 drives GPIO21 HIGH, easily overrides 10K (0.33mA "leakage")

### Modification 4: Bulk Capacitor (Recommended)

The A7670 modem can spike to 2A during 4G transmissions. The LiPo can supply this, but
the wires/PCB traces have impedance, causing voltage sag that may reset the ESP32.

**Fix:** Solder a **1000-2200µF electrolytic capacitor** across the battery terminals
(or B+/B- pads) to absorb the spikes:

```
    B+ ────┬──── (to board)
           │
       [1000-2200µF]   (≥6.3V rated)
           │
    B- ────┴──── (to board)
```

### Modification 5: No Touch Wiring

Touch is not used. Display pins 9-14 (MISO + 5 touch signals) are left **unconnected**.
This frees up pins for other uses but mostly just simplifies the build.

---

## 6. Pin Assignment (Verified)

**Source of truth:** `docs/examples/ATdebug/utilities.h` lines 88-145
(the `LILYGO_T_A7670` block in the official LILYGO repo).

### Pins Used Internally by Board (DO NOT TOUCH)

| GPIO | Define | Function | Notes |
|:----:|--------|----------|-------|
| 4 | BOARD_PWRKEY_PIN | Modem power key | Pulse to power on/off (100ms) |
| 5 | MODEM_RESET_PIN | Modem reset | Active HIGH (MODEM_RESET_LEVEL = HIGH) |
| 12 | BOARD_POWERON_PIN | Modem & SD power | **MUST be HIGH for battery operation** |
| 25 | MODEM_DTR_PIN | Modem DTR (sleep ctrl) | HIGH = sleep, LOW = wake |
| 26 | MODEM_TX_PIN | UART TX to modem | Serial1 |
| 27 | MODEM_RX_PIN | UART RX from modem | Serial1 |
| 33 | MODEM_RING_PIN | RI pin (incoming SMS/call) | Reserved (not used in timer mode) |
| 35 | BOARD_BAT_ADC_PIN | Battery voltage divider | Input only -- analog read |
| 36 | BOARD_SOLAR_ADC_PIN | Solar input ADC | Input only, v1.4 board only |
| 2 | BOARD_MISO_PIN | SD MISO | HSPI bus |
| 13 | BOARD_SD_CS_PIN | SD chip select | HSPI bus |
| 14 | BOARD_SCK_PIN | SD SCK | HSPI bus |
| 15 | BOARD_MOSI_PIN | SD MOSI | HSPI bus |

### Pins Free on Headers (output-capable)

| GPIO | Available | RTC GPIO? | Notes |
|:----:|:---------:|:---------:|-------|
| 0 | Yes | YES | Boot-sensitive (must be HIGH at boot) |
| 16 | Yes | No | Free |
| 17 | Yes | No | Free |
| 18 | Yes | No | VSPI default SCK |
| 19 | Yes | No | VSPI default MISO (we don't use MISO) |
| 21 | Yes | No | Board labels as I2C SDA (no actual I2C device) |
| 22 | Yes | No | Board labels as I2C SCL (no actual I2C device) |
| 23 | Yes | No | VSPI default MOSI |

### Pins Free (input-only)

| GPIO | Notes |
|:----:|-------|
| 34 | Input only (no internal pull-up/down) |
| 39 | Input only (no internal pull-up/down) |

### Final Pin Assignments (all 8 output GPIOs used)

| GPIO | Assignment | Why This Pin |
|:----:|------------|-------------|
| 16 | Display DC | Free, no boot issues |
| 17 | Display CS | Free, no boot issues |
| 18 | Display SCK | VSPI hardware SPI default |
| 23 | Display MOSI | VSPI hardware SPI default |
| 21 | Display BL | Last free pin (with 10K pull-down resistor) |
| 19 | I2S BCLK | Free, output capable |
| 0 | I2S LRC | RTC GPIO -- can hold HIGH through deep sleep for safe boot |
| 22 | I2S DIN | Free, output capable |

### Hardwired (No GPIO)

| Signal | Wiring | Notes |
|--------|--------|-------|
| Display RST | 3.3V via 10K + 100nF cap to GND | Auto power-on reset |
| MAX98357A SD pin | Tied to 3.3V | L+R mono mix mode, 15dB gain |

### Why GPIO0 for I2S LRC is Safe

GPIO0 must be **HIGH at boot** or ESP32 enters flash/download mode. Our usage is safe:

1. **Boot phase:** I2S peripheral isn't started yet. GPIO0's internal pull-up keeps it HIGH. ✅
2. **Active operation:** I2S toggles LRC. Boot is long done. Doesn't matter. ✅
3. **Pre-sleep:** Firmware stops I2S, sets GPIO0 HIGH, calls `gpio_hold_en()`. GPIO0 IS an RTC GPIO so the hold persists through deep sleep. ✅
4. **Wake from sleep:** GPIO0 still HIGH from the hold. Boot ROM reads HIGH. Normal boot. ✅
5. **Crash recovery:** Even if the hold is somehow lost, the internal pull-up (~45K) pulls GPIO0 HIGH. Normal boot. ✅

The only theoretical risk: someone presses the physical Boot button on the board while
audio is playing. This causes a brief audio glitch. Since the device is in a sealed
transparent case, this won't happen.

### Why GPIO21 Backlight Needs External Pull-Down

GPIO21 is NOT in the ESP32 RTC GPIO list. ESP32 RTC GPIOs are:
**0, 2, 4, 12, 13, 14, 15, 25, 26, 27, 32, 33, 34, 35, 36, 39**

`gpio_hold_en(21)` works only in light sleep, not deep sleep. When ESP32 enters deep
sleep, GPIO21 becomes high-impedance. If the display BL line has any pull-up (most do),
the backlight turns on. Without intervention this drains 80-150mA continuously during
"sleep" -- catastrophic.

**The 10K pull-down resistor** between GPIO21 and GND solves it: when GPIO21 floats, the
resistor wins and pulls it LOW (backlight off). When ESP32 drives GPIO21 HIGH, it
overrides the weak 10K easily (current sink: 3.3V/10K = 0.33mA, negligible).

---

## 7. Complete Wiring Diagram

```
╔══════════════════════════════════════════════════════════════════════════╗
║                  CUTE ESP -- COMPLETE WIRING                           ║
╚══════════════════════════════════════════════════════════════════════════╝

LILYGO T-A7670E R2 (top-level)
═══════════════════════════════
Internal pins (DO NOT WIRE):
  GPIO 4, 5, 12, 25, 26, 27, 33, 35, 36 -- modem control / battery sense
  GPIO 2, 13, 14, 15 -- SD card (HSPI bus)

Used pins (8):
  GPIO 16 -> Display DC
  GPIO 17 -> Display CS
  GPIO 18 -> Display SCK
  GPIO 23 -> Display MOSI
  GPIO 21 -> Display BL  (also: 10K resistor to GND)
  GPIO 19 -> MAX98357A BCLK
  GPIO  0 -> MAX98357A LRC
  GPIO 22 -> MAX98357A DIN

Power rails:
  3.3V -> Display VCC
  3.3V -> MAX98357A VIN
  3.3V -> MAX98357A SD pin (mono L+R mix mode)
  3.3V -> Display RST circuit (10K resistor)
  GND  -> Display GND, MAX98357A GND, RST cap, BL pull-down


DISPLAY 14-PIN CONNECTOR (only 7 pins connected)
═══════════════════════════════════════════════════
  Pin 1  VCC    <-- 3.3V                [Red wire]
  Pin 2  GND    <-- GND                 [Black wire]
  Pin 3  CS     <-- GPIO17              [Orange wire]
  Pin 4  RST    <-- (RC reset circuit)  [hardwired]
  Pin 5  DC     <-- GPIO16              [Yellow wire]
  Pin 6  MOSI   <-- GPIO23              [Green wire]
  Pin 7  SCK    <-- GPIO18              [Blue wire]
  Pin 8  BL     <-- GPIO21              [White wire]
  Pin 9  MISO   -- not connected
  Pin 10 T_CLK  -- not connected (touch unused)
  Pin 11 T_CS   -- not connected
  Pin 12 T_DIN  -- not connected
  Pin 13 T_DO   -- not connected
  Pin 14 T_IRQ  -- not connected


DISPLAY RST RC CIRCUIT (hardwired auto-reset)
═══════════════════════════════════════════════
       3.3V ───[10K Ω]───┬─── Display RST (pin 4)
                          │
                       [100nF]
                          │
                         GND


GPIO21 BACKLIGHT PULL-DOWN (CRITICAL)
══════════════════════════════════════
  GPIO21 ───┬─── Display BL (pin 8)
            │
         [10K Ω]
            │
           GND


MAX98357A AUDIO AMP
═══════════════════
  VIN     <-- 3.3V                    [Red]
  GND     <-- GND                     [Black]
  BCLK    <-- GPIO19                  [Purple]
  LRC     <-- GPIO0                   [Gray]
  DIN     <-- GPIO22                  [Brown]
  SD pin  <-- 3.3V (mono mix, 15dB)   [short jumper to VIN]
  SPK+    --> Speaker + terminal      [Red speaker wire]
  SPK-    --> Speaker - terminal      [Black speaker wire]
  GAIN    -- not connected (default 9dB)


BATTERY (LiPo PL985525, 8000mAh)
═══════════════════════════════════
  Red   --> B+ pad on LILYGO board   [after desoldering 18650 holder]
  Black --> B- pad on LILYGO board

  Optional bulk cap across B+/B-:
    B+ ───┬─── (to board)
          │
      [1000-2200µF]   (≥6.3V rated electrolytic)
          │
    B- ───┴─── (to board)


CONNECTION COUNT
════════════════
  Signal wires:    17
  Battery wires:    2
  Speaker wires:    2
  TOTAL:           21 connections
  GPIOs used:      8 of 8 available
  Resistors:        2× 10K (RST + BL pull-down)
  Capacitors:       1× 100nF (RST), 1× 1000-2200µF (bulk, optional)
```

---

## 8. Power Architecture & Battery Math

### Sleep Mode Current Breakdown (~3.3mA total)

| Component | State | Current |
|-----------|-------|:-------:|
| ESP32 | Deep sleep (RTC only) | 0.157mA |
| A7670 modem | DTR sleep mode (network registered) | ~3.0mA |
| Display ILI9488 | Sleep mode (cmd 0x10 sent) | ~0.01mA |
| Backlight | OFF (GPIO21 LOW via 10K pull-down) | 0mA |
| MAX98357A | Auto-shutdown (no I2S clock for >1ms) | ~0.003mA |
| SD card | Idle (powered but inactive) | ~0.1mA |
| GPS | OFF | 0mA |
| **Total** | | **~3.3mA** |

### Per-Poll Cost (Empty Poll, No Content)

| Step | Duration | Avg Current | Energy |
|------|:--------:|:-----------:|:------:|
| ESP32 wake from deep sleep | <1 sec | ~80mA | minor |
| Set BOARD_POWERON_PIN HIGH | instant | -- | -- |
| DTR LOW (modem wakes from nap) | ~2 sec | ~150mA | 0.083mAh |
| HTTP GET request to server | ~3 sec | ~300mA | 0.250mAh |
| DTR HIGH (modem naps) | instant | -- | -- |
| ESP32 enters deep sleep | instant | -- | -- |
| **TOTAL per empty poll** | **~5 sec** | -- | **~0.28mAh** |

### Active Mode (Content Delivery)

| Activity | Avg Current | Notes |
|----------|:-----------:|-------|
| ESP32 @ 240MHz processing | ~100mA | Full-speed CPU |
| 4G data download | 500mA avg | Up to 2A peak during TX bursts |
| Display + backlight | 80-150mA | Backlight is dominant |
| MAX98357A + speaker | 100-300mA | Depends on volume |
| GPS (when active) | 30-50mA | Built-in GPS via modem |

### Daily Power Budget Formula

```
total_daily_mAh = (sleep_drain) + (empty_polls × 0.28mAh) + (content_minutes × 500/60) + (gps_minutes × 200/60)

where:
  sleep_drain = 3.3mA × 24h = 79.2mAh/day (constant)
  empty_polls = 288/day for 5-min intervals (assuming nothing is sent)
  content_minutes = avg minutes per message × messages per day
  gps_minutes = avg minutes per GPS fix × GPS requests per day
```

### Battery Life Table (8000mAh LiPo, 5-min polling, DTR sleep)

| Daily Pattern | Empty Polls | Active Time | GPS Time | Daily Drain | **Battery Life** |
|--------------|:-----------:|:-----------:|:--------:|:-----------:|:----------------:|
| 2-3 msgs/day, no GPS | 288 × 0.28 = 80 | 6 min × 500/60 = 50 | 0 | 209mAh | **~38 days** |
| 5 msgs/day + 1× GPS | 80 | 15 min = 125 | 2 min = 7 | 291mAh | **~27 days** |
| 7 msgs/day + 1× GPS | 80 | 21 min = 175 | 7 | 341mAh | **~23 days** |
| 10 msgs/day + 3× GPS | 80 | 30 min = 250 | 6 min = 20 | 429mAh | **~18 days** |
| 15+ msgs/day spam | 80 | 45 min = 375 | 0 | 530mAh | **~15 days** |

### Why Polling Without DTR Sleep is Catastrophic

If we did NOT use DTR sleep and instead powered the modem off between polls:

- Cold boot of modem: ~15-30 seconds at ~500mA average = **4-8mAh per poll**
- 288 polls/day × 6mAh = **1728mAh/day from polls alone**
- Battery life: 8000 / (1728 + 79 + 125) = **~4 days**

DTR sleep makes polling feasible. The modem stays registered to the network, just naps
its UART. Waking from DTR sleep is ~2 seconds (no re-registration needed).

### Charging

The LILYGO board has a **single-cell lithium charger IC** (likely TP4056 or similar) with
USB-C input. It outputs a constant ~4.2V CV charge to the battery.

- Charge current: typically 1A (set by an onboard resistor)
- 8000mAh / 1A = **~8-9 hours full charge** (CC + CV phase)
- She plugs it in overnight, like a phone

**Verify before final assembly:** check the charge current setting on the board to ensure
it's safe for the LiPo (typical safe range: 0.3-1C, so up to 8A for an 8000mAh cell -- 1A is well below that).

---

## 9. Sleep/Wake Cycle (Full Detail)

### Going to Sleep (Order Matters)

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
    sendDisplayCommand(0x10);              // ILI9488 "Sleep In"

    // 5. Modem DTR sleep -- modem keeps network registration but naps
    digitalWrite(MODEM_DTR_PIN, HIGH);     // GPIO25
    gpio_hold_en((gpio_num_t)MODEM_DTR_PIN);

    // 6. Hold critical control pins
    gpio_hold_en((gpio_num_t)BOARD_POWERON_PIN);  // GPIO12 stays HIGH
    gpio_hold_en((gpio_num_t)MODEM_RESET_PIN);    // GPIO5 stays LOW (don't reset)
    gpio_deep_sleep_hold_en();             // persist all RTC holds through deep sleep

    // 7. Configure timer wakeup
    esp_sleep_enable_timer_wakeup((uint64_t)seconds * 1000000ULL);

    // 8. Goodbye
    esp_deep_sleep_start();
}
```

### Waking Up (Boot Sequence)

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

### Why setup() and Not loop()

ESP32 deep sleep is essentially a reset. After waking, execution starts from the top of
`setup()`, not where `loop()` left off. Therefore all logic lives in `setup()`, and
`loop()` is unused.

### Initial Cold Boot (First Power-On)

The first time the device is powered (after she opens the package), there's no prior
sleep state. The ESP32 boots normally and:

1. Sets BOARD_POWERON_PIN HIGH
2. Resets the modem (PWRKEY pulse)
3. Waits for "SMS DONE" (network registration complete, ~10-30 seconds)
4. **Plays the "first boot greeting"** -- a special pre-loaded animation + TTS message
   that introduces the character ("Hello! I'm [name]!" or similar)
5. Polls server immediately for any waiting content
6. Goes to sleep, polling cycle begins

The first-boot greeting can be hardcoded in firmware or downloaded on first poll.

---

## 10. Behavior States

### State Diagram

```
                    ┌─────────────────┐
                    │  POWER OFF      │
                    │ (battery dead   │
                    │  or USB unplug) │
                    └────────┬────────┘
                             │  power applied
                             ▼
                    ┌─────────────────┐
                    │  COLD BOOT       │
                    │  (first time or  │
                    │   crash recovery)│
                    └────────┬─────────┘
                             │
                             │  greeting + initial poll
                             ▼
              ┌──── ┌─────────────────┐ ◄────────────┐
              │     │  DEEP SLEEP +    │              │
              │     │  DTR NAP (~3mA)  │              │
              │     └────────┬─────────┘              │
              │              │                        │
              │              │ timer 5 min            │
              │              ▼                        │
              │     ┌─────────────────┐               │
              │     │  POLL SERVER    │               │
              │     │  (~5 sec)       │               │
              │     └────────┬────────┘               │
              │              │                        │
              │              │                        │
              │      ┌───────┴───────┐                │
              │      │               │                │
              │   no content     new content          │
              │      │               │                │
              │      └───────────────┼────────────────┤
              │                      │                │
              │                      ▼                │
              │             ┌─────────────────┐       │
              │             │  CONTENT DELIV. │       │
              │             │  - download SD  │       │
              │             │  - display on   │       │
              │             │  - play audio   │       │
              │             │  - report bat   │       │
              │             └────────┬────────┘       │
              │                      │                │
              │           gps requested?              │
              │                      │                │
              │              ┌───────┴───────┐        │
              │              │               │        │
              │             yes              no       │
              │              │               │        │
              │              ▼               │        │
              │     ┌─────────────────┐      │        │
              │     │  GPS FIX +      │      │        │
              │     │  REPORT (~2 min)│      │        │
              │     └────────┬────────┘      │        │
              │              │               │        │
              │              └───────────────┼────────┤
              │                              │        │
              │                              ▼        │
              │      battery < 3.3V? ◄──────┘        │
              │              │                        │
              │      ┌───────┴───────┐                │
              │      │               │                │
              │     yes              no               │
              │      │               │                │
              │      ▼               └────────────────┘
              │  ┌─────────────────┐
              └──┤ EMERGENCY SLEEP │
                 │  (1 hour, then  │
                 │  re-check)      │
                 └─────────────────┘
```

### State Descriptions

**DEEP SLEEP + DTR NAP** -- The default. Device drawing 3.3mA. Display off, backlight off,
audio off. Modem registered to network but UART asleep. Lasts up to 5 minutes between polls.

**POLL** -- Brief wake, ~5 seconds. ESP32 wakes, modem wakes (~2s), HTTP GET to server,
back to sleep. If the server has nothing, device goes straight back to sleep.

**CONTENT DELIVERY** -- Active. Display on, audio playing. Duration depends on content
(typical 1-5 minutes per message). Could be 10+ minutes if Karim queues many messages.

**GPS FIX** -- Triggered by server flag during a poll. GPS module activates, gets a fix
(30s-2min depending on conditions), reports lat/lon to server, GPS off.

**LOW BATTERY** -- When voltage < ~3.7V (~25%), the character starts showing "sleepy"
animations. When < 3.5V, it says "feed me" / shows charge prompt. When < 3.3V, **emergency
shutdown**: device sleeps for 1 hour at a time, just hoping she plugs it in. This protects
the LiPo from over-discharge damage.

**CHARGING** -- USB-C plugged in. The board's charger IC takes over. Modem cannot enter
DTR sleep on USB power (VBUS keeps it awake) so power consumption is higher, but it
doesn't matter because we're plugged in. Character could show a happy "thank you for
charging me" animation when she plugs in.

---

## 11. Content Types & Delivery

### Content Types Karim Can Send

| Type | Format | Storage | Playback |
|------|--------|---------|----------|
| **Text message** | UTF-8 string | RAM | Display + TTS |
| **Animation + text** | Pixel art frame sequence + caption | SD card | Display loop + TTS |
| **Audio** | MP3 or WAV | SD card | I2S to MAX98357A |
| **Photo** | JPEG | SD card | Display |
| **Video** | MJPEG (animated JPEG sequence) | SD card | Display + audio sync |
| **Music** | MP3 | SD card | I2S to MAX98357A |

### Why MJPEG for Video

The ESP32 cannot decode H.264/H.265 video in real time. **MJPEG** (Motion JPEG -- a
sequence of JPEG frames) is decodable on ESP32 at modest resolutions. Server transcodes
incoming videos to MJPEG before delivery.

Typical specs:
- 320×240 @ 15fps MJPEG
- ~50KB per frame, ~750KB per second
- Audio track stripped + sent separately as MP3 (synced via timestamp)

### TTS Pipeline (Server-Side)

```
Karim types text in app
         │
         ▼
Server receives text
         │
         ▼
Server runs TTS engine (e.g., ElevenLabs, Google TTS, Coqui)
         │
         ▼
Audio file saved (MP3, ~64kbps for size, mono)
         │
         ▼
Audio URL added to content queue
         │
         ▼
Device polls, downloads MP3 to SD card
         │
         ▼
Device plays via I2S MP3 decoder library + MAX98357A
```

The character has a "voice" -- a specific TTS voice ID is chosen and reused for all
messages so the character has consistent personality.

### Content Queue Behavior

If Karim sends 5 messages while the device is asleep, all 5 sit in the server queue.
At the next poll (max 5 min later), the device downloads all 5 in one wake cycle and
plays them sequentially. This is more efficient than waking up 5 separate times.

After playback, the device acknowledges to the server (the queue is cleared).

---

## 12. Firmware Implementation Patterns

### SPI Bus Configuration (Two Separate Instances)

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

### I2S Setup for MAX98357A

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

### Battery Monitoring (GPIO35 ADC)

The LILYGO board has a voltage divider on GPIO35 that scales the battery voltage to fit
the ESP32 ADC range (0-3.3V). Typical divider: 100K + 100K (ratio 1:2).

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

### Modem AT Command Patterns

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

### Display Sleep/Wake (ILI9488)

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

### LiPo Over-Discharge Protection (Critical)

LiPo cells without protection can be permanently damaged below ~3.0V. Even though the
LILYGO board may have basic protection, we should be conservative in firmware:

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

---

## 13. Server API Specification

### Endpoints

| Method | Path | Purpose |
|:------:|------|---------|
| POST | `/api/content` | Karim's app pushes new content |
| GET | `/api/device/{id}/pending` | Device polls for queued content + flags |
| GET | `/api/device/{id}/content/{contentId}` | Device downloads specific media file |
| POST | `/api/device/{id}/status` | Device reports battery level |
| POST | `/api/device/{id}/location` | Device reports GPS coordinates |
| POST | `/api/content/{contentId}/ack` | Device acknowledges content played |

### Example: Device Polls

```http
GET /api/device/karim-device-1/pending HTTP/1.1
Authorization: Bearer <device_token>

200 OK
{
  "pending": true,
  "gps_requested": false,
  "items": [
    {
      "id": "msg_001",
      "type": "text_with_animation",
      "animation": "happy_wave",
      "text": "Good morning! Hope you slept well.",
      "audio_url": "/api/device/karim-device-1/content/audio_001.mp3"
    },
    {
      "id": "msg_002",
      "type": "photo",
      "photo_url": "/api/device/karim-device-1/content/photo_002.jpg",
      "caption": "I saw this and thought of you",
      "caption_audio_url": "/api/device/karim-device-1/content/audio_002.mp3"
    }
  ]
}
```

### Example: Device Reports Status

```http
POST /api/device/karim-device-1/status HTTP/1.1
Content-Type: application/json
Authorization: Bearer <device_token>

{
  "battery_voltage": 3.94,
  "battery_percent": 74,
  "uptime_seconds": 1234,
  "last_wake_reason": "timer",
  "free_sd_mb": 7800
}
```

### Example: Device Reports Location

```http
POST /api/device/karim-device-1/location HTTP/1.1
Content-Type: application/json

{
  "lat": 25.276987,
  "lon": 55.296249,
  "accuracy_m": 12,
  "speed_kmh": 0,
  "altitude_m": 8,
  "fixed_at": "2026-04-20T14:32:11Z"
}
```

### Server Responsibilities

1. **Authentication** -- device has a token, Karim's app has a separate token
2. **Content queue** -- accumulate messages until device picks them up
3. **TTS generation** -- on text submission, generate MP3 via TTS API, store URL
4. **Media storage** -- accept Karim's photos/videos, transcode video to MJPEG, serve files
5. **Acknowledgment tracking** -- mark items as delivered when device confirms
6. **GPS logging** -- store location history for Karim to see in app
7. **Battery history** -- track battery level over time (Karim can see if she's charging)

---

## 14. Karim's React Native App

### Screens

**Home / Send screen:**
- Text input with character emotion picker (happy, sad, shy, excited, sleepy)
- "Send text" button (server generates TTS automatically)
- Photo picker (gallery or camera)
- Audio recorder (record voice memo, server runs through cute character voice filter or just plays as-is)
- Video picker (gallery or camera)
- Music picker (from device's library)

**Status screen:**
- Device battery level (gauge)
- Last seen (when was last poll)
- Battery history (chart)

**Location screen:**
- Map with current location pin
- "Request location" button (sets gps_requested flag, returns within ~5-7 min)
- Location history with timestamps
- Patterns: "She visited [place] for [duration]" if Karim wants to be creepy/poetic

**Queue screen:**
- See pending messages waiting for next poll
- Cancel/edit pending messages

**Settings:**
- Polling interval (default 5 min, can change)
- Character voice selection (affects TTS voice)
- Connection status (is device reachable?)

### Tech Stack

- **React Native Expo** (already chosen)
- Backend communication via HTTPS REST
- Image picker, audio recorder from Expo libraries
- Map view (e.g., react-native-maps) for location screen

---

## 15. Problems Found & Solutions

| # | Problem | Status | Solution | Cost |
|---|---------|:------:|----------|:----:|
| 1 | Pin shortage (initially needed 14 GPIOs, only 8 free) | SOLVED | Drop touch (-4) + hardwire RST (-1) + use all 8 free GPIOs | AED 0 |
| 2 | Boot pin conflicts | SOLVED | Pin assignment avoids GPIO 2, 12, 15. Uses GPIO0 carefully (RTC + pull-up) | AED 0 |
| 3 | SPI bus conflict (display vs SD) | SOLVED | Display on VSPI, SD on HSPI -- separate hardware buses | AED 0 |
| 4 | Battery life too short with 18650 | SOLVED | Replace with 8000mAh LiPo (>2× capacity) | AED 45 |
| 5 | A7670 modem 2A current spikes | PLANNED | Bulk capacitor 1000-2200µF across battery | AED ~5 |
| 6 | 18650 holder occupies space, low capacity | SOLVED | Desolder, wire LiPo directly to B+/B- | AED 0 |
| 7 | Display 14-pin connector mostly unused | SOLVED | Only connect 7 pins (skip touch + MISO) | AED 0 |
| 8 | Mono vs stereo audio | SOLVED | MAX98357A SD pin to 3.3V = L+R mono mix at 15dB | AED 0 |
| 9 | Wake-up method (SMS vs timer) | DECIDED | Timer 5-min + DTR sleep -- simpler, no SMS gateway | AED 0 |
| 10 | Backlight drain in deep sleep (worst issue) | SOLVED | 10K pull-down resistor on GPIO21 to GND | AED 0 |
| 11 | GPIO21 not RTC GPIO | SOLVED | Pull-down resistor handles it, no need for gpio_hold | AED 0 |
| 12 | GPIO0 boot safety with I2S | SOLVED | Set HIGH + gpio_hold (GPIO0 IS RTC), pull-up backup | AED 0 |
| 13 | Display draws current in idle | PLANNED | Send ILI9488 sleep cmd 0x10 before deep sleep | AED 0 |
| 14 | LiPo over-discharge damages cell | PLANNED | Firmware monitors GPIO35 ADC, force sleep <3.3V | AED 0 |
| 15 | Modem keeps drawing power between polls | SOLVED | DTR sleep mode (~3mA vs ~20mA awake) | AED 0 |
| 16 | Battery life initial estimate was wrong | CORRECTED | Re-calculated: ~27 days @ 5 msgs, ~23 days @ 7 msgs | -- |

**Total extra cost beyond purchased parts:** AED 0-15 (just for optional bulk cap)

---

## 16. Critical Gotchas & Pitfalls

These are the things that would silently break the project if missed:

### Gotcha 1: BOARD_POWERON_PIN (GPIO12) Reset Loop
On battery power, if GPIO12 is not driven HIGH immediately after boot, the board will
reset in an endless loop. The very first thing in `setup()` after `Serial.begin()` must be:
```c
pinMode(BOARD_POWERON_PIN, OUTPUT);
digitalWrite(BOARD_POWERON_PIN, HIGH);
```

### Gotcha 2: GPIO12 Also Powers SD Card
GPIO12 is shared between modem power AND SD card power. You cannot turn off the SD card
without killing the modem too. We don't try to.

### Gotcha 3: A7670E vs A7670G (Critical Verification)
- **A7670E** -- has built-in GPS inside the modem chip. GPIO 19, 21, 22, 23 are FREE.
- **A7670G** -- uses an external L76K GPS chip that occupies GPIO 19, 21, 22, 23.

If you accidentally received the A7670G version, the L76K module is physically soldered
on the board and using the EXACT same pins we want for display + audio. Everything breaks.

**Verification:** Visual inspection. Look for a small chip labeled L76K near the modem.
If present, you have the wrong board.

### Gotcha 4: GPIO21/22 May Have I2C Pull-ups
The board defines GPIO21 as I2C SDA and GPIO22 as I2C SCL (in the SimShield section).
If the main board has built-in I2C pull-up resistors (typically 4.7K to 3.3V), they would
fight our 10K pull-down on GPIO21/BL.

**Verification:** When the board arrives, with nothing connected, measure voltage on
GPIO21 and GPIO22. If ~3.3V, there are pull-ups. If ~0V or floating, we're good.

If pull-ups exist: the 10K pull-down still wins for our purposes (4.7K pull-up + 10K
pull-down forms a divider, GPIO21 sits at ~2.25V which is HIGH from the display's
perspective -- backlight on during sleep). We'd need a stronger pull-down (1K) to
override, but that draws 3.3mA when GPIO21 is HIGH (not ideal). Better approach: cut
the I2C pull-up trace if present, or use a transistor on the BL line.

### Gotcha 5: SPI Initialization with -1 for Unused Pins
When initializing VSPI for the display, we don't use MISO (display write-only) or the
default CS pin (we manage CS manually as GPIO17). If we don't pass -1 for unused pins,
the SPI driver might claim GPIO 19 (default VSPI MISO) which we use for I2S BCLK.

```c
displaySPI->begin(18, -1, 23, -1);   // SCK, MISO=none, MOSI, CS=manual
```

### Gotcha 6: Modem Cannot Sleep on USB-C Power
When USB-C is connected, VBUS keeps the modem powered and DTR sleep won't actually save
power. This is fine because USB power is unlimited. But verify behavior with a multimeter:
when USB is plugged in, total current may be ~30-50mA instead of ~3mA.

### Gotcha 7: Display Sleep-Out Needs 120ms
After sending ILI9488 Sleep Out command (0x11), the controller needs **120ms** before
accepting any other commands. Sending a draw command too soon = corrupted display.

### Gotcha 8: SMS Over SGs (Even Though We Ditched SMS)
Just so we know: if we ever fall back to SMS wake-up, the operator must support SMS
delivery while modem is in 4G mode ("SMS over SGs"). Most modern operators do, but
verify with the chosen SIM card.

### Gotcha 9: I2S LRC on GPIO0 -- Don't Press Boot Button
The Boot button on the LILYGO board connects GPIO0 to GND when pressed. While audio is
playing, this would cause an audio glitch. Since the device is in a sealed transparent
case, the user can't press it. But during development/testing, be aware: don't press
Boot during audio playback.

### Gotcha 10: Cold Modem Boot is Slow (10-30 sec)
On first boot or after a full power cycle, the modem takes 10-30 seconds to register to
the network ("SMS DONE" event). This is a one-time cost. After that, DTR sleep keeps the
registration alive across thousands of poll cycles.

### Gotcha 11: GPIO34, 35, 36, 39 are Input Only
These pins cannot be used as outputs. They have no internal pull-up/pull-down. They're
useful for buttons (with external pull-up) or analog reads, not for driving anything.

### Gotcha 12: SD Card and Display Cannot Share SPI Bus
Even though both are SPI, they MUST be on separate buses (HSPI vs VSPI) because:
1. They're at different speeds (display fast, SD card slower)
2. Mutex-locking the bus would slow everything down
3. Their CS pins would need careful coordination

ESP32 has two hardware SPI buses (HSPI and VSPI). Use both.

### Gotcha 13: 8000mAh LiPo Charging Time
With a typical 1A charger (TP4056), 8000mAh takes ~8-9 hours to fully charge (CC + CV).
She must charge overnight, can't fast-charge.

### Gotcha 14: LiPo Cells Need Over-Discharge Protection
Below 3.0V, the LiPo can be permanently damaged. Set firmware cutoff at 3.3V to be safe.
Check if the LILYGO board has an under-voltage protection IC. If not, firmware is the
only line of defense.

---

## 17. Pre-Build Verification Checklist

Before any soldering or wiring, verify these when the LILYGO board arrives:

- [ ] **Board variant:** Confirm A7670E (built-in GPS) NOT A7670G (external L76K)
  - Look for any extra chip near the modem labeled "L76K" -- if present, wrong board
- [ ] **Board version:** Should be R2 (latest revision). Check silkscreen.
- [ ] **GPIO21/22 voltage:** With nothing connected, measure GPIO21 and GPIO22 to GND.
  - If ~0V or floating: no I2C pull-ups, our pull-down works fine
  - If ~3.3V: there are pull-ups, may need to cut traces or use transistor for BL
- [ ] **Display pinout:** Read the silkscreen on the 14-pin connector
  - Confirm pins 1-8 are: VCC, GND, CS, RST, DC, MOSI, SCK, BL
  - Confirm pins 10-14 are touch (T_*)
  - If different, adjust wiring accordingly
- [ ] **SD card slot:** Test with a formatted micro SD, run the SDCard.ino example
- [ ] **SIM card:** Insert SIM, verify network registration, test SMS receive (fallback option)
- [ ] **Battery charger current:** Test with multimeter while charging --
  current should be ~1A. If higher (rare), reduce by changing the program resistor on the charger IC
- [ ] **USB-C boot:** Verify board boots from USB-C alone (no battery)
- [ ] **Battery boot:** Verify board boots from battery alone (no USB)

Once verified, proceed with hardware modifications:

1. Desolder 18650 holder
2. Solder LiPo wires to B+/B- pads
3. Solder bulk capacitor across B+/B-
4. Build display RST RC circuit (10K + 100nF)
5. Solder 10K pull-down resistor between GPIO21 and GND
6. Wire display (7 connections)
7. Wire MAX98357A (6 connections)
8. Wire speaker (2 connections)

Then flash firmware and test step-by-step.

---

## 18. What's Still Not Decided

### Design

- [ ] **Pixel art character design** -- Karim's choice (he said "don't worry about it for now")
- [ ] **Character name** -- does it introduce itself? "Hi, I'm [name]!"
- [ ] **Character voice** -- pick a TTS voice ID for consistency
- [ ] **Animation library** -- how many emotions? (happy, sad, shy, sleepy, excited, etc.)
- [ ] **Color palette** -- pixel art typically uses limited colors; pick a cohesive set

### Server Choices

- [ ] **Server tech stack** -- Node.js? Python (FastAPI)? Go? (User said "easy part")
- [ ] **Hosting** -- VPS? Vercel? Railway? AWS?
- [ ] **TTS service** -- ElevenLabs (best voices, costs money), Google TTS, OpenAI, Coqui (free, runs on server)
- [ ] **Media storage** -- S3? local disk? CDN?
- [ ] **Database** -- Postgres? SQLite? for content queue and history

### Enclosure

- [ ] **Enclosure dimensions** -- exact size based on assembled hardware
- [ ] **Manufacturing method** -- 3D print? Laser-cut acrylic? Off-the-shelf transparent box?
- [ ] **Mounting** -- how is the LiPo secured? How is the display mounted to look good?
- [ ] **Charging port access** -- USB-C must be reachable but look intentional, not hacked

### UX Details

- [ ] **First boot greeting** -- exact animation + speech for the magical first impression
- [ ] **Low battery animations** -- what does "sleepy" look like? "Feed me"?
- [ ] **Charging animation** -- happy when she plugs in?
- [ ] **Idle behavior** -- does the character ever appear briefly to say hi when nothing is sent?
  - Risk: she might think it's "alive" if it talks unprompted
  - Benefit: feels less robotic
- [ ] **Volume control** -- can Karim set volume from his app?

### Technical Details

- [ ] **Video codec/format** -- MJPEG specifics (resolution, frame rate, file size)
- [ ] **TTS sample rate** -- 16kHz is sufficient for voice; 22kHz is nicer
- [ ] **Polling interval flexibility** -- always 5 min? Adapt to time of day?
- [ ] **Charge rate verification** -- exactly what's the charger IC and current setting?

### Operational

- [ ] **SIM card / data plan** -- which operator? (User said "don't worry about it")
- [ ] **Device commissioning** -- how is the device token / device ID set up?
- [ ] **Crash recovery** -- if firmware crashes, does it restart cleanly?
- [ ] **OTA updates** -- can firmware be updated remotely? (Useful for bug fixes after gift)

---

## 19. Reference Files in Repo

The official LILYGO repo (`docs/`) is the source of truth for board details:

| File | Contains |
|------|----------|
| `docs/examples/ATdebug/utilities.h` | Pin definitions for all board variants. **Lines 88-145** for our T-A7670 board. |
| `docs/examples/WakeupByRingOrSMS/WakeupByRingOrSMS.ino` | Reference for SMS wake-up + gpio_hold patterns (we use the patterns even though we don't use SMS) |
| `docs/examples/WakeupByRingOrSMS_SendLocation/WakeupByRingOrSMS_SendLocation.ino` | **Starter code for our GPS feature.** Receives a flag, gets GPS, reports it. |
| `docs/examples/DeepSleep/DeepSleep.ino` | Timer-based deep sleep example. **Reports 157µA** when modem is fully off. |
| `docs/examples/ModemSleep/ModemSleep.ino` | DTR sleep control patterns. **Critical for our power optimization.** |
| `docs/examples/ModemPlayMusic/ModemPlayMusic.ino` | A7670 internal audio (SPK+/SPK-). Fallback option, not primary. |
| `docs/examples/SDCard/SDCard.ino` | SD card init + file ops. Pin definitions for HSPI bus. |
| `docs/datasheet/A76XX/A76XX_Series_AT_Command_Manual_V1.12.pdf` | A7670 AT command reference manual |

### Key Examples to Combine for Our Firmware

Our firmware combines patterns from THREE official examples:

1. **From `DeepSleep.ino`:** the `esp_sleep_enable_timer_wakeup()` + `esp_deep_sleep_start()` pattern
2. **From `ModemSleep.ino`:** the DTR sleep mode + `gpio_hold_en()` pattern for the modem
3. **From `WakeupByRingOrSMS_SendLocation.ino`:** the GPS fix + report pattern

We do NOT use `WakeupByRingOrSMS.ino`'s ext0 wake-up (we ditched SMS). We just use it as
a reference for `gpio_hold_en()` patterns.

### Repo Notes

The `docs/` folder is the LILYGO repo's contents inlined (its embedded git was removed
to avoid nested-git issues -- see commit `9805245`). Treat it as documentation/reference,
not as a tracked submodule.

---

## 20. Glossary

| Term | Meaning |
|------|---------|
| **A7670E** | The 4G LTE Cat1 modem chip. Has built-in GPS. Communicates via UART AT commands. |
| **AT command** | Text-based commands to the modem (e.g., `AT+CGPS=1` to enable GPS) |
| **APN** | Access Point Name. Identifies which 4G data gateway to use (e.g., "internet" for some operators). Operator-specific. |
| **BCLK** | Bit Clock for I2S audio. One pulse per bit. |
| **BL** | Backlight (display LED illumination) |
| **CS** | Chip Select for SPI. LOW = device active, HIGH = device ignored. |
| **DC** | Data/Command select for display SPI. LOW = command, HIGH = data. |
| **Deep sleep** | ESP32 power state where most of the chip is off. Wakes from timer or RTC GPIO. |
| **DIN** | Data In for I2S audio (the actual audio samples) |
| **DTR** | Data Terminal Ready. A modem control pin. HIGH = sleep, LOW = active. |
| **ext0 wake** | ESP32 wake source. Triggers on a single GPIO pin going to a specified level. Requires RTC GPIO. |
| **FT6236** | Capacitive touch controller (we don't use it) |
| **gpio_hold_en()** | ESP-IDF function to maintain a GPIO output level through sleep. Only persists in deep sleep for RTC GPIOs. |
| **HSPI** | One of ESP32's two hardware SPI peripherals. Used by SD card on this board. |
| **I2S** | Digital audio bus standard. Three wires: BCLK, LRC, DIN. |
| **ILI9488** | The display controller chip. 480×320 RGB, SPI interface. |
| **L76K** | An external GPS chip. Used on A7670G boards (NOT our A7670E). |
| **LiPo** | Lithium polymer battery. Pouch cell form factor. 3.7V nominal. |
| **LRC** | Left-Right Clock for I2S. Also called WS (Word Select). HIGH = right channel, LOW = left. |
| **MAX98357A** | I2S Class D audio amplifier chip. Built-in DAC. Drives speakers up to 3W. |
| **MISO** | Master In Slave Out. SPI data line from peripheral to MCU. We don't use it (write-only display). |
| **MJPEG** | Motion JPEG. Video format = sequence of JPEG frames. ESP32 can decode in real time. |
| **MOSI** | Master Out Slave In. SPI data line from MCU to peripheral. |
| **MUX/PSRAM** | Pseudo-Static RAM. The ESP32 has 8MB external PSRAM for buffers. |
| **PWRKEY** | Power Key. Modem control pin. Pulse to power on/off. |
| **RI** | Ring Indicator. Modem pin that pulses on incoming call/SMS. |
| **RTC GPIO** | A subset of ESP32 GPIOs that remain powered during deep sleep. Required for ext0 wake-up and persistent gpio_hold. |
| **SCK** | Serial Clock for SPI |
| **SCL** | Serial Clock for I2C |
| **SD pin (MAX98357A)** | Shutdown / channel select pin. 3.3V = mono mix, GND = shutdown. |
| **SDA** | Serial Data for I2C |
| **SMS over SGs** | A network feature allowing SMS delivery while phone is on 4G/LTE |
| **TinyGSM** | Arduino library for cellular modem control via AT commands |
| **TP4056** | Common single-cell lithium charging IC. Likely on the LILYGO board. |
| **TTS** | Text-to-Speech. Server converts Karim's text to spoken audio file. |
| **VSPI** | The other ESP32 hardware SPI bus. We use it for the display. |
| **WS** | Word Select for I2S. Same as LRC. |

---

**End of knowledge.md**

This document is the single source of truth. When in doubt, read this first.
