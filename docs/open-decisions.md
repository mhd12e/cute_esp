# Open Decisions

> Things still to decide. Update this page as decisions land.

## Decided

- [x] **Language: English only.** No RTL, no multi-script. (2026-05-06)
- [x] **TTS in scope.** One of several content tools, not the only one. (2026-05-06, option B)
- [x] **TTS provider: ElevenLabs.** With voice cloning planned for "Karim" voice (he records ~5 min of sample audio at some point). Mai's voice picked from library. (2026-05-07)
- [x] **Character framing: Mai + Karim themselves, "us in another world."** Pixel-art versions of the two of them. Whether/when his name appears is per-message, not a system rule. (2026-05-06)
- [x] **Architecture: ESP32 is a dumb media player.** Server pre-renders/transcodes every payload to JPEG / MJPEG / MP3. (2026-05-06)
- [x] **Server stack: NestJS + Postgres + MinIO + Redis (only if needed).** Self-hosted on a VPS. Delivered end-to-end. (2026-05-07)
- [x] **Hardware in hand:** LILYGO T-SIM A7670E R2 confirmed (not A7670G), display, MAX98357A, SOTAMIA speaker, 8000mAh LiPo. (2026-05-07)
- [x] **Pin allocation: 7 of 7 free output GPIOs used.** Display DC on GPIO 32 (newly discovered free), display CS hardwired to GND. GPIO 16/17 confirmed PSRAM-blocked. (2026-05-07)

## Design

- [ ] **Pixel art look for Mai and Karim** — visual style of the two characters
- [ ] **Mai's voice (TTS)** — voice ID for when "Mai" speaks
- [ ] **Karim's voice (TTS)** — voice ID for when "Karim" speaks (or just use his recorded voice?)
- [ ] **Animation library** — what scenes are pre-rendered? (morning routine, walk, cafe, beach, sleepy, etc.)
- [ ] **Color palette** — pixel art typically uses limited colors; pick a cohesive set

## Server Choices

- [ ] **Server tech stack** — Node.js? Python (FastAPI)? Go? (User said "easy part")
- [ ] **Hosting** — VPS? Vercel? Railway? AWS?
- [ ] **TTS service** — ElevenLabs (best voices, costs money), Google TTS, OpenAI, Coqui (free, runs on server)
- [ ] **Media storage** — S3? local disk? CDN?
- [ ] **Database** — Postgres? SQLite? for content queue and history

## Enclosure

- [ ] **Enclosure dimensions** — exact size based on assembled hardware
- [ ] **Manufacturing method** — 3D print? Laser-cut acrylic? Off-the-shelf transparent box?
- [ ] **Mounting** — how is the LiPo secured? How is the display mounted to look good?
- [ ] **Charging port access** — USB-C must be reachable but look intentional, not hacked

## UX Details

- [ ] **First boot greeting** — exact animation + speech for the magical first impression
- [ ] **Low battery animations** — what does "sleepy" look like? "Feed me"?
- [ ] **Charging animation** — happy when she plugs in?
- [ ] **Idle behavior** — does the character ever appear briefly to say hi when nothing is sent?
  - Risk: she might think it's "alive" if it talks unprompted
  - Benefit: feels less robotic
- [ ] **Volume control** — can Karim set volume from his app?

## Technical Details

- [ ] **Video codec/format** — MJPEG specifics (resolution, frame rate, file size)
- [ ] **TTS sample rate** — 16kHz is sufficient for voice; 22kHz is nicer
- [ ] **Polling interval flexibility** — always 5 min? Adapt to time of day?
- [ ] **Charge rate verification** — exactly what's the charger IC and current setting?

## Operational

- [ ] **SIM card / data plan** — which operator? (User said "don't worry about it")
- [ ] **Device commissioning** — how is the device token / device ID set up?
- [ ] **Crash recovery** — if firmware crashes, does it restart cleanly?
- [ ] **OTA updates** — can firmware be updated remotely? (Useful for bug fixes after gift)

## Related pages

- [Story and purpose](story-and-purpose.md) — the why that should drive design choices
- [Goals and success criteria](goals-and-success-criteria.md) — what to optimize for
- [Content types](content-types.md) — what playback the design must support
- [Server API](server-api.md) — the contract any server impl must hit
- [Mobile app](mobile-app.md) — UX surface that pairs with these decisions
