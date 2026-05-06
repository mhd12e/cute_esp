# Behavior States

> What the device is doing at any given moment, and how it transitions between modes.

## State Diagram

```
                    ┌─────────────────┐
                    │  POWER OFF      │
                    │ (battery dead   │
                    │  or USB unplug) │
                    └────────┬────────┘
                             │  power applied
                             ▼
                    ┌─────────────────┐
                    │  COLD BOOT       │
                    │  (first time or  │
                    │   crash recovery)│
                    └────────┬─────────┘
                             │
                             │  greeting + initial poll
                             ▼
              ┌──── ┌─────────────────┐ ◄────────────┐
              │     │  DEEP SLEEP +    │              │
              │     │  DTR NAP (~3mA)  │              │
              │     └────────┬─────────┘              │
              │              │                        │
              │              │ timer 5 min            │
              │              ▼                        │
              │     ┌─────────────────┐               │
              │     │  POLL SERVER    │               │
              │     │  (~5 sec)       │               │
              │     └────────┬────────┘               │
              │              │                        │
              │              │                        │
              │      ┌───────┴───────┐                │
              │      │               │                │
              │   no content     new content          │
              │      │               │                │
              │      └───────────────┼────────────────┤
              │                      │                │
              │                      ▼                │
              │             ┌─────────────────┐       │
              │             │  CONTENT DELIV. │       │
              │             │  - download SD  │       │
              │             │  - display on   │       │
              │             │  - play audio   │       │
              │             │  - report bat   │       │
              │             └────────┬────────┘       │
              │                      │                │
              │           gps requested?              │
              │                      │                │
              │              ┌───────┴───────┐        │
              │              │               │        │
              │             yes              no       │
              │              │               │        │
              │              ▼               │        │
              │     ┌─────────────────┐      │        │
              │     │  GPS FIX +      │      │        │
              │     │  REPORT (~2 min)│      │        │
              │     └────────┬────────┘      │        │
              │              │               │        │
              │              └───────────────┼────────┤
              │                              │        │
              │      battery < 3.3V? ◄──────┘        │
              │              │                        │
              │      ┌───────┴───────┐                │
              │      │               │                │
              │     yes              no               │
              │      │               │                │
              │      ▼               └────────────────┘
              │  ┌─────────────────┐
              └──┤ EMERGENCY SLEEP │
                 │  (1 hour, then  │
                 │  re-check)      │
                 └─────────────────┘
```

## State Descriptions

**DEEP SLEEP + DTR NAP** — The default. Device drawing 3.3mA. Display off, backlight off, audio off. Modem registered to network but UART asleep. Lasts up to 5 minutes between polls.

**POLL** — Brief wake, ~5 seconds. ESP32 wakes, modem wakes (~2s), HTTP GET to server, back to sleep. If the server has nothing, device goes straight back to sleep.

**CONTENT DELIVERY** — Active. Display on, audio playing. Duration depends on content (typical 1–5 minutes per message). Could be 10+ minutes if Karim queues many messages.

**GPS FIX** — Triggered by server flag during a poll. GPS module activates, gets a fix (30s–2min depending on conditions), reports lat/lon to server, GPS off.

**LOW BATTERY** — When voltage < ~3.7V (~25%), the character starts showing "sleepy" animations. When < 3.5V, it says "feed me" / shows charge prompt. When < 3.3V, **emergency shutdown**: device sleeps for 1 hour at a time, just hoping she plugs it in. This protects the LiPo from over-discharge damage.

**CHARGING** — USB-C plugged in. The board's charger IC takes over. Modem cannot enter DTR sleep on USB power (VBUS keeps it awake) so power consumption is higher, but it doesn't matter because we're plugged in. Character could show a happy "thank you for charging me" animation when she plugs in.

## Related pages

- [Sleep/wake cycle](sleep-wake-cycle.md) — code-level transitions
- [Power and battery life](power-and-battery-life.md) — current draw per state
- [Content types](content-types.md) — what plays in CONTENT DELIVERY
- [Server API](server-api.md) — the protocol that drives state changes
- [Story and purpose](story-and-purpose.md) — the why behind low-battery behaviors
