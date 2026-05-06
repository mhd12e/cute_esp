# Wiring Diagram

> Every connection on a single page.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                  CUTE ESP — COMPLETE WIRING                            ║
╚══════════════════════════════════════════════════════════════════════════╝

LILYGO T-A7670E R2 (top-level)
═══════════════════════════════
Internal pins (DO NOT WIRE):
  GPIO 4, 5, 12, 25, 26, 27, 33, 35, 36 — modem control / battery sense
  GPIO 2, 13, 14, 15 — SD card (HSPI bus)

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
```

## Display 14-Pin Connector (only 7 pins connected)

| Pin | Signal | Connect to | Wire color |
|:---:|--------|------------|-----------|
| 1 | VCC | 3.3V | Red |
| 2 | GND | GND | Black |
| 3 | CS | GPIO17 | Orange |
| 4 | RST | (RC reset circuit) | hardwired |
| 5 | DC | GPIO16 | Yellow |
| 6 | MOSI | GPIO23 | Green |
| 7 | SCK | GPIO18 | Blue |
| 8 | BL | GPIO21 | White |
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
| BCLK | GPIO19 | Purple |
| LRC | GPIO0 | Gray |
| DIN | GPIO22 | Brown |
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
| Signal wires | 17 |
| Battery wires | 2 |
| Speaker wires | 2 |
| **Total connections** | **21** |
| GPIOs used | 8 of 8 available |
| Resistors | 2× 10K (RST + BL pull-down) |
| Capacitors | 1× 100nF (RST), 1× 1000-2200µF (bulk, optional) |

## Related pages

- [Pin assignment](pin-assignment.md) — why each GPIO got its job
- [Hardware modifications](hardware-modifications.md) — the resistors, caps, and battery swap
- [Bill of materials](bill-of-materials.md) — what each part is
- [Pre-build checklist](pre-build-checklist.md) — verify before soldering
- LILYGO pinout reference: [`screenshots/lilygo-board/`](screenshots/lilygo-board/)
