# To Buy

> Tracking list of every item we still need before we can build. Strike through as ordered. Honest about what's strictly required vs. nice-to-have.

## Critical (build cannot proceed without these)

- [ ] **PL985525 8000mAh LiPo battery** — already chosen, just hasn't arrived. The build is on hold on the power side until this lands. See [bill of materials](bill-of-materials.md#5-pl985525-lipo-battery).
- [ ] **micro SD card (8–32GB, Class 10)** — we never explicitly listed this and it's mandatory. Media (JPEG/MJPEG/MP3 from server) is stored on it. Card slot is already on the LILYGO board. SanDisk or Samsung Evo are reliable. 16GB is plenty.
- [ ] **Nano SIM card with 4G data plan** — operator-specific. Needs to support LTE Cat1 in the bands the A7670E supports (B1/B3/B5/B7/B8/B20). The SIM also needs to allow international roaming if the recipient travels. You said "don't worry about it" earlier, but flagging here so we don't forget.

## Tools (need before we can assemble & test)

- [ ] **Multimeter** — essential. Two specific checks the [pre-build checklist](pre-build-checklist.md) requires:
  1. GPIO 21/22 voltage with nothing connected (detects I2C pull-up trap, [gotcha 4](gotchas.md#gotcha-4-gpio2122-may-have-i2c-pull-ups))
  2. Battery charge current ~1A while charging
  - A cheap ~AED 30 multimeter (Uni-T UT33+ class) is enough. Don't need a Fluke for this.
- [ ] **Soldering iron + solder + flux** — assumed you have. If not, a 60W temperature-controlled iron (~AED 100) and 0.6mm leaded solder. Lead-free is harder for the 18650 holder desoldering job.
- [ ] **Solder wick / desoldering braid OR hot-air station** — needed to remove the 18650 holder from the LILYGO board ([Mod 1](hardware-modifications.md#mod-1--battery-swap-18650--8000mah-lipo)). Wick is fine if you don't have hot air.
- [ ] **USB-C cable (data, not just charging)** — for flashing firmware to the ESP32. Many "charging" cables are power-only; verify the cable carries data.

## Prototyping & wiring

- [ ] **Jumper wires (M-M, M-F, F-F set)** — at least 40 of each. The wiring diagram has 21 connections; you want spares. AED ~15 for a 120-piece kit.
- [ ] **Perfboard (~5×7cm, 2.54mm pitch)** — for the *final* build. The transparent-case aesthetic implies perfboard or PCB; perfboard is the cheap path. Buy 2–3 boards in case one is botched. AED ~15.
- [ ] **Breadboard (full-size, ~830 tie-points)** — useful for bench-testing the display + amp + speaker before committing to perfboard. The LILYGO board itself doesn't sit nicely in a breadboard (battery holder fouls it), but the display and amp do. Optional if you're confident going straight to perfboard. AED ~10.
- [ ] **Hookup wire (22 AWG, multiple colors)** — for battery wires (need the gauge for ~2A modem peaks), and for permanent perfboard runs. A small spool set is enough. AED ~20.
- [ ] **Heat shrink tubing (assorted)** — insulate the LiPo solder joints (mandatory — bare LiPo terminals are a fire risk). AED ~10.
- [ ] **Kapton tape** — extra insulation under the LiPo cell, around solder joints, and to hold the cell to the perfboard during assembly. AED ~15.

## Components from parts bin (or buy if you don't have)

| Item | Quantity | What for |
|------|:--------:|----------|
| 10 KΩ resistor (1/4W) | 2 (+ spares) | Display RST RC + GPIO21 BL pull-down. See [hardware-modifications](hardware-modifications.md). |
| 100 nF ceramic capacitor | 1 (+ spares) | Display RST RC circuit |
| 1000–2200 µF electrolytic, ≥6.3V | 1 | Bulk decoupling for modem 2A current spikes |

If you don't have a parts bin, buy a **basic resistor assortment kit** (E12 series, 1/4W) for ~AED 30 — it'll cover these and any future "I need a 4.7K right now" moments.

## Insurance — buy a few in case something goes wrong

- [ ] **Small-signal NPN transistors (BC547 or 2N3904), 5 pcs** — *only* needed if the board turns out to have I2C pull-ups on GPIO 21/22 fighting our 10K backlight pull-down. The fallback in [gotcha 4](gotchas.md#gotcha-4-gpio2122-may-have-i2c-pull-ups) is to drive BL through a transistor. AED ~5. Cheap insurance.
- [ ] **Spare 10 KΩ resistors and 100 nF caps** — already covered in the assortment kit above; just noting.

## Skip unless something specific comes up

- ~~Diodes~~ — **not needed for our wiring**. The LILYGO board already has reverse-polarity and charging-path diodes. We're not switching inductive loads. If your parts bin has 1N4148 or 1N4007 lying around, fine, but don't buy them just for this.
- ~~Logic analyzer~~ — useful for debugging SPI/I2S if something glitches, but ~AED 25 cheap clones from AliExpress are good enough *if* a problem appears. Don't buy upfront.
- ~~Bench power supply~~ — overkill for a single-cell project. The LiPo + USB charger is the power supply.
- ~~Oscilloscope~~ — definitely overkill.
- ~~ESD wrist strap~~ — boring but real. ESP32 modules can survive without it; just don't shuffle on carpet before touching the board.

## Enclosure — deferred

The transparent case, mounting, and charging-port access are all in [open decisions](open-decisions.md#enclosure). Don't buy enclosure materials until the assembled hardware's actual dimensions are measured. This goes in its own to-buy round once we're past the perfboard stage.

## Estimated total to spend (best case)

If your parts bin already has resistors, caps, soldering iron, hookup wire, and heat shrink:

| Item | AED |
|------|:---:|
| LiPo battery | already paid (~45) |
| micro SD card | ~25 |
| SIM card + small data plan | varies; ~30/month |
| Multimeter (if needed) | ~30 |
| Jumper wire kit | ~15 |
| Perfboard ×3 | ~15 |
| Breadboard | ~10 |
| **Estimated extra spend** | **~AED 95–125 + SIM monthly** |

If parts bin is empty and you need everything from scratch, add ~AED 100 for soldering iron + solder + assortment kit + heat shrink + transistors as insurance.

## Related pages

- [Bill of materials](bill-of-materials.md) — what's already purchased
- [Hardware modifications](hardware-modifications.md) — what each component is for
- [Pre-build checklist](pre-build-checklist.md) — what to verify when items arrive
- [Open decisions](open-decisions.md) — enclosure, SIM operator still TBD
