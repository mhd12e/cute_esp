# Pin Assignment

> Verified pin map for LILYGO T-SIM A7670E R2.
> **Source of truth:** the board schematic at [`lilygo-reference/schematic/esp32/T-A7670X-V1.4.pdf`](lilygo-reference/schematic/esp32/T-A7670X-V1.4.pdf), cross-checked against `lilygo-reference/examples/ATdebug/utilities.h` lines 88–145 (the `LILYGO_T_A7670` block) and the ESP32-WROVER datasheet.

## Pins Used Internally by Board (DO NOT TOUCH)

| GPIO | Define | Function | Notes |
|:----:|--------|----------|-------|
| 4 | BOARD_PWRKEY_PIN | Modem power key | Pulse to power on/off (100ms) |
| 5 | MODEM_RESET_PIN | Modem reset | Active HIGH (MODEM_RESET_LEVEL = HIGH) |
| 12 | BOARD_POWERON_PIN | Modem & SD power | **MUST be HIGH for battery operation** |
| 25 | MODEM_DTR_PIN | Modem DTR (sleep ctrl) | HIGH = sleep, LOW = wake |
| 26 | MODEM_TX_PIN | UART TX to modem | Serial1 |
| 27 | MODEM_RX_PIN | UART RX from modem | Serial1 |
| 33 | MODEM_RING_PIN | RI pin (incoming SMS/call) | Hardwired to modem RI even in timer-poll mode |
| 35 | BOARD_BAT_ADC_PIN | Battery voltage divider | Input only — analog read |
| 36 | BOARD_SOLAR_ADC_PIN | Solar input ADC | Input only, V1.4 hardwires solar pad to this pin |
| 2 | BOARD_MISO_PIN | SD MISO | HSPI bus |
| 13 | BOARD_SD_CS_PIN | SD chip select | HSPI bus |
| 14 | BOARD_SCK_PIN | SD SCK | HSPI bus |
| 15 | BOARD_MOSI_PIN | SD MOSI | HSPI bus |

## Pins NOT Available — PSRAM Inside the WROVER Module

| GPIO | Status | Why |
|:----:|--------|-----|
| 16 | **BLOCKED** | ESP32-WROVER ties this to PSRAM CS internally. **Not broken out to any header.** |
| 17 | **BLOCKED** | ESP32-WROVER ties this to PSRAM CLK internally. **Not broken out to any header.** |

These two pins look "free" if you only read `utilities.h`, but the WROVER datasheet is explicit: GPIO 16 and 17 are reserved for embedded PSRAM and not recommended for other uses. The schematic confirms they are not present on either header (PX or PY). Don't try to use them. See [gotchas](gotchas.md#gotcha-gpio-16-and-17-are-psram-not-free).

## Pins NOT Available — UART0 (USB Serial Debug)

| GPIO | Status | Notes |
|:----:|--------|-------|
| 1 | UART0 TX | Used for USB-serial debug. Sacrificing means no console output during development. Treat as unavailable. |
| 3 | UART0 RX | Same as above. |

## Pins Free on Headers (output-capable)

| GPIO | Header | RTC GPIO? | Notes |
|:----:|:------:|:---------:|-------|
| 0 | PX | YES | Boot-sensitive (must be HIGH at boot); internal pull-up |
| 18 | PX | No | VSPI default SCK |
| 19 | PX | No | VSPI default MISO (we don't use MISO) |
| 21 | PX | No | Board labels as I2C SDA (no actual I2C device) |
| 22 | PX | No | Board labels as I2C SCL (no actual I2C device) |
| 23 | PX | No | VSPI default MOSI |
| **32** | **PY pin 14** | **YES** | RTC, ADC1, DAC2, no boot-strap function — clean pin |

**Total free output GPIOs: 7.**

## Pins Free (input-only)

| GPIO | Header | Notes |
|:----:|:------:|-------|
| 34 | PY pin 12 | Input only (no internal pull-up/down), ADC1 |
| 39 | PX pin 10 | Input only (no internal pull-up/down), ADC1 |

## Pin Budget

We need **8** outputs (display: SCK, MOSI, BL, CS, DC + I2S: BCLK, LRC, DIN). We have **7** free output GPIOs. Pin allocation is therefore one short.

**Resolution:** the display sits alone on the VSPI bus (SD card lives on HSPI), so its CS doesn't need to be toggled. We tie the display's CS pin directly to GND on the wiring side and let the display stay permanently selected. That eliminates the need for a CS GPIO and the budget closes at 7/7.

## Final Pin Assignments (7 of 7 output GPIOs used)

| GPIO | Assignment | Why this pin |
|:----:|------------|-------------|
| 0 | I2S LRC | RTC GPIO — can hold HIGH through deep sleep for safe boot |
| 18 | Display SCK | VSPI hardware SPI default |
| 19 | I2S BCLK | Free, output-capable |
| 21 | Display BL | Last general-purpose pin (with 10K pull-down resistor — see [hardware-modifications](hardware-modifications.md)) |
| 22 | I2S DIN | Free, output-capable |
| 23 | Display MOSI | VSPI hardware SPI default |
| 32 | Display DC | RTC GPIO, no boot-strap function, on PY header pin 14 |

## Hardwired (No GPIO)

| Signal | Wiring | Notes |
|--------|--------|-------|
| Display RST | 3.3V via 10K + 100nF cap to GND | Auto power-on reset |
| **Display CS** | **Tied directly to GND** | Permanently selected. Display alone on VSPI, so this is safe. |
| MAX98357A SD pin | Tied to 3.3V | L+R mono mix mode, 15dB gain |

## Why GPIO0 for I2S LRC is Safe

GPIO0 must be **HIGH at boot** or ESP32 enters flash/download mode. Our usage is safe:

1. **Boot phase:** I2S peripheral isn't started yet. GPIO0's internal pull-up keeps it HIGH. ✅
2. **Active operation:** I2S toggles LRC. Boot is long done. Doesn't matter. ✅
3. **Pre-sleep:** Firmware stops I2S, sets GPIO0 HIGH, calls `gpio_hold_en()`. GPIO0 IS an RTC GPIO so the hold persists through deep sleep. ✅
4. **Wake from sleep:** GPIO0 still HIGH from the hold. Boot ROM reads HIGH. Normal boot. ✅
5. **Crash recovery:** Even if the hold is somehow lost, the internal pull-up (~45K) pulls GPIO0 HIGH. Normal boot. ✅

The only theoretical risk: someone presses the physical Boot button on the board while audio is playing. This causes a brief audio glitch. Since the device is in a sealed transparent case, this won't happen.

## Why GPIO 32 is the Right Home for Display DC

GPIO 32 has the cleanest profile of any pin we have left:

- **No boot-strap function** — GPIO 32 doesn't influence boot mode at all. Free to drive HIGH or LOW from the moment the chip wakes.
- **RTC GPIO** — `gpio_hold_en(32)` persists through deep sleep, so we can park DC in a known state during sleep. (We don't strictly need to, since the display is in `0x10` Sleep In and ignores commands, but it's a nice-to-have.)
- **No alternate function we care about** — GPIO 32 can be XTAL_32K_P input *if* a 32 kHz crystal is added; this board has no such crystal, so the pin is free.
- **Already exposed on header PY pin 14** — no soldering tricks needed.

## Why GPIO21 Backlight Needs External Pull-Down

GPIO21 is NOT in the ESP32 RTC GPIO list. ESP32 RTC GPIOs are: **0, 2, 4, 12, 13, 14, 15, 25, 26, 27, 32, 33, 34, 35, 36, 39**.

`gpio_hold_en(21)` works only in light sleep, not deep sleep. When ESP32 enters deep sleep, GPIO21 becomes high-impedance. If the display BL line has any pull-up (most do), the backlight turns on. Without intervention this drains 80–150mA continuously during "sleep" — catastrophic.

**The 10K pull-down resistor** between GPIO21 and GND solves it: when GPIO21 floats, the resistor wins and pulls it LOW (backlight off). When ESP32 drives GPIO21 HIGH, it overrides the weak 10K easily (current sink: 3.3V/10K = 0.33mA, negligible).

See [hardware-modifications](hardware-modifications.md) for the wiring of this resistor.

## Sleep-Time Behavior of the Permanent CS

Tying display CS to GND means the display is "selected" 24/7. During ESP32 deep sleep:

- The display itself is in **Sleep In mode** (we send command `0x10` before sleeping). Per ILI9488 datasheet, in this mode the controller ignores all commands except `Sleep Out (0x11)`.
- SCK (GPIO 18) and MOSI (GPIO 23) are not RTC GPIOs, so they go high-impedance during deep sleep. The 10K-pull-down trick we use on GPIO 21 isn't strictly needed here because the display is ignoring input — but if you want belt-and-suspenders, a 10K pull-down on each of SCK and MOSI is harmless and removes any chance of spurious commands.

In practice, no extra pull-downs on SCK/MOSI are necessary. Verify on the bench.

## Spare Capacity

**0** spare output GPIOs. **2** input-only spares (GPIO 34, 39). No I2C bus available. If a future feature needs a new output (haptic motor, second LED, etc.), the only paths are: sacrifice UART0 (lose serial debug), add a port expander on VSPI/HSPI, or move the display to the modem's built-in audio (`SPK+`/`SPK-`) and reclaim the I2S pins.

## Related pages

- [Hardware modifications](hardware-modifications.md) — physical changes that this pin map depends on (battery swap, RST RC, BL pull-down, CS-to-GND)
- [Wiring diagram](wiring-diagram.md) — connections on a single page
- [Sleep/wake cycle](sleep-wake-cycle.md) — why GPIO0, GPIO21, GPIO32 need special care
- [Gotchas](gotchas.md) — PSRAM-blocked GPIO 16/17, boot pin pitfalls, pull-up conflicts
- [Problems and solutions](problems-and-solutions.md) — the planning-stage record of this pin discovery
- LILYGO pinout photos: [`screenshots/lilygo-board/06–08-pin-diagram-*.png`](screenshots/lilygo-board/) — note the marketing diagrams may misleadingly show GPIO 16/17 as accessible; the schematic is the truth
