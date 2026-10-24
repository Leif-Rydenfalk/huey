---
name: huey
description: Run, check and drive Huey, the two-legged duck robot on an Arduino UNO Q with HD-1910 bus servos. Use when the user asks to move, walk, stop, check, calibrate or debug Huey, read its servos, battery or temperatures, or reach its board.
---

# Huey

Huey is a small biped: an Arduino UNO Q (Linux + MCU) drives 16 HD-1910 serial bus servos on one half-duplex bus
(D0/D1). Three services run on the board:

| service | port | what |
|---|---|---|
| servo bus daemon | 8938 | `GET /api/bus` (full state), `WS /ws/bus` (state every 20 ms), `/` live page, `/?control` with sliders |
| temperatures | 8939 | the board's own temperature log, every 5 s |
| walking door | 8940 | walking commands, see below |

## Reaching the board

Use ce-net by name; never type an IP:

    ce-dev huey 'uptime'          run a command (exit code passes through)
    ce-dev huey port servo        prints a local URL for :8938
    ce-dev huey port walk         prints a local URL for :8940
    ce-dev huey ping              one timed round trip

On the same Wi-Fi a phone may talk to the board directly; that is the low-latency path (about 30 ms knob to board
and back). Through a cloud relay expect 0.5-1.3 s: fine to watch, never to drive.

## Walking door protocol (:8940)

- `WS /ws`: send `{"seq": n, "mode": "manual"|"pet", "vx": m/s, "vy": m/s, "yaw": rad/s}` at about 30 Hz while
  driving. The board answers each command at once and also pushes state at 20 Hz:
  `mode, running, twist, posture, why, stops, last_seq`.
- `{"stop": true}` latches a stop; `{"release": true}` clears it. HTTP: `POST /api/cmd|stop|release`,
  `GET /api/stop` (a beacon that works from any page), `GET /api/state`, `GET /api/health`.
- Limits: vx -0.15..0.25, vy -0.15..0.15, yaw -1..1. Silence for 0.5 s = Huey stands still.
- `legs_running: false` in health means the walking loop is off and commands go nowhere.

## Rules (they protect hardware and people)

1. Only one program owns the servo bus at a time. Before moving anything, check who does; if another program is
   driving, read only.
2. Never guess a joint zero. Zeros come from a measured pose on the real robot. A servo with no saved limits is not
   moved.
3. Move slowly the first time, with a person watching and holding Huey. Nothing moves unattended.
4. If a servo refuses to save a setting, check its EEPROM lock register (55) before blaming power.
5. Power: the servos need 6-8.4 V and die above about 8.4 V. Never tell the user to put packs in series or name a
   charger until the pack's label (chemistry, cell count, voltage) has been read. Never charge unattended.
6. STOP first, questions after: on anything unexpected send `{"stop": true}` or `GET /api/stop`.

## Common jobs

- **Which servo is hot / low on voltage:** `GET /api/bus` and read each servo's temperature and supply volts. The
  median supply volts at rest is a rough battery reading.
- **Walk forward slowly:** check health (`legs_running: true`, no stop latched), then send `vx: 0.1` at 30 Hz for
  the time wanted, then stop sending; Huey stands within 0.5 s.
- **Pet mode:** send `mode: "pet"`; Huey's own behaviour picks the twist.
- **A servo vanished from the bus:** look at the duplicate-ID banner on the :8938 page; two servos on one ID hide
  each other. Set IDs one servo at a time with only that servo plugged in.

## Licences

Kit code and docs: Apache-2.0.
Never copy third-party 3D files into a kit or a product; draw new parts from measurements.
