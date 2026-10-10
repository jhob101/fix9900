Firmware for the hack2you uConsole trackpad keyboard, **V2.1**.

**It does not work on the V1.1 keyboard.** For V1.1, use
[noodleboy91/uconsole-bb9900-keyboard-v1.1](https://github.com/noodleboy91/uconsole-bb9900-keyboard-v1.1),
a fork of this firmware for that version. Other versions are untested.

## Which file

The two files differ only in what pressing the trackpad does:

| File | Pressing the trackpad |
| --- | --- |
| `bb9900-zmk.uf2` | Left click |
| `bb9900-zmk-middle-click.uf2` | Middle click |

Download one and follow the
[flashing instructions](https://github.com/jhob101/uconsole-bb9900-keyboard#flashing).
What every key does is in the
[README](https://github.com/jhob101/uconsole-bb9900-keyboard#what-it-does).

Back up the firmware you are running now before you flash (copy `CURRENT.UF2`
off the bootloader drive). This is a spare-time project, tested on one
keyboard: flash at your own risk.
