# System Architecture

> Three pieces: Karim's phone, a server, the device. Connected by HTTPS over 4G.

## High-Level Flow

```
┌────────────────────────────────────────────────────────────────────────┐
│                                                                        │
│   Karim's phone (React Native Expo app)                                │
│                  │                                                     │
│                  │ HTTPS POST                                          │
│                  ▼                                                     │
│   Remote Server (REST API)                                             │
│       - Stores content queue                                           │
│       - Generates TTS audio (MP3)                                      │
│       - Pre-renders scenes/animations as MJPEG                         │
│       - Resizes photos to 480x320 JPEG                                 │
│       - Transcodes videos to MJPEG + separate MP3                      │
│       - Tracks GPS history & battery level                             │
│                  ▲                                                     │
│                  │ HTTPS GET / POST                                    │
│                  │ (every 5 min over 4G LTE)                           │
│                  │                                                     │
│   ESP32 Device (LILYGO T-SIM A7670E R2)                                │
│       - Polls server every 5 min while in deep sleep cycle             │
│       - Downloads media files when present                             │
│       - Plays JPEG / MJPEG / MP3 directly (no rendering)               │
│       - Reports battery + GPS when asked                               │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

## Division of Labor (Final)

The ESP32 is intentionally a **dumb media player**. All rendering — text, fonts, scenes, video, photo resizing — happens on the server. The device only knows how to display a JPEG, play an MJPEG, and play an MP3 through I2S. See [content types](content-types.md#architectural-rule-esp32-is-a-dumb-media-player) for why.

## Why Timer Polling Over SMS Wake-up

We considered SMS wake-up (modem RI pin triggers ESP32 from deep sleep on incoming SMS). We **rejected it**:

| Concern | SMS Wake-up | Timer Polling |
|---------|:-----------:|:-------------:|
| Latency | Instant (seconds) | 0–5 min |
| Server complexity | REST + SMS gateway | REST only |
| Per-message cost | SMS gateway fees | Free |
| Operator dependency | Needs "SMS over SGs" | Any 4G works |
| Battery life @ 5 msgs/day | ~40 days | ~27 days |
| Code complexity | High (RI pin + AT cmds) | Low (HTTP polling) |

**Tradeoff:** ~5 min latency for ~13 days less battery, but **vastly simpler** server and no gateway costs. For a romantic message, 5 minutes is invisible. We chose simplicity.

## Why DTR Sleep is Mandatory

Without DTR sleep (modem fully awake between polls): ~20mA idle = **6–12 days battery**.
With DTR sleep (modem napping at ~3mA): **~27–38 days battery**.

DTR sleep is non-negotiable. See [sleep/wake cycle](sleep-wake-cycle.md) for details.

## Related pages

- [Sleep/wake cycle](sleep-wake-cycle.md) — exact code-level flow for poll/sleep
- [Power and battery life](power-and-battery-life.md) — current draw breakdown
- [Server API](server-api.md) — the REST contract between device and server
- [Mobile app](mobile-app.md) — what Karim sees on his phone
- [Behavior states](behavior-states.md) — what the device does between polls
