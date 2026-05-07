# Cute ESP — Project Wiki

> Everything we know about building Karim's mystery love device. Browse by topic.

A small portable gadget with a pixel art character that Karim secretly remote-controls over 4G. It charms his arranged-marriage match into saying yes. One-way comms only. Built on a LILYGO T-SIM A7670E R2 (ESP32 + 4G + GPS), with a 4″ ILI9488 display, MAX98357A I2S amp + speaker, and an 8000mAh LiPo. Polls a REST server every 5 minutes; ~27-day battery life.

If you're new here, start with **[Story and purpose](story-and-purpose.md)** and **[System architecture](system-architecture.md)**.

---

## Concept

- [Story and purpose](story-and-purpose.md) — the romance, the constraints, the win condition
- [Goals and success criteria](goals-and-success-criteria.md) — what we're optimizing for and what we're not building
- [System architecture](system-architecture.md) — the device + server + phone trio over 4G
- [Behavior states](behavior-states.md) — what the device is doing at any moment
- [Content types](content-types.md) — every kind of message Karim can send

## Hardware

- [Bill of materials](bill-of-materials.md) — what's bought, what's needed, specs
- [Hardware modifications](hardware-modifications.md) — battery swap, RST circuit, BL pull-down, bulk cap
- [Pin assignment](pin-assignment.md) — every GPIO accounted for
- [Wiring diagram](wiring-diagram.md) — every connection on one page
- [Power and battery life](power-and-battery-life.md) — current draw, daily budget, ~27-day target
- [Pre-build checklist](pre-build-checklist.md) — verify before soldering

## Firmware

- [Sleep/wake cycle](sleep-wake-cycle.md) — code-level deep sleep flow
- [Firmware patterns](firmware-patterns.md) — SPI, I2S, modem AT, display sleep, battery ADC
- [LILYGO examples reference](lilygo-examples.md) — which official examples to crib from

## Server & App

- [Server API](server-api.md) — REST contract device ↔ server ↔ app
- [Mobile app](mobile-app.md) — Karim's React Native screens

## Risk & Decisions

- [Problems and solutions](problems-and-solutions.md) — every problem found in planning
- [Gotchas](gotchas.md) — silent failures to watch for
- [Open decisions](open-decisions.md) — what's still TBD
- [To-buy list](to-buy.md) — items still to procure (battery, SD card, tools, etc.)

## Reference

- [Glossary](glossary.md) — every acronym and signal name
- [`lilygo-reference/`](lilygo-reference/) — inlined LILYGO upstream repo (examples, schematics, datasheets, libs)
- [`screenshots/`](screenshots/README.md) — labeled product photos and pinout diagrams
- [`archive/`](archive/) — older planning docs (superseded but kept for history)

---

## Repo layout

```
cute_esp/
├── docs/                     ← this wiki
│   ├── README.md             ← you are here
│   ├── *.md                  ← 20 wiki pages
│   ├── lilygo-reference/     ← inlined LILYGO upstream repo (read-only)
│   ├── screenshots/          ← labeled product photos
│   │   ├── lilygo-board/     ← 21 board, pinout, GPS-variant images
│   │   ├── display/          ← 5 display reference images
│   │   ├── speaker/          ← 3 SOTAMIA speaker images
│   │   └── amplifier/        ← 1 MAX98357A image
│   └── archive/              ← initial-planning-doc.md (superseded)
├── (firmware lives at repo root once we start writing it)
└── (server / mobile dirs as we add them)
```

## Page status legend

Pages mark hardware solutions as **SOLVED** (decided + verified), **PLANNED** (decided, awaiting build), or **DECIDED** (chosen approach, no build action). See [problems and solutions](problems-and-solutions.md) for the master table.
