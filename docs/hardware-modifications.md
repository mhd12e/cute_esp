# Hardware Modifications

> Physical changes to make to the LILYGO board before wiring anything.

## Mod 1 — Battery Swap (18650 → 8000mAh LiPo)

1. **Desolder the 18650 battery holder** from the LILYGO board (hot air or iron + braid)
2. Identify the **B+** and **B-** pads on the PCB underneath
3. Solder the LiPo's **red wire to B+**, **black wire to B-**
4. Insulate exposed solder joints with heat shrink or kapton tape
5. The board's USB-C charger IC will charge the LiPo same as it would a 18650 (same voltage)

## Mod 2 — Display RST RC Circuit (saves 1 GPIO)

Build a tiny power-on reset circuit so we don't need a GPIO for RST:

```
    3.3V ────┬───[10KΩ]─── Display RST (pin 4)
             │
          [100nF]
             │
            GND
```

When power applies, the cap holds RST low briefly (~1ms), then the resistor pulls it HIGH. Display resets itself on every power-on. No firmware control needed.

## Mod 3 — Backlight Pull-Down on GPIO21 (CRITICAL)

GPIO21 is **not an RTC GPIO**. `gpio_hold_en()` does NOT persist its state through deep sleep. When ESP32 sleeps, GPIO21 floats. The display backlight has a pull-up internally, so floating = backlight stays on = ~100mA constant drain = **battery dies in days**.

**Fix:** Solder a 10K resistor from GPIO21 to GND.

```
    GPIO21 ────┬──── Display BL (pin 8)
                │
             [10KΩ]
                │
               GND
```

- During sleep: GPIO21 floats, pull-down keeps it LOW = backlight off
- During active: ESP32 drives GPIO21 HIGH, easily overrides 10K (0.33mA "leakage")

## Mod 4 — Bulk Capacitor (Recommended)

The A7670 modem can spike to 2A during 4G transmissions. The LiPo can supply this, but the wires/PCB traces have impedance, causing voltage sag that may reset the ESP32.

**Fix:** Solder a **1000–2200µF electrolytic capacitor** across the battery terminals (or B+/B- pads) to absorb the spikes:

```
    B+ ────┬──── (to board)
           │
       [1000-2200µF]   (≥6.3V rated)
           │
    B- ────┴──── (to board)
```

## Mod 5 — Skip Touch Wiring

Touch is not used. Display pins 9–14 (MISO + 5 touch signals) are left **unconnected**. This frees up pins for other uses but mostly just simplifies the build.

## Mod 6 — Display CS Hardwired to GND

The pin budget is exactly one short of needing CS as a GPIO (see [pin assignment](pin-assignment.md#pin-budget)). Solution: tie display CS (pin 3 of the 14-pin connector) directly to GND with a short jumper.

```
   Display CS (pin 3) ───── GND
```

The display is the only device on the VSPI bus (SD card lives on HSPI), so it can stay permanently selected. The ESP32 never needs to deselect it. During deep sleep the display sits in command-`0x10` Sleep In and ignores all input, so noise on SCK/MOSI is not a concern.

## Related pages

- [Pin assignment](pin-assignment.md) — why GPIO21 specifically
- [Wiring diagram](wiring-diagram.md) — full picture including these mods
- [Gotchas](gotchas.md) — what can go wrong (especially I2C pull-ups on GPIO21/22)
- [Pre-build checklist](pre-build-checklist.md) — what to verify before doing any of this
