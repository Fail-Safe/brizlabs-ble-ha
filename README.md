# BrizLabs Micro String Lights (Home Assistant, local Bluetooth)

Custom integration v0.3.0, for BrizLabs RY micro string lights over BLE via BlueZ. No cloud.

**Status:** v0.3.0. Automated tests use a fake device (242 tests). Behaviour was verified against real hardware by the author; scene names are provisional (see below). Unofficial, not affiliated with BrizLabs or Tuya.

## Installation
Copy `custom_components/brizlabs` into your Home Assistant `config/custom_components/` directory (or add this repository to HACS as a custom repository), restart Home Assistant, then add the **BrizLabs Micro String Lights** integration. Requires a Bluetooth adapter reachable by BlueZ and `bleak`.

## Entities
- **Light** – on/off, brightness, HS colour, and an *effect* list (scenes + music modes). Choosing a scene effect sends its first captured (app default) palette and speed.
- **Scene palette** (select) – the 15 app preset palettes (global, offered for every scene): Green gradient, Rainbow 1, Rainbow 2, Rainbow 3, Red, Green, Blue, Cyan, Purple, Orange, Purple + Orange, White + Red, Red + Green, White + Blue, White + Red + Green. Labels are descriptive (app swatches are unlabeled). Older captured palettes that are not presets (e.g. the 7-colour Rainbow) remain as extra options. If the lights show a palette that was not captured (e.g. from `set_scene`), it appears as read-only `Custom: …`; re-selecting it does nothing. Only available while a scene is active. Changing it keeps scene, speed and brightness. The entry updates only when the device reports back.
- **Scene speed** (number, 0–100) – speed byte of the active scene; only available in scene mode.
- **Scene direction** (select, v0.3) – `Left` / `Right`; header byte 0 of the scene payload (0 = Right, 1 = Left). Palette and speed changes preserve it. Only available in scene mode.
- **Music mode** (select) – `Rock & Roll`, `Jazz`, `Classical`, `Energy`, `Roll`, `Pop`, `Rock`, `Kpop` (device_music 0–7, verified by ordered capture). The light effect list also still accepts `Music 0`…`Music 7` and legacy `Music` (= 7).
- **Music sensitivity** and **Pixel count** (numbers) – shown when the device reports them.

## Services
- `brizlabs.set_scene` – `address`, `effect` (scene name), `colors` (1–7 × `{hue 0-360, saturation 0-100, value 0-100}`), optional `speed` (default 50), `brightness` (1–100), `direction` (`Left`/`Right`, default Right).
- `brizlabs.paint_segments` – colour chosen zero-based segments (`address`, `segments`, `hue`, `saturation`, `value`, `segment_count`).
- `brizlabs.set_properties` – send raw device properties (advanced; no validation).

## Provisional names
Scene names come from the app's tile labels, mapped to scene IDs by the **order tapped during capture**, not confirmed against documentation. Some names may be wrong (e.g. `Fall`/`Fade`/`Jump`, IDs 08/07/06); `Scene 16` had no readable label in the screenshot. Music names are verified (v0.3). Scene labels remain provisional. The 4-byte header (direction, scene id, change mode, speed) and 10-byte colour units are inferred from captures. A custom two-color scene was accepted and reported by the device; other combinations and the full 1–7 colour range remain unverified.

## Known limits
- Names may change in later versions, which would change effect names in automations.
- Tests use a fake device and synthetic fixtures; run with `pip install pytest pytest-asyncio homeassistant bleak` then `python -m pytest`.
