# Gotchas

> Things that would silently break the project if missed. Read before assembling.

## Gotcha 1: BOARD_POWERON_PIN (GPIO12) Reset Loop

On battery power, if GPIO12 is not driven HIGH immediately after boot, the board will reset in an endless loop. The very first thing in `setup()` after `Serial.begin()` must be:

```c
pinMode(BOARD_POWERON_PIN, OUTPUT);
digitalWrite(BOARD_POWERON_PIN, HIGH);
```

## Gotcha 2: GPIO12 Also Powers SD Card

GPIO12 is shared between modem power AND SD card power. You cannot turn off the SD card without killing the modem too. We don't try to.

## Gotcha 3: A7670E vs A7670G (Critical Verification)

- **A7670E** — has built-in GPS inside the modem chip. GPIO 19, 21, 22, 23 are FREE.
- **A7670G** — uses an external L76K GPS chip that occupies GPIO 19, 21, 22, 23.

If you accidentally received the A7670G version, the L76K module is physically soldered on the board and using the EXACT same pins we want for display + audio. **Everything breaks.**

**Verification:** Visual inspection. Look for a small chip labeled L76K near the modem. If present, you have the wrong board. See reference photos: [`screenshots/lilygo-board/19-a7670e-vs-g-note.png`](screenshots/lilygo-board/19-a7670e-vs-g-note.png), [`screenshots/lilygo-board/15-gps-interface-e-vs-g.png`](screenshots/lilygo-board/15-gps-interface-e-vs-g.png).

## Gotcha 4: GPIO21/22 May Have I2C Pull-ups

The board defines GPIO21 as I2C SDA and GPIO22 as I2C SCL (in the SimShield section). If the main board has built-in I2C pull-up resistors (typically 4.7K to 3.3V), they would fight our 10K pull-down on GPIO21/BL.

**Verification:** When the board arrives, with nothing connected, measure voltage on GPIO21 and GPIO22. If ~3.3V, there are pull-ups. If ~0V or floating, we're good.

If pull-ups exist: the 10K pull-down still wins for our purposes (4.7K pull-up + 10K pull-down forms a divider, GPIO21 sits at ~2.25V which is HIGH from the display's perspective — backlight on during sleep). We'd need a stronger pull-down (1K) to override, but that draws 3.3mA when GPIO21 is HIGH (not ideal). Better approach: cut the I2C pull-up trace if present, or use a transistor on the BL line.

## Gotcha 5: SPI Initialization with -1 for Unused Pins

When initializing VSPI for the display, we don't use MISO (display write-only) or the default CS pin (we manage CS manually as GPIO17). If we don't pass -1 for unused pins, the SPI driver might claim GPIO 19 (default VSPI MISO) which we use for I2S BCLK.

```c
displaySPI->begin(18, -1, 23, -1);   // SCK, MISO=none, MOSI, CS=manual
```

## Gotcha 6: Modem Cannot Sleep on USB-C Power

When USB-C is connected, VBUS keeps the modem powered and DTR sleep won't actually save power. This is fine because USB power is unlimited. But verify behavior with a multimeter: when USB is plugged in, total current may be ~30–50mA instead of ~3mA.

## Gotcha 7: Display Sleep-Out Needs 120ms

After sending ILI9488 Sleep Out command (0x11), the controller needs **120ms** before accepting any other commands. Sending a draw command too soon = corrupted display.

## Gotcha 8: SMS Over SGs (Even Though We Ditched SMS)

Just so we know: if we ever fall back to SMS wake-up, the operator must support SMS delivery while modem is in 4G mode ("SMS over SGs"). Most modern operators do, but verify with the chosen SIM card.

## Gotcha 9: I2S LRC on GPIO0 — Don't Press Boot Button

The Boot button on the LILYGO board connects GPIO0 to GND when pressed. While audio is playing, this would cause an audio glitch. Since the device is in a sealed transparent case, the user can't press it. But during development/testing, be aware: don't press Boot during audio playback.

## Gotcha 10: Cold Modem Boot is Slow (10–30 sec)

On first boot or after a full power cycle, the modem takes 10–30 seconds to register to the network ("SMS DONE" event). This is a one-time cost. After that, DTR sleep keeps the registration alive across thousands of poll cycles.

## Gotcha 11: GPIO34, 35, 36, 39 are Input Only

These pins cannot be used as outputs. They have no internal pull-up/pull-down. They're useful for buttons (with external pull-up) or analog reads, not for driving anything.

## Gotcha 12: SD Card and Display Cannot Share SPI Bus

Even though both are SPI, they MUST be on separate buses (HSPI vs VSPI) because:

1. They're at different speeds (display fast, SD card slower)
2. Mutex-locking the bus would slow everything down
3. Their CS pins would need careful coordination

ESP32 has two hardware SPI buses (HSPI and VSPI). Use both.

## Gotcha 13: 8000mAh LiPo Charging Time

With a typical 1A charger (TP4056), 8000mAh takes ~8–9 hours to fully charge (CC + CV). She must charge overnight, can't fast-charge.

## Gotcha 14: LiPo Cells Need Over-Discharge Protection

Below 3.0V, the LiPo can be permanently damaged. Set firmware cutoff at 3.3V to be safe. Check if the LILYGO board has an under-voltage protection IC. If not, firmware is the only line of defense.

## Related pages

- [Pre-build checklist](pre-build-checklist.md) — verify these before soldering
- [Pin assignment](pin-assignment.md) — explains why each pin choice avoids these
- [Hardware modifications](hardware-modifications.md) — physical fixes for #4 (pull-ups), #14 (LiPo)
- [Sleep/wake cycle](sleep-wake-cycle.md) — implements the fix for #1, #6, #7
- [Problems and solutions](problems-and-solutions.md) — the planning-stage version of these
