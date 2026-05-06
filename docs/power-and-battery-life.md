# Power and Battery Life

> Current draw breakdown, daily energy budget, and how we get a month between charges.

## Sleep Mode Current Breakdown (~3.3mA total)

| Component | State | Current |
|-----------|-------|:-------:|
| ESP32 | Deep sleep (RTC only) | 0.157mA |
| A7670 modem | DTR sleep mode (network registered) | ~3.0mA |
| Display ILI9488 | Sleep mode (cmd 0x10 sent) | ~0.01mA |
| Backlight | OFF (GPIO21 LOW via 10K pull-down) | 0mA |
| MAX98357A | Auto-shutdown (no I2S clock for >1ms) | ~0.003mA |
| SD card | Idle (powered but inactive) | ~0.1mA |
| GPS | OFF | 0mA |
| **Total** | | **~3.3mA** |

## Per-Poll Cost (Empty Poll, No Content)

| Step | Duration | Avg Current | Energy |
|------|:--------:|:-----------:|:------:|
| ESP32 wake from deep sleep | <1 sec | ~80mA | minor |
| Set BOARD_POWERON_PIN HIGH | instant | — | — |
| DTR LOW (modem wakes from nap) | ~2 sec | ~150mA | 0.083mAh |
| HTTP GET request to server | ~3 sec | ~300mA | 0.250mAh |
| DTR HIGH (modem naps) | instant | — | — |
| ESP32 enters deep sleep | instant | — | — |
| **TOTAL per empty poll** | **~5 sec** | — | **~0.28mAh** |

## Active Mode (Content Delivery)

| Activity | Avg Current | Notes |
|----------|:-----------:|-------|
| ESP32 @ 240MHz processing | ~100mA | Full-speed CPU |
| 4G data download | 500mA avg | Up to 2A peak during TX bursts |
| Display + backlight | 80–150mA | Backlight is dominant |
| MAX98357A + speaker | 100–300mA | Depends on volume |
| GPS (when active) | 30–50mA | Built-in GPS via modem |

## Daily Power Budget Formula

```
total_daily_mAh = (sleep_drain) + (empty_polls × 0.28mAh) + (content_minutes × 500/60) + (gps_minutes × 200/60)

where:
  sleep_drain = 3.3mA × 24h = 79.2mAh/day (constant)
  empty_polls = 288/day for 5-min intervals (assuming nothing is sent)
  content_minutes = avg minutes per message × messages per day
  gps_minutes = avg minutes per GPS fix × GPS requests per day
```

## Battery Life Table (8000mAh LiPo, 5-min polling, DTR sleep)

| Daily Pattern | Empty Polls | Active Time | GPS Time | Daily Drain | **Battery Life** |
|--------------|:-----------:|:-----------:|:--------:|:-----------:|:----------------:|
| 2-3 msgs/day, no GPS | 288 × 0.28 = 80 | 6 min × 500/60 = 50 | 0 | 209mAh | **~38 days** |
| 5 msgs/day + 1× GPS | 80 | 15 min = 125 | 2 min = 7 | 291mAh | **~27 days** |
| 7 msgs/day + 1× GPS | 80 | 21 min = 175 | 7 | 341mAh | **~23 days** |
| 10 msgs/day + 3× GPS | 80 | 30 min = 250 | 6 min = 20 | 429mAh | **~18 days** |
| 15+ msgs/day spam | 80 | 45 min = 375 | 0 | 530mAh | **~15 days** |

## Why Polling Without DTR Sleep is Catastrophic

If we did NOT use DTR sleep and instead powered the modem off between polls:

- Cold boot of modem: ~15–30 seconds at ~500mA average = **4–8mAh per poll**
- 288 polls/day × 6mAh = **1728mAh/day from polls alone**
- Battery life: 8000 / (1728 + 79 + 125) = **~4 days**

DTR sleep makes polling feasible. The modem stays registered to the network, just naps its UART. Waking from DTR sleep is ~2 seconds (no re-registration needed).

## Charging

The LILYGO board has a **single-cell lithium charger IC** (likely TP4056 or similar) with USB-C input. It outputs a constant ~4.2V CV charge to the battery.

- Charge current: typically 1A (set by an onboard resistor)
- 8000mAh / 1A = **~8–9 hours full charge** (CC + CV phase)
- She plugs it in overnight, like a phone

**Verify before final assembly:** check the charge current setting on the board to ensure it's safe for the LiPo (typical safe range: 0.3–1C, so up to 8A for an 8000mAh cell — 1A is well below that).

## Related pages

- [System architecture](system-architecture.md) — why DTR sleep is mandatory
- [Sleep/wake cycle](sleep-wake-cycle.md) — code-level deep sleep flow
- [Hardware modifications](hardware-modifications.md) — the LiPo swap and the GPIO21 pull-down
- [Bill of materials](bill-of-materials.md) — the 8000mAh PL985525 spec
- [Behavior states](behavior-states.md) — what triggers active vs sleep mode
