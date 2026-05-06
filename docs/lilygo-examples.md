# LILYGO Examples Reference

> Which official LILYGO examples to crib from, and for what.

The official LILYGO repo (`docs/lilygo-reference/`) is the source of truth for board details:

| File | Contains |
|------|----------|
| [`lilygo-reference/examples/ATdebug/utilities.h`](lilygo-reference/examples/ATdebug/utilities.h) | Pin definitions for all board variants. **Lines 88–145** for our T-A7670 board. |
| [`lilygo-reference/examples/WakeupByRingOrSMS/WakeupByRingOrSMS.ino`](lilygo-reference/examples/WakeupByRingOrSMS/WakeupByRingOrSMS.ino) | Reference for SMS wake-up + `gpio_hold` patterns (we use the patterns even though we don't use SMS) |
| [`lilygo-reference/examples/WakeupByRingOrSMS_SendLocation/`](lilygo-reference/examples/WakeupByRingOrSMS_SendLocation/) | **Starter code for our GPS feature.** Receives a flag, gets GPS, reports it. |
| [`lilygo-reference/examples/DeepSleep/DeepSleep.ino`](lilygo-reference/examples/DeepSleep/DeepSleep.ino) | Timer-based deep sleep example. **Reports 157µA** when modem is fully off. |
| [`lilygo-reference/examples/ModemSleep/ModemSleep.ino`](lilygo-reference/examples/ModemSleep/ModemSleep.ino) | DTR sleep control patterns. **Critical for our power optimization.** |
| [`lilygo-reference/examples/ModemPlayMusic/ModemPlayMusic.ino`](lilygo-reference/examples/ModemPlayMusic/ModemPlayMusic.ino) | A7670 internal audio (SPK+/SPK-). Fallback option, not primary. |
| [`lilygo-reference/examples/SDCard/SDCard.ino`](lilygo-reference/examples/SDCard/SDCard.ino) | SD card init + file ops. Pin definitions for HSPI bus. |
| [`lilygo-reference/datasheet/A76XX/`](lilygo-reference/datasheet/A76XX/) | A7670 AT command reference manual (PDF) |

## Key Examples to Combine for Our Firmware

Our firmware combines patterns from THREE official examples:

1. **From `DeepSleep.ino`:** the `esp_sleep_enable_timer_wakeup()` + `esp_deep_sleep_start()` pattern
2. **From `ModemSleep.ino`:** the DTR sleep mode + `gpio_hold_en()` pattern for the modem
3. **From `WakeupByRingOrSMS_SendLocation.ino`:** the GPS fix + report pattern

We do NOT use `WakeupByRingOrSMS.ino`'s ext0 wake-up (we ditched SMS). We just use it as a reference for `gpio_hold_en()` patterns.

## Repo Notes

The `docs/lilygo-reference/` folder is the LILYGO repo's contents inlined (its embedded git was removed to avoid nested-git issues — see commit `9805245`). Treat it as documentation/reference, not as a tracked submodule.

## Related pages

- [Sleep/wake cycle](sleep-wake-cycle.md) — our implementation built from these examples
- [Firmware patterns](firmware-patterns.md) — extracted snippets ready to use
- [Pin assignment](pin-assignment.md) — pulls verified values from `utilities.h`
- [Pre-build checklist](pre-build-checklist.md) — `SDCard.ino` is one of the bring-up tests
