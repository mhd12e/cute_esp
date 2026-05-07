# Screenshots

> Labeled product photos and reference images, organized by component.

All screenshots originated as AliExpress listings captured 2026-04-20. They are treated as reference material — pin diagrams, dimensions, variant comparisons, and what's in the box.

## [`lilygo-board/`](lilygo-board/) — LILYGO T-SIM A7670E R2

The main board. **Most important here are the pin diagrams (06–08) and the A7670E vs A7670G comparison (15, 19, 20)** — see [gotchas](../gotchas.md#gotcha-3-a7670e-vs-a7670g-critical-verification).

> **Caveat about the marketing pin diagrams (06–08):** these were drawn by LILYGO for the AliExpress listing and may show GPIO 16 and GPIO 17 as if they were available on the headers. They are not — those pins are inside the ESP32-WROVER module driving the embedded PSRAM, and aren't broken out anywhere. The board schematic at [`../lilygo-reference/schematic/esp32/T-A7670X-V1.4.pdf`](../lilygo-reference/schematic/esp32/T-A7670X-V1.4.pdf) is the source of truth. See [gotchas](../gotchas.md#gotcha-gpio-16-and-17-are-psram-not-free).

| File | Shows |
|------|-------|
| `01-aliexpress-listing.png` | Top of the AliExpress product page (price, version selector) |
| `02-product-specifications.png` | Spec table (chemistry, dimensions, weight) |
| `03-mcu-and-modem-parameters.png` | ESP32 specs (240MHz, 4MB flash, 8MB PSRAM) + A7670 features |
| `04-power-and-protocols.png` | Voltage range, software protocols (TCP/UDP/HTTPS/MQTT) |
| `05-a7670-variants-frequency-bands.png` | A7670G/A7670E/A7670SA frequency band comparison |
| `06-pin-diagram-overview.png` | Pin diagram, top half of the page |
| `07-pin-diagram-with-legend.png` | Pin diagram with color legend (Power/GND/GPIO/I2C/ADC/SPI) |
| `08-pin-diagram-full-with-regions.png` | Full pin diagram including supported regions per variant |
| `09-board-dimensions.png` | Mechanical drawing with board dimensions |
| `10-board-features-labeled.png` | Annotated photo: USB-C, USB micro, SIM, SD, GNSS, status LED, switch, 18650 holder |
| `11-shipping-list.png` | What ships in the box (top of section) |
| `12-shipping-list-detail.png` | Continued shipping list view |
| `13-included-accessories.png` | T-A7670E + LTE antenna + GPS antenna + battery cable |
| `14-gps-antenna-instructions.png` | Where to plug the GPS antenna on A7670E/A7670SA |
| `15-gps-interface-e-vs-g.png` | **Critical**: A7670E (built-in GPS) vs A7670G (separate L76K GPS interface) |
| `16-use-scene-reference.png` | Marketing photo: solar panel + board + dev computer |
| `17-product-display-a7670g.png` | A7670G product shots |
| `18-a7670g-board-views.png` | More A7670G photos |
| `19-a7670e-vs-g-note.png` | Note: "A7670E/A7670SA has built-in GPS, while A7670G has external GPS" |
| `20-a7670e-builtin-gps-detail.png` | Close-up showing the built-in GPS section on A7670E |
| `21-a7670e-top-and-bottom.png` | A7670E top and bottom board views |

## [`display/`](display/) — 4.0″ IPS SPI Display (ILI9488)

| File | Shows |
|------|-------|
| `01-aliexpress-listing.png` | Product page top: 4.0″ IPS, 320×480, ILI9488 + FT6236 capacitive touch |
| `02-top-and-bottom-views.png` | Top + bottom of the display PCB (yellow variant) |
| `03-pcb-schematic-note.png` | Schematic snippet from product description |
| `04-pcb-dimensions.png` | Mechanical drawing of the display PCB |
| `05-product-photos.png` | Marketing photos of the display in use |

## [`speaker/`](speaker/) — SOTAMIA 4Ω 3W Speaker

| File | Shows |
|------|-------|
| `01-sotamia-listing.png` | Product page top (2pcs, 4Ω, 3W, 20×30mm) |
| `02-sotamia-specs-part1.png` | Specifications table (impedance, power, frequency range) |
| `03-sotamia-specs-part2.png` | Continued specs (material, channel count, manuals) |

## [`amplifier/`](amplifier/) — MAX98357A I2S Class D Amp

| File | Shows |
|------|-------|
| `01-max98357a-listing.png` | Product page (5pcs pack, 3W, AED 3.75) |

## Related pages

- [Bill of materials](../bill-of-materials.md) — what each component is for
- [Pin assignment](../pin-assignment.md) — uses the pin diagrams (06–08)
- [Hardware modifications](../hardware-modifications.md) — battery swap, RC circuit, BL pull-down
- [Gotchas](../gotchas.md) — A7670E vs A7670G verification (uses 15, 19, 20)
- [Pre-build checklist](../pre-build-checklist.md) — visual verification steps
