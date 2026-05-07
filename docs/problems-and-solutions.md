# Problems and Solutions

> Every problem found during planning and what we did about it.

| # | Problem | Status | Solution | Cost |
|---|---------|:------:|----------|:----:|
| 1 | Pin shortage (initially needed 14 GPIOs) | SOLVED | Drop touch (-4) + hardwire RST (-1) + tie CS to GND (-1) — see #17 below for the deeper PSRAM finding | AED 0 |
| 2 | Boot pin conflicts | SOLVED | Pin assignment avoids GPIO 2, 12, 15. Uses GPIO0 carefully (RTC + pull-up) | AED 0 |
| 3 | SPI bus conflict (display vs SD) | SOLVED | Display on VSPI, SD on HSPI — separate hardware buses | AED 0 |
| 4 | Battery life too short with 18650 | SOLVED | Replace with 8000mAh LiPo (>2× capacity) | AED 45 |
| 5 | A7670 modem 2A current spikes | PLANNED | Bulk capacitor 1000–2200µF across battery | AED ~5 |
| 6 | 18650 holder occupies space, low capacity | SOLVED | Desolder, wire LiPo directly to B+/B- | AED 0 |
| 7 | Display 14-pin connector mostly unused | SOLVED | Only connect 7 pins (skip touch + MISO) | AED 0 |
| 8 | Mono vs stereo audio | SOLVED | MAX98357A SD pin to 3.3V = L+R mono mix at 15dB | AED 0 |
| 9 | Wake-up method (SMS vs timer) | DECIDED | Timer 5-min + DTR sleep — simpler, no SMS gateway | AED 0 |
| 10 | Backlight drain in deep sleep (worst issue) | SOLVED | 10K pull-down resistor on GPIO21 to GND | AED 0 |
| 11 | GPIO21 not RTC GPIO | SOLVED | Pull-down resistor handles it, no need for `gpio_hold` | AED 0 |
| 12 | GPIO0 boot safety with I2S | SOLVED | Set HIGH + `gpio_hold` (GPIO0 IS RTC), pull-up backup | AED 0 |
| 13 | Display draws current in idle | PLANNED | Send ILI9488 sleep cmd 0x10 before deep sleep | AED 0 |
| 14 | LiPo over-discharge damages cell | PLANNED | Firmware monitors GPIO35 ADC, force sleep <3.3V | AED 0 |
| 15 | Modem keeps drawing power between polls | SOLVED | DTR sleep mode (~3mA vs ~20mA awake) | AED 0 |
| 16 | Battery life initial estimate was wrong | CORRECTED | Re-calculated: ~27 days @ 5 msgs, ~23 days @ 7 msgs | — |
| 17 | GPIO 16 / 17 are PSRAM (not free) — pin budget short by 1 | SOLVED | Tie display CS directly to GND (display alone on VSPI, no need to deselect); use newly-discovered free GPIO 32 for display DC | AED 0 |
| 18 | GPIO 32 missing from earlier docs | CORRECTED | Verified on schematic page 3 (PY pin 14); RTC, ADC1, no boot-strap function | — |

**Total extra cost beyond purchased parts:** AED 0–15 (just for optional bulk cap).

## Related pages

- [Gotchas](gotchas.md) — the "silent failure" version of these (things that won't show up as obvious problems)
- [Pre-build checklist](pre-build-checklist.md) — verify before assuming any of these are still solved
- [Hardware modifications](hardware-modifications.md) — mods that solve #4, #5, #6, #10
- [Pin assignment](pin-assignment.md) — solves #1, #2, #3, #11, #12
- [Sleep/wake cycle](sleep-wake-cycle.md) — implements #13, #14, #15
