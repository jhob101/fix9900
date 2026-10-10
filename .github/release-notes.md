Firmware for the hack2you uConsole trackpad keyboard, **V2.1**.

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

## New in 1.5.1

- **Fixes the trackpad not registering a finger** on some keyboards, reported
  on 1.3.0 to 1.5.0. The cause was lift detection, which has been removed.

If you were affected, flashing this is all that is needed.

## Before you flash

Back up the firmware you are running now before you flash (copy `CURRENT.UF2`
off the bootloader drive). This is a spare-time project, tested on one
keyboard: flash at your own risk.
