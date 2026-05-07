# Wiring Diagram

> Every connection on a single page.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                  CUTE ESP — COMPLETE WIRING                            ║
╚══════════════════════════════════════════════════════════════════════════╝

LILYGO T-A7670E R2 (top-level)
═══════════════════════════════
Internal pins (DO NOT WIRE):
  GPIO 4, 5, 12, 25, 26, 27, 33, 35, 36 — modem control / battery / solar sense
  GPIO 2, 13, 14, 15 — SD card (HSPI bus)

Not accessible (PSRAM-blocked, inside WROVER module):
  GPIO 16, 17 — DO NOT attempt to use, not on any header

Used pins (7):
  GPIO 18 -> Display SCK
  GPIO 23 -> Display MOSI
  GPIO 21 -> Display BL  (also: 10K resistor to GND)
  GPIO 32 -> Display DC  (on PY header pin 14)
  GPIO 19 -> MAX98357A BCLK
  GPIO  0 -> MAX98357A LRC
  GPIO 22 -> MAX98357A DIN

Hardwired (no GPIO):
  Display CS  -> GND directly (display permanently selected on VSPI)
  Display RST -> 3.3V via 10K + 100nF cap to GND (RC reset)
  MAX98357A SD pin -> 3.3V (mono L+R mix mode)

Power rails:
  3.3V -> Display VCC
  3.3V -> MAX98357A VIN
  3.3V -> MAX98357A SD pin
  3.3V -> Display RST circuit (10K resistor)
  GND  -> Display GND, Display CS, MAX98357A GND, RST cap, BL pull-down
```

## Display 14-Pin Connector

| Pin | Signal | Connect to | Wire color |
|:---:|--------|------------|-----------|
| 1 | VCC | 3.3V | Red |
| 2 | GND | GND | Black |
| 3 | CS | **GND directly (hardwire)** | Black (short jumper) |
| 4 | RST | (RC reset circuit) | hardwired |
| 5 | DC | **GPIO 32** | Yellow |
| 6 | MOSI | GPIO 23 | Green |
| 7 | SCK | GPIO 18 | Blue |
| 8 | BL | GPIO 21 | White |
| 9 | MISO | — | not connected |
| 10 | T_CLK | — | not connected (touch unused) |
| 11 | T_CS | — | not connected |
| 12 | T_DIN | — | not connected |
| 13 | T_DO | — | not connected |
| 14 | T_IRQ | — | not connected |

## Display RST RC Circuit (hardwired auto-reset)

```
       3.3V ───[10K Ω]───┬─── Display RST (pin 4)
                          │
                       [100nF]
                          │
                         GND
```

## Display CS Hardwire (eliminates the GPIO requirement)

```
   Display CS (pin 3) ───── GND
```

The display is the only device on the VSPI bus, so it can stay permanently selected. This frees the one GPIO that the pin budget was short on. See [pin-assignment.md](pin-assignment.md#pin-budget) for the math, and [pin-assignment.md](pin-assignment.md#sleep-time-behavior-of-the-permanent-cs) for why this is safe during deep sleep.

## GPIO21 Backlight Pull-Down (CRITICAL)

```
  GPIO21 ───┬─── Display BL (pin 8)
            │
         [10K Ω]
            │
           GND
```

## MAX98357A Audio Amp

| Amp pin | Connect to | Wire color |
|---------|------------|-----------|
| VIN | 3.3V | Red |
| GND | GND | Black |
| BCLK | GPIO 19 | Purple |
| LRC | GPIO 0 | Gray |
| DIN | GPIO 22 | Brown |
| SD pin | 3.3V (mono mix, 15dB) | short jumper to VIN |
| SPK+ | Speaker + terminal | Red speaker wire |
| SPK- | Speaker − terminal | Black speaker wire |
| GAIN | not connected | (default 9dB) |

## Battery (LiPo PL985525, 8000mAh)

```
  Red   --> B+ pad on LILYGO board   (after desoldering 18650 holder)
  Black --> B- pad on LILYGO board

  Optional bulk cap across B+/B-:
    B+ ───┬─── (to board)
          │
      [1000-2200µF]   (≥6.3V rated electrolytic)
          │
    B- ───┴─── (to board)
```

## Connection Count

| What | Count |
|------|:-----:|
| Signal wires from ESP32 to peripherals | 7 |
| Display CS hardwire to GND | 1 |
| Power/GND wires (3.3V × 4, GND × 5) | 9 |
| Battery wires | 2 |
| Speaker wires | 2 |
| **Total connections** | **21** |
| GPIOs used | 7 of 7 free outputs |
| Resistors | 2× 10K (RST + BL pull-down) |
| Capacitors | 1× 100nF (RST), 1× 1000-2200µF (bulk, optional) |

## Related pages

- [Pin assignment](pin-assignment.md) — why each GPIO got its job
- [Hardware modifications](hardware-modifications.md) — the resistors, caps, battery swap, and CS-to-GND hardwire
- [Bill of materials](bill-of-materials.md) — what each part is
- [Pre-build checklist](pre-build-checklist.md) — verify before soldering
- [Gotchas](gotchas.md) — PSRAM-blocked GPIO 16/17 trap
- LILYGO pinout reference: [`screenshots/lilygo-board/`](screenshots/lilygo-board/)
