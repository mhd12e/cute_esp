# Bill of Materials

> What's been bought, what's still needed, and what each piece is for.

## Already Purchased (Total: AED 255.83)

| # | Component | Specs | Price | Status |
|---|-----------|-------|:-----:|:------:|
| 1 | LILYGO T-SIM A7670E R2 | ESP32 + 4G LTE Cat1 + Built-in GPS + SD slot + 18650 holder | AED 141.70 | **In hand** |
| 2 | SOTAMIA Mini Speakers (2pcs) | 4Ω, 3W, full-range, 20×30mm | AED 9.46 | **In hand** |
| 3 | MAX98357A I2S Amplifier (5pcs) | Class D, 3W output, mono | AED 3.75 | **In hand** |
| 4 | 4.0" IPS SPI Display | ILI9488 + FT6236 capacitive touch, 480×320, 14-pin connector | AED 55.92 | **In hand** |
| 5 | PL985525 LiPo Battery | 3.7V, 8000mAh, lithium polymer pouch | AED 45.00 | ⏳ **Ordered, not yet arrived** |

> **Things still to procure** (battery, micro SD, SIM, tools, perfboard, etc.) live on the [to-buy page](to-buy.md).

Reference photos: see [`screenshots/lilygo-board/`](screenshots/lilygo-board/), [`screenshots/display/`](screenshots/display/), [`screenshots/speaker/`](screenshots/speaker/), [`screenshots/amplifier/`](screenshots/amplifier/).

## Additional Components Needed (from parts bin)

| Part | Purpose | Quantity |
|------|---------|:--------:|
| 10K Ω resistor (1/4W) | Display RST pull-up + GPIO21 backlight pull-down | 2 |
| 100nF ceramic capacitor | Display RST RC reset circuit | 1 |
| 1000–2200 µF electrolytic cap (≥6.3V) | Bulk decoupling for modem current spikes | 1 |
| Jumper wires / hookup wire | Connections | ~25 |
| Solder + flux | Assembly | — |

## LILYGO Board Specs (Detailed)

- **MCU:** ESP32 (Tensilica LX6 dual-core, 240MHz)
- **Flash:** 4MB
- **PSRAM:** 8MB
- **Modem:** A7670E (Asia/Europe LTE Cat1: B1/B3/B5/B7/B8/B20)
- **GPS:** Built into A7670E modem (BeiDou/GPS/GLONASS) — accessed via AT commands, no extra pins
- **Connectivity:** Wi-Fi 802.11 b/g/n, BT v4.2 (BR/EDR + BLE)
- **Storage:** Built-in SD card slot (HSPI: GPIO 14, 2, 15, 13)
- **Power:** 18650 holder + USB-C charging (single-cell lithium charger IC)
- **SIM:** Nano SIM, 1.8V/3.0V
- **USB:** USB-C for power/programming + Micro USB for A7670 firmware updates (separate)
- **Voltage range:** 3.4–4.2V (battery direct)

## Display Specs (4.0" IPS)

- **Controller:** ILI9488
- **Touch:** FT6236 capacitive (NOT USED in this project)
- **Resolution:** 480×320
- **Interface:** SPI (4-wire, write-only mode)
- **Connector:** 14-pin FPC ribbon cable
- **Color:** "Yellow" PCB variant
- **Power:** 3.3V logic, 3.3V backlight

## Speaker & Amp

- **MAX98357A:** I2S input, Class D output, 3W into 4Ω, integrated DAC
- **Speaker:** SOTAMIA 4Ω 3W full-range, ~20×30mm flat speaker

## Battery (LiPo Replacement)

- **PL985525** — standard naming convention: 9mm × 55mm × 25mm pouch cell
- **3.7V nominal, 4.2V max charge, 3.0V cutoff**
- **8000mAh capacity** (vs ~3500mAh for typical 18650)
- **Single cell** — compatible with onboard charger as-is

## Related pages

- [Hardware modifications](hardware-modifications.md) — what to physically change on the board
- [Pin assignment](pin-assignment.md) — which GPIOs go where
- [Wiring diagram](wiring-diagram.md) — every connection on one page
- [Power and battery life](power-and-battery-life.md) — why we picked 8000mAh
