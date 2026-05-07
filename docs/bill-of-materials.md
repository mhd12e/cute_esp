# Bill of Materials

> Master inventory for the build. Tick boxes as items land. Procurement reasoning lives in [to-buy](to-buy.md).

## Inventory Checklist

### Major components

- [x] **LILYGO T-SIM A7670E R2** — AED 141.70 — *in hand, A7670E variant verified*
- [x] **SOTAMIA Mini Speaker** (2-pack, 4Ω 3W, 20×30mm) — AED 9.46 — *in hand (1 used + 1 spare)*
- [x] **MAX98357A I2S Amplifier** (5-pack, 3W Class D) — AED 3.75 — *in hand (1 used + 4 spares)*
- [x] **4.0″ IPS SPI Display** (ILI9488 + FT6236, 480×320, 14-pin) — AED 55.92 — *in hand*
- [ ] **PL985525 LiPo Battery** (3.7V, 8000mAh) — AED 45.00 — *ordered, not yet arrived*

### Storage & connectivity

- [ ] **micro SD card** (16GB Class 10, e.g. SanDisk / Samsung Evo)
- [ ] **Nano SIM card with 4G data plan** (must support LTE Cat1 on B1/B3/B5/B7/B8/B20)

### Passive components

- [ ] **10 KΩ resistor, 1/4W** × 2 — display RST pull-up + GPIO 21 backlight pull-down
- [ ] **100 nF ceramic capacitor** × 1 — display RST RC reset
- [ ] **1000–2200 µF electrolytic capacitor**, ≥6.3V — bulk decoupling for modem 2A spikes
- [ ] **BC547 or 2N3904 NPN transistor** × 5 — *insurance only, in case the I2C pull-up trap fires* (see [gotcha 4](gotchas.md#gotcha-4-gpio2122-may-have-i2c-pull-ups))

### Wiring & insulation

- [ ] **22 AWG hookup wire** (multiple colors) — battery + power runs
- [ ] **Jumper wire kit** (M-M, M-F, F-F, ~40 of each)
- [ ] **Heat shrink tubing**, assorted diameters — mandatory for LiPo solder joints
- [ ] **Kapton tape** — battery insulation, assembly fixturing

### Build substrate

- [ ] **Perfboard** (~5×7 cm, 2.54 mm pitch) × 2–3 — final build
- [ ] **Breadboard** (~830 tie-points) — optional, useful for bench-testing display + amp before committing

### Tools

- [ ] **Multimeter** — required by [pre-build checklist](pre-build-checklist.md)
- [ ] **Soldering iron** (60W temp-controlled) + 0.6 mm solder + flux
- [ ] **Solder wick / desoldering braid** *or* hot-air station — needed to remove the 18650 holder
- [ ] **USB-C cable** (data-capable, not charging-only) — for flashing firmware

### Skip unless something specific comes up

- ~~Diodes~~ — board has its own charging/reverse-polarity diodes. No inductive loads in our wiring. Not needed.
- ~~Logic analyzer / oscilloscope~~ — only useful if a specific debug session calls for it.
- ~~Bench power supply~~ — overkill; the LiPo + USB charger is the supply.

---

## Major Component Specs (detailed)

### 1. LILYGO T-SIM A7670E R2

- **MCU:** ESP32 (Tensilica LX6 dual-core, 240 MHz)
- **Flash:** 4 MB
- **PSRAM:** 8 MB (uses GPIO 16 + 17 internally — see [pin assignment](pin-assignment.md))
- **Modem:** A7670E (Asia/Europe LTE Cat1: B1/B3/B5/B7/B8/B20)
- **GPS:** Built into A7670E modem (BeiDou/GPS/GLONASS) — accessed via AT commands, no extra pins
- **Connectivity:** Wi-Fi 802.11 b/g/n, BT v4.2 (BR/EDR + BLE)
- **Storage:** Built-in micro-SD slot (HSPI: GPIO 14, 2, 15, 13)
- **Power:** 18650 holder + USB-C charging (single-cell lithium charger IC)
- **SIM:** Nano SIM, 1.8V/3.0V
- **USB:** USB-C for power/programming + Micro USB for A7670 firmware updates (separate)
- **Voltage range:** 3.4–4.2V (battery direct)

### 2. 4.0″ IPS SPI Display

- **Controller:** ILI9488
- **Touch:** FT6236 capacitive (NOT USED in this project)
- **Resolution:** 480 × 320
- **Interface:** SPI (4-wire, write-only mode)
- **Connector:** 14-pin FPC ribbon cable
- **Color:** "Yellow" PCB variant
- **Power:** 3.3V logic, 3.3V backlight

### 3. MAX98357A I2S Amplifier

- I2S input → Class D output, 3 W into 4 Ω, integrated DAC
- Pins: VIN / GND / BCLK / LRC / DIN / SD / GAIN / Speaker±
- We tie SD pin to 3.3V → mono L+R mix mode, 15 dB gain

### 4. SOTAMIA Mini Speaker

- 4 Ω, 3 W, full-range, ~20 × 30 mm flat speaker
- Two terminals, simple solder

### 5. PL985525 LiPo Battery

- **PL985525** name: 9 mm × 55 mm × 25 mm pouch cell
- **3.7 V nominal**, 4.2 V max charge, 3.0 V cutoff
- **8000 mAh** capacity (vs ~3500 mAh for typical 18650)
- Single cell — compatible with onboard charger as-is

## Reference photos

- Board: [`screenshots/lilygo-board/`](screenshots/lilygo-board/)
- Display: [`screenshots/display/`](screenshots/display/)
- Speaker: [`screenshots/speaker/`](screenshots/speaker/)
- Amplifier: [`screenshots/amplifier/`](screenshots/amplifier/)

## Related pages

- [To-buy](to-buy.md) — procurement reasoning and price estimates
- [Hardware modifications](hardware-modifications.md) — what to physically change on the board
- [Pin assignment](pin-assignment.md) — which GPIOs go where
- [Wiring diagram](wiring-diagram.md) — every connection on one page
- [Power and battery life](power-and-battery-life.md) — why we picked 8000 mAh
