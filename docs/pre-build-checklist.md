# Pre-Build Checklist

> Verify these when the LILYGO board arrives. Before any soldering.

## Verification Steps

- [ ] **Board variant:** Confirm A7670E (built-in GPS) NOT A7670G (external L76K)
  - Look for any extra chip near the modem labeled "L76K" — if present, wrong board
  - See: [`screenshots/lilygo-board/15-gps-interface-e-vs-g.png`](screenshots/lilygo-board/15-gps-interface-e-vs-g.png)
- [ ] **Board version:** Should be R2 (latest revision). Check silkscreen.
- [ ] **GPIO21/22 voltage:** With nothing connected, measure GPIO21 and GPIO22 to GND.
  - If ~0V or floating: no I2C pull-ups, our pull-down works fine
  - If ~3.3V: there are pull-ups, may need to cut traces or use transistor for BL
- [ ] **Display pinout:** Read the silkscreen on the 14-pin connector
  - Confirm pins 1–8 are: VCC, GND, CS, RST, DC, MOSI, SCK, BL
  - Confirm pins 10–14 are touch (T_*)
  - If different, adjust wiring accordingly
- [ ] **SD card slot:** Test with a formatted micro SD, run the SDCard.ino example
- [ ] **SIM card:** Insert SIM, verify network registration, test SMS receive (fallback option)
- [ ] **Battery charger current:** Test with multimeter while charging — current should be ~1A. If higher (rare), reduce by changing the program resistor on the charger IC
- [ ] **USB-C boot:** Verify board boots from USB-C alone (no battery)
- [ ] **Battery boot:** Verify board boots from battery alone (no USB)

## Once Verified, Proceed With Hardware Modifications

1. Desolder 18650 holder
2. Solder LiPo wires to B+/B- pads
3. Solder bulk capacitor across B+/B-
4. Build display RST RC circuit (10K + 100nF)
5. Solder 10K pull-down resistor between GPIO21 and GND
6. Wire display (7 connections)
7. Wire MAX98357A (6 connections)
8. Wire speaker (2 connections)

Then flash firmware and test step-by-step.

## Related pages

- [Hardware modifications](hardware-modifications.md) — the mods to perform after this checklist
- [Wiring diagram](wiring-diagram.md) — what to wire after mods
- [Gotchas](gotchas.md) — what each verification step is checking for
- [LILYGO examples reference](lilygo-examples.md) — `SDCard.ino` and other test sketches
