Firmware for the hack2you uConsole trackpad keyboard, **V2.1**.

It may work on other versions of the keyboard, but it has only been tested on
V2.1.

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
