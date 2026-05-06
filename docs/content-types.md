# Content Types

> Every kind of message Karim can send, how it's stored, and how it plays.

## Content Types Karim Can Send

| Type | Format | Storage | Playback |
|------|--------|---------|----------|
| **Text message** | UTF-8 string | RAM | Display + TTS |
| **Animation + text** | Pixel art frame sequence + caption | SD card | Display loop + TTS |
| **Audio** | MP3 or WAV | SD card | I2S to MAX98357A |
| **Photo** | JPEG | SD card | Display |
| **Video** | MJPEG (animated JPEG sequence) | SD card | Display + audio sync |
| **Music** | MP3 | SD card | I2S to MAX98357A |

## Why MJPEG for Video

The ESP32 cannot decode H.264/H.265 video in real time. **MJPEG** (Motion JPEG — a sequence of JPEG frames) is decodable on ESP32 at modest resolutions. Server transcodes incoming videos to MJPEG before delivery.

Typical specs:
- 320×240 @ 15fps MJPEG
- ~50KB per frame, ~750KB per second
- Audio track stripped + sent separately as MP3 (synced via timestamp)

## TTS Pipeline (Server-Side)

```
Karim types text in app
         │
         ▼
Server receives text
         │
         ▼
Server runs TTS engine (e.g., ElevenLabs, Google TTS, Coqui)
         │
         ▼
Audio file saved (MP3, ~64kbps for size, mono)
         │
         ▼
Audio URL added to content queue
         │
         ▼
Device polls, downloads MP3 to SD card
         │
         ▼
Device plays via I2S MP3 decoder library + MAX98357A
```

The character has a "voice" — a specific TTS voice ID is chosen and reused for all messages so the character has consistent personality.

## Content Queue Behavior

If Karim sends 5 messages while the device is asleep, all 5 sit in the server queue. At the next poll (max 5 min later), the device downloads all 5 in one wake cycle and plays them sequentially. This is more efficient than waking up 5 separate times.

After playback, the device acknowledges to the server (the queue is cleared).

## Related pages

- [Server API](server-api.md) — how items reach the device
- [Behavior states](behavior-states.md) — what state the device enters during playback
- [Mobile app](mobile-app.md) — where Karim picks the content type
- [Open decisions](open-decisions.md) — TTS provider, video resolution, voice choice still TBD
- [Firmware patterns](firmware-patterns.md) — I2S setup, SD card I/O
