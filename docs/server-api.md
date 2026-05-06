# Server API

> The REST contract between the device, the server, and Karim's app.

## Endpoints

| Method | Path | Purpose |
|:------:|------|---------|
| POST | `/api/content` | Karim's app pushes new content |
| GET | `/api/device/{id}/pending` | Device polls for queued content + flags |
| GET | `/api/device/{id}/content/{contentId}` | Device downloads specific media file |
| POST | `/api/device/{id}/status` | Device reports battery level |
| POST | `/api/device/{id}/location` | Device reports GPS coordinates |
| POST | `/api/content/{contentId}/ack` | Device acknowledges content played |

## Example: Device Polls

```http
GET /api/device/karim-device-1/pending HTTP/1.1
Authorization: Bearer <device_token>

200 OK
{
  "pending": true,
  "gps_requested": false,
  "items": [
    {
      "id": "msg_001",
      "type": "text_with_animation",
      "animation": "happy_wave",
      "text": "Good morning! Hope you slept well.",
      "audio_url": "/api/device/karim-device-1/content/audio_001.mp3"
    },
    {
      "id": "msg_002",
      "type": "photo",
      "photo_url": "/api/device/karim-device-1/content/photo_002.jpg",
      "caption": "I saw this and thought of you",
      "caption_audio_url": "/api/device/karim-device-1/content/audio_002.mp3"
    }
  ]
}
```

## Example: Device Reports Status

```http
POST /api/device/karim-device-1/status HTTP/1.1
Content-Type: application/json
Authorization: Bearer <device_token>

{
  "battery_voltage": 3.94,
  "battery_percent": 74,
  "uptime_seconds": 1234,
  "last_wake_reason": "timer",
  "free_sd_mb": 7800
}
```

## Example: Device Reports Location

```http
POST /api/device/karim-device-1/location HTTP/1.1
Content-Type: application/json

{
  "lat": 25.276987,
  "lon": 55.296249,
  "accuracy_m": 12,
  "speed_kmh": 0,
  "altitude_m": 8,
  "fixed_at": "2026-04-20T14:32:11Z"
}
```

## Server Responsibilities

1. **Authentication** — device has a token, Karim's app has a separate token
2. **Content queue** — accumulate messages until device picks them up
3. **TTS generation** — on text submission, generate MP3 via TTS API, store URL
4. **Media storage** — accept Karim's photos/videos, transcode video to MJPEG, serve files
5. **Acknowledgment tracking** — mark items as delivered when device confirms
6. **GPS logging** — store location history for Karim to see in app
7. **Battery history** — track battery level over time (Karim can see if she's charging)

## Related pages

- [System architecture](system-architecture.md) — where the server fits overall
- [Content types](content-types.md) — what `type` values mean
- [Mobile app](mobile-app.md) — what Karim sends from the other end
- [Sleep/wake cycle](sleep-wake-cycle.md) — when the device hits these endpoints
- [Open decisions](open-decisions.md) — server stack, hosting, TTS provider TBD
