# Story and Purpose

> Why this device exists. The romance, the constraints, the win condition.

## The People

- **Karim** — friend of the developer (MHD). Wants to marry his dream girl.
- **MHD** — the developer. Building the device for Karim.
- **Her** — the recipient. Already approved by her parents for the marriage, but **has not met Karim**, has never seen him, and **is not yet convinced**.

## The Plan

Karim gifts her a small portable device. Inside lives a pixel-art version of **Mai and Karim themselves** — "us in another world." The two characters live, talk, and share little moments inside the device, like a relationship that already exists somewhere.

Karim controls every message from his phone over 4G. The device plays whatever he sends:

- **Voice** — Karim's recordings, or text he typed (server runs TTS to MP3)
- **Animations and scenes** — pixel art of the two of them, pre-rendered on the server
- **Photos** Karim picks — the server resizes/processes, the device just displays
- **Videos** Karim records — the server transcodes to MJPEG, device just plays
- **Music** Karim chooses
- **A first-boot greeting** when she powers it on

How fast Karim's name and identity appear is a **content decision he makes per-message**. The device platform itself is neutral — it'll play whatever he sends, named or unnamed, intimate or coy. He could keep it anonymous early to build curiosity, or be open from day one. Up to him, message by message.

The aim: charm her gradually until she **tells her dad she's interested in the marriage**.

That's the win condition.

## The Subtle Genius

Karim also wants to know if she's **carrying the device with her** — to uni, to the mall, to friends. If she is, that means she's emotionally attached. The device has on-demand **GPS location reporting** so Karim can quietly verify her interest without ever asking.

## Communication Rules

- **One-way only** until the win condition is met
- She receives, never sends back
- After she says yes to marriage: future versions could enable two-way (out of scope)
- Whether and when Karim is named on screen is a per-message content choice, not a hard system rule

## Related pages

- [Goals and success criteria](goals-and-success-criteria.md) — what counts as winning, what we're explicitly not building
- [System architecture](system-architecture.md) — how the device, server, and Karim's phone fit together
- [Behavior states](behavior-states.md) — what the device does at any given moment
- [Content types](content-types.md) — every kind of message Karim can send
