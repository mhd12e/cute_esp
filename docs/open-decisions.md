# Open Decisions

> Things still to decide. Update this page as decisions land.

## Design

- [ ] **Pixel art character design** — Karim's choice (he said "don't worry about it for now")
- [ ] **Character name** — does it introduce itself? "Hi, I'm [name]!"
- [ ] **Character voice** — pick a TTS voice ID for consistency
- [ ] **Animation library** — how many emotions? (happy, sad, shy, sleepy, excited, etc.)
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
