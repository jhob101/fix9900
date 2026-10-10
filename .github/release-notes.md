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

## New in 1.2.0

- **B is a mouse button.** A tap on B is a middle click, or a left click in the
  middle-click firmware. It used to send F23. Holding B with the D-pad still
  moves the pointer.
- **Optional scroll switch.** A key can now turn trackpad scrolling on and off,
  as on the stock firmware. No key does by default: it takes a one-word change
  to the keymap and a build of your own. See
  [Scroll switch](https://github.com/jhob101/uconsole-bb9900-keyboard#scroll-switch),
  and the
  [one-handed layout](https://github.com/jhob101/uconsole-bb9900-keyboard#example-a-one-handed-layout)
  example.

## Before you flash

Back up the firmware you are running now before you flash (copy `CURRENT.UF2`
off the bootloader drive). This is a spare-time project, tested on one
keyboard: flash at your own risk.
