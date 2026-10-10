Firmware for the hack2you uConsole trackpad keyboard, **V2.1**.

**It does not work on the V1.1 keyboard.** For V1.1, use
[noodleboy91/uconsole-bb9900-keyboard-v1.1](https://github.com/noodleboy91/uconsole-bb9900-keyboard-v1.1),
a fork of this firmware for that version. Other versions are untested.

## Which file

| File | What is different |
| --- | --- |
| `bb9900-zmk.uf2` | The standard firmware. Pressing the trackpad is a left click and a tap on B is a middle click. |
| `bb9900-zmk-middle-click.uf2` | Those two swapped: pressing the trackpad is a middle click and a tap on B is a left click. |
| `bb9900-zmk-one-handed.uf2` | The [one-handed layout](https://github.com/jhob101/uconsole-bb9900-keyboard#one-handed-layout): Y middle click, B right click, X a scroll switch, A F11. |

Download one and follow the
[flashing instructions](https://github.com/jhob101/uconsole-bb9900-keyboard#flashing).
What every key does is in the
[README](https://github.com/jhob101/uconsole-bb9900-keyboard#what-it-does).

## New in 1.4.0

- **A one-handed firmware**, `bb9900-zmk-one-handed.uf2`, with the mouse
  buttons and a scroll switch on the face buttons for right-hand use. It gives
  up the D-pad pointer and the kitty key.

The other two files are unchanged from 1.3.0.

## Before you flash

Back up the firmware you are running now before you flash (copy `CURRENT.UF2`
off the bootloader drive). This is a spare-time project, tested on one
keyboard: flash at your own risk.
