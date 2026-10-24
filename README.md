# Huey: a walking duck robot you build from a component kit

Huey is a small two-legged robot about the size of a shoebox. You buy the kit of parts, print the shell yourself, and
run it from your phone. It walks with an open reinforcement-learning policy and runs on an Arduino UNO Q.

**Status, 2026-10-24:** Huey moves servos, streams them live to a phone at 30+ updates per second, and takes joystick
commands in about 30 ms. He has not walked on his own yet: the balance policy runs next, once the joint zeros are
saved again. All 26 printed parts are in [`hardware/`](hardware/README.md) as STEP files.

## What is in the box

| part | qty | notes |
|---|---|---|
| Arduino UNO Q (Linux + MCU on one board) | 1 | runs everything: servo bus, walking, Wi-Fi, the phone app |
| HD-1910 serial bus servos | 16 | legs 2 x 5 or 6, head 5 (neck pitch, head pitch, yaw, roll, jaw) |
| servo bus hat for the UNO Q | 1 | half-duplex bus on D0/D1 |
| battery pack with protection board | 1 | the pack must give 6-8.4 V to the servos. Check the label before wiring anything |
| bearings, screws, cables | set | list per part in the build book |

You print: the shell, legs and head from [`hardware/parts/`](hardware/README.md) (PLA, TPU where that table says). The head is the square "pixel" design.

## Software

The board software and the app move into this repo as they are cleaned up; until then the table says what each does.

| piece | what it does | licence |
|---|---|---|
| servo bus daemon (Rust, on the board) | owns the servo bus, 50 Hz live state over WebSocket on :8938, clamps every goal to saved limits | Apache-2.0 (this kit) |
| walking door (on the board, :8940) | takes `{vx, vy, yaw}` from a phone at 30 Hz, stops within 0.5 s of silence | Apache-2.0 (this kit) |
| Huey app (phone and desktop) | Home (battery, temperatures, servos), Drive (joystick, Manual/Pet, STOP), Sim, Servos | Apache-2.0 (this kit) |
| walking policy (ONNX) | the neural network that turns a twist command into joint targets | Apache-2.0 |

## Reach the board without setting up networking: ce-net

The board joins ce-net, a peer-to-peer device mesh. From your laptop you reach it by name, with no IP address, port
forwarding or ssh keys to type:

    ce-dev huey                 a shell on the board
    ce-dev huey 'uptime'        run one command
    ce-dev huey cp a.txt :/tmp/ copy a file
    ce-dev huey port servo      the servo page on a local URL

Measured 2026-10-04 on one Wi-Fi: servo state round trip 47 ms over ce-net, 53 ms on plain Wi-Fi. Away from home the
same command goes through a relay. On the same Wi-Fi, the phone talks to the board directly and nothing leaves the room.

## Safety

- Every servo goal is clamped to that servo's saved limits; a servo with no limits is refused.
- Letting go of the joystick stops Huey within 0.5 s. STOP latches until you press Release. STOP ALL turns off torque.
- Joint zeros are measured on the real robot, never guessed.
- Never charge or run the pack unattended, and never put two packs in series without checking their voltage:
  the servos die above about 8.4 V.

## The Claude skill

`skill/huey/SKILL.md` is a skill for Claude Code: drop the folder in `~/.claude/skills/` and Claude knows Huey's
hardware, ports, protocols and safety rules, so you can say "make Huey walk forward slowly" or "which servo is hot".

## Licences

- Code and docs in this repo: Apache-2.0 (`LICENSE`). Upstream software keeps its notices (`NOTICE`).
- **Printed parts:** all 26 in [`hardware/`](hardware/README.md) are our own designs, drawn from our own
  measurements, under CERN-OHL-W-2.0 (`hardware/LICENSES/CERN-OHL-W-2.0.txt`). Hardware docs and pictures are
  CC BY 4.0 (`hardware/LICENSES/CC-BY-4.0.txt`). The per-part table is in `hardware/README.md`.
