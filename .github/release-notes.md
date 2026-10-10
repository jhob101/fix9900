Firmware for the hack2you uConsole trackpad keyboard, **V2.1**.

**It does not work on the V1.1 keyboard.** For V1.1, use
[noodleboy91/uconsole-bb9900-keyboard-v1.1](https://github.com/noodleboy91/uconsole-bb9900-keyboard-v1.1),
a fork of this firmware for that version. Other versions are untested.

## Which file

The two files differ only in where left click and middle click sit:

| File | Pressing the trackpad | Tapping B |
| --- | --- | --- |
| `bb9900-zmk.uf2` | Left click | Middle click |
| `bb9900-zmk-middle-click.uf2` | Middle click | Left click |

Download one and follow the
[flashing instructions](https://github.com/jhob101/uconsole-bb9900-keyboard#flashing).
What every key does is in the
[README](https://github.com/jhob101/uconsole-bb9900-keyboard#what-it-does).

## New in 1.3.0

- **No pointer jump when you lift your thumb.** The trackpad sensor's own lift
  detection is now switched on. A bug that sent the pointer backwards on fast
  movements is fixed too.
- **The trackpad light shows scroll mode.** It goes out while you scroll and
  comes back afterwards.
- **Pointer speed** is slightly lower: 20% above stock, down from 30%.

Each of these can be changed or switched off in `config/bb9900.conf` if you
build your own: see
[Customising](https://github.com/jhob101/uconsole-bb9900-keyboard#customising).

## Before you flash

Back up the firmware you are running now before you flash (copy `CURRENT.UF2`
off the bootloader drive). This is a spare-time project, tested on one
keyboard: flash at your own risk.
