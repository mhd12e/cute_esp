# Mobile App (Karim's Phone)

> The React Native Expo app that drives the device.

## Screens

### Home / Send

- Text input with character emotion picker (happy, sad, shy, excited, sleepy)
- "Send text" button (server generates TTS automatically)
- Photo picker (gallery or camera)
- Audio recorder (record voice memo, server runs through cute character voice filter or just plays as-is)
- Video picker (gallery or camera)
- Music picker (from device's library)

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
