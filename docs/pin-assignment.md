# Pin Assignment

> Verified pin map for LILYGO T-SIM A7670E R2. Source: `lilygo-reference/examples/ATdebug/utilities.h` lines 88–145 (the `LILYGO_T_A7670` block).

## Pins Used Internally by Board (DO NOT TOUCH)

| GPIO | Define | Function | Notes |
|:----:|--------|----------|-------|
| 4 | BOARD_PWRKEY_PIN | Modem power key | Pulse to power on/off (100ms) |
| 5 | MODEM_RESET_PIN | Modem reset | Active HIGH (MODEM_RESET_LEVEL = HIGH) |
| 12 | BOARD_POWERON_PIN | Modem & SD power | **MUST be HIGH for battery operation** |
| 25 | MODEM_DTR_PIN | Modem DTR (sleep ctrl) | HIGH = sleep, LOW = wake |
| 26 | MODEM_TX_PIN | UART TX to modem | Serial1 |
| 27 | MODEM_RX_PIN | UART RX from modem | Serial1 |
| 33 | MODEM_RING_PIN | RI pin (incoming SMS/call) | Reserved (not used in timer mode) |
| 35 | BOARD_BAT_ADC_PIN | Battery voltage divider | Input only — analog read |
| 36 | BOARD_SOLAR_ADC_PIN | Solar input ADC | Input only, v1.4 board only |
| 2 | BOARD_MISO_PIN | SD MISO | HSPI bus |
| 13 | BOARD_SD_CS_PIN | SD chip select | HSPI bus |
| 14 | BOARD_SCK_PIN | SD SCK | HSPI bus |
| 15 | BOARD_MOSI_PIN | SD MOSI | HSPI bus |

## Pins Free on Headers (output-capable)

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

## Pins Free (input-only)

| GPIO | Notes |
|:----:|-------|
| 34 | Input only (no internal pull-up/down) |
| 39 | Input only (no internal pull-up/down) |

## Final Pin Assignments (all 8 output GPIOs used)

| GPIO | Assignment | Why This Pin |
|:----:|------------|-------------|
| 16 | Display DC | Free, no boot issues |
| 17 | Display CS | Free, no boot issues |
| 18 | Display SCK | VSPI hardware SPI default |
| 23 | Display MOSI | VSPI hardware SPI default |
| 21 | Display BL | Last free pin (with 10K pull-down resistor) |
| 19 | I2S BCLK | Free, output capable |
| 0 | I2S LRC | RTC GPIO — can hold HIGH through deep sleep for safe boot |
| 22 | I2S DIN | Free, output capable |

## Hardwired (No GPIO)

| Signal | Wiring | Notes |
|--------|--------|-------|
| Display RST | 3.3V via 10K + 100nF cap to GND | Auto power-on reset |
| MAX98357A SD pin | Tied to 3.3V | L+R mono mix mode, 15dB gain |

## Why GPIO0 for I2S LRC is Safe

GPIO0 must be **HIGH at boot** or ESP32 enters flash/download mode. Our usage is safe:

1. **Boot phase:** I2S peripheral isn't started yet. GPIO0's internal pull-up keeps it HIGH. ✅
2. **Active operation:** I2S toggles LRC. Boot is long done. Doesn't matter. ✅
3. **Pre-sleep:** Firmware stops I2S, sets GPIO0 HIGH, calls `gpio_hold_en()`. GPIO0 IS an RTC GPIO so the hold persists through deep sleep. ✅
4. **Wake from sleep:** GPIO0 still HIGH from the hold. Boot ROM reads HIGH. Normal boot. ✅
5. **Crash recovery:** Even if the hold is somehow lost, the internal pull-up (~45K) pulls GPIO0 HIGH. Normal boot. ✅

The only theoretical risk: someone presses the physical Boot button on the board while audio is playing. This causes a brief audio glitch. Since the device is in a sealed transparent case, this won't happen.

## Why GPIO21 Backlight Needs External Pull-Down

GPIO21 is NOT in the ESP32 RTC GPIO list. ESP32 RTC GPIOs are: **0, 2, 4, 12, 13, 14, 15, 25, 26, 27, 32, 33, 34, 35, 36, 39**.

`gpio_hold_en(21)` works only in light sleep, not deep sleep. When ESP32 enters deep sleep, GPIO21 becomes high-impedance. If the display BL line has any pull-up (most do), the backlight turns on. Without intervention this drains 80–150mA continuously during "sleep" — catastrophic.

**The 10K pull-down resistor** between GPIO21 and GND solves it: when GPIO21 floats, the resistor wins and pulls it LOW (backlight off). When ESP32 drives GPIO21 HIGH, it overrides the weak 10K easily (current sink: 3.3V/10K = 0.33mA, negligible).

See [hardware modifications](hardware-modifications.md) for the wiring of this resistor.

## Related pages

- [Hardware modifications](hardware-modifications.md) — physical changes that this pin map depends on
- [Wiring diagram](wiring-diagram.md) — connections on a single page
- [Sleep/wake cycle](sleep-wake-cycle.md) — why GPIO0 and GPIO21 need special care
- [Gotchas](gotchas.md) — boot pin pitfalls and pull-up conflicts
- LILYGO pinout photos: [`screenshots/lilygo-board/06–08-pin-diagram-*.png`](screenshots/lilygo-board/)
