# Droid Depot BLE Web Controller

A single-file HTML Web Bluetooth controller for Disney Galaxy's Edge Droid Depot droids (BB-series). Reverse engineered from [Baptiste Laget's droid.bap.dev](https://droid.bap.dev) and the corresponding article 
https://medium.com/@baptistelaget/controlling-disneys-droids-from-droid-depots-with-webbluetooth-febbabe50587

## Requirements

- Chrome or Edge (desktop) — Web Bluetooth API required
- A Droid Depot BB-unit (BB-8, BB-9E, etc.)
- Droid must be powered on and awake

## Usage

Open `droid-controller.html` in Chrome or Edge. No server, build step, or install needed.

1. **Select droid type** — BB-series or R-series (panel 01)
2. **Connect** — click `CONNECT DROID`, select `DROID` from the browser popup
3. The init handshake runs automatically; the droid will play a sound on success

## Panels

### 01 — Connection
Connect/disconnect and droid type selection. Status indicator shows current BLE state.

### 02 — Sound
9 sound banks (BOOT, HAPPY, SAD, ALERT, CHATTY, SCARED, EXCITED, ANGRY, SPECIAL), 5 slots each. Select a bank first, then a slot to play. Lights are triggered automatically by the droid firmware — they are not separately controllable over BLE.

### 03 — Motion
Pivot left/right, drive forward/back, sweep, and centre. Speed and duration sliders control power and how long each command runs before stopping.

### 04 — Autonomous Mode
Periodically fires random sounds and movements on a configurable interval (3–60s).

- **FULL** — random sounds, pivot, sweep, and forward/back driving
- **STATIC** — random sounds and left/right pivot only (no driving)

### 05 — Raw Commands
Preset buttons for the confirmed `2942` movement format, plus a manual hex textarea for sending arbitrary commands. One command per line; optionally append `,<delay_ms>` for inter-command delays.

### 06 — Mission Log
Live TX/RX log of all BLE traffic.

## BLE Protocol

Confirmed from bap.dev Angular bundle.

| Item | Value |
|------|-------|
| Service UUID | `09b600a0-3e42-41fc-b474-e9c0c8f0c801` |
| Write characteristic | `09b600b1-3e42-41fc-b474-e9c0c8f0c801` |
| Notify characteristic | `09b600b0-3e42-41fc-b474-e9c0c8f0c801` |

**Sound** — two-step: `sendBank` (`2742 0F44 4400 1F{bank}`) × 2, then `playSound` (`2742 0F44 4400 18{slot}`) × 2, ~50ms apart.

**Movement** — `2942 0546 {dir}{motor}{power} 012C 0000`
- `dir`: `0` = forward, `8` = backward (single nibble)
- `motor`: `0` or `1` (BB body wheels), `2` (R-series head)
- `power`: 2-char hex, `80` = half, `FF` = full

## Notes

- Requires a browser with Web Bluetooth support. Firefox does not support this API.
- Init sequence must complete before movement/sound commands will work.
- All motion commands use a 10ms inter-motor gap matching the bap.dev source.
