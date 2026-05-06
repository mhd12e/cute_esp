# Mobile App (Karim's Phone)

> The React Native Expo app that drives the device.

## Screens

### Home / Send

- Text input with character emotion picker (happy, sad, shy, excited, sleepy) — text goes to server, server runs TTS to MP3 and (optionally) renders a scene MJPEG
- "Send text" button
- Photo picker (gallery or camera) — server resizes to 480×320 JPEG
- Audio recorder (record voice memo, sent as MP3)
- Video picker (gallery or camera) — server transcodes to MJPEG + separate MP3
- Music picker (from device's library)

The device never receives raw text. Everything Karim sends is server-prepared into JPEG / MJPEG / MP3 before reaching the device.

### Status

- Device battery level (gauge)
- Last seen (when was last poll)
- Battery history (chart)

### Location

- Map with current location pin
- "Request location" button (sets `gps_requested` flag, returns within ~5–7 min)
- Location history with timestamps
- Patterns: "She visited [place] for [duration]" if Karim wants to be creepy/poetic

### Queue

- See pending messages waiting for next poll
- Cancel/edit pending messages

### Settings

- Polling interval (default 5 min, can change)
- Character voice selection (affects TTS voice)
- Connection status (is device reachable?)

## Tech Stack

- **React Native Expo** (already chosen)
- Backend communication via HTTPS REST
- Image picker, audio recorder from Expo libraries
- Map view (e.g., react-native-maps) for location screen

## Related pages

- [Server API](server-api.md) — the contract this app talks to
- [Content types](content-types.md) — what each "send" action produces
- [System architecture](system-architecture.md) — where the app fits overall
- [Story and purpose](story-and-purpose.md) — why GPS tracking is part of the app
- [Open decisions](open-decisions.md) — character voice still TBD
