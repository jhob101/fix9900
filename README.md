# uConsole trackpad keyboard firmware

Custom [ZMK](https://zmk.dev) firmware for the [hack2you uConsole trackpad keyboard kit](https://hack2you.tech/products/uconsole-trackpad-keyboard).
It brings over the features of [qmk-uconsole](https://github.com/jhob101/qmk-uconsole)
(gamepad mode, hold-Select scrolling, keyboard lock) and adds a few more.
Everything is in the firmware: nothing needs to run on the uConsole.

**For the V2.1 keyboard only.** It does not work on V1.1: use
[noodleboy91/uconsole-bb9900-keyboard-v1.1](https://github.com/noodleboy91/uconsole-bb9900-keyboard-v1.1)
for that. Other versions are untested.

> **Flash at your own risk.** This is a spare-time project, tested on one V2.1
> keyboard. Back up your current firmware first (see [Flashing](#flashing)).

## Contents

- [What it does](#what-it-does)
- [Changes in this fork](#changes-in-this-fork)
- [Building](#building)
- [Flashing](#flashing)
- [Desktop setup](#desktop-setup)
- [Customising](#customising)
- [Known limitations](#known-limitations)
- [Credits and licence](#credits-and-licence)

## What it does

USB only: Bluetooth is switched off.

### Keyboard mode

| Control | What it does |
| --- | --- |
| Trackpad | Moves the pointer. Pressing it is a left click. |
| L / R buttons | Left click / right click. |
| D-pad | Arrow keys. |
| Select | Hold and move the trackpad to scroll. The trackpad light goes out while scrolling. A tap sends the Select key (`KEY_FRONT` on Linux). |
| Start | Super. |
| A | The [kitty key](#kitty-key). |
| Y, B | Hold either and the D-pad moves the pointer. |
| Y tap | F21. |
| B tap | Middle click, sent on release, so it cannot drag. |
| X | F22. |
| Speaker | Volume down. With Shift, volume up. With Fn, mute. |
| Shift, Ctrl, Alt, AltGr | [One-shot modifiers](#one-shot-modifiers). |

The [middle-click firmware](#which-file) swaps two of these: pressing the
trackpad is a middle click and a tap on B is a left click.

### One-handed layout

A third firmware puts the mouse buttons and a scroll switch on the face
buttons, for right-hand use:

| Control | What it does |
| --- | --- |
| Y | Middle click |
| B | Right click |
| X | Scroll switch: press to scroll with the trackpad, press again to stop. The trackpad light is out while it is on. |
| A | F11 |

Y and B click on press and can be held to drag. Everything else is as above,
except that there is no D-pad pointer and no kitty key.

### One-shot modifiers

Tap Shift, Ctrl, Alt or AltGr and it applies to the next key only. Holding
works as normal. An unused tap expires after 60 seconds.

### Kitty key

A is a one-shot Ctrl+Shift for the [kitty](https://sw.kovidgoyal.net/kitty/)
terminal: tap A, then C, for Ctrl+Shift+C. Holding A works too. It covers:

- Letters `Q W E R T P S F G H J K L X C V B N M`
- `Enter`, the arrow keys, `[ ] / - = , .`
- `1` to `9`, sent as Ctrl+Shift+F1 to F9

Any other key is sent unchanged.

### Fn keys

Either Fn key works unless a row says otherwise.

| Keys | What it does |
| --- | --- |
| Fn + `1` to `0` | F1 to F10 |
| Fn + `-`, Fn + `=` | F11, F12 |
| Fn + Backspace | Delete |
| Fn + Tab | Caps Lock (no indicator light) |
| Fn + U, Fn + K | Page Up, Page Down |
| Fn + H, Fn + J | Home, End |
| Fn + I | Insert |
| Fn + `,`, Fn + `.` | Screen brightness down, up |
| Fn + Space | Key backlight on/off. It turns itself off after 30 seconds idle. |
| Fn + Speaker | Mute |
| Fn + Select | Print |
| Fn + Start | Media pause |
| Fn + G | [Gamepad mode](#gamepad-mode) on/off |
| Fn + Esc | [Lock](#lock) / unlock |
| Left Fn + D-pad up / down | Volume up / down |
| Left Fn + Left Alt | Super |
| Right Fn, then Left Fn | Restart the keyboard firmware |

### Gamepad mode

Fn + G toggles it. The keyboard also appears as a USB joystick
(`/dev/input/js0` on Linux):

| Control | Joystick |
| --- | --- |
| D-pad | X and Y axes |
| A, B, X, Y | Buttons 1 to 4 |
| Select | Button 5. Holding it still scrolls. |
| Start | Button 6 |

Modifiers are not one-shot in this mode. Everything else works as usual.

To test it with [sdl-jstest](https://github.com/Grumbel/sdl-jstest):

```sh
git clone https://github.com/Grumbel/sdl-jstest.git
cd sdl-jstest && mkdir build && cd build
cmake .. -DBUILD_SDL_JSTEST=OFF -DBUILD_SDL3_JSTEST=OFF && make
./sdl2-jstest --list && ./sdl2-jstest --test 0
```

### Lock

Fn + Esc locks the keyboard for carrying the uConsole around: keys, mouse
buttons and trackpad are ignored and the backlight goes off. Release Fn, then
Fn + Esc again to unlock. It does not blank the screen, and the keyboard always
starts unlocked.

### Bootloader

Two ways into the bootloader for [flashing](#flashing):

- **Left Alt + Right Alt + Start.** Not available in gamepad mode.
- **Hold Left Fn, then Right Fn, then press `\`.** The order matters.

Neither works while the keyboard is locked.

## Changes in this fork

A fork of [thoughtfix/fix9900](https://github.com/thoughtfix/fix9900), itself a
fork of the vendor's [Bill-lulu/uc9900](https://github.com/Bill-lulu/uc9900).
Added here:

- Gamepad mode, as a real USB joystick.
- Hold Select to scroll, replacing fix9900's click-the-trackpad toggle. An
  optional [scroll switch](#scroll-switch) is there for those who prefer one.
- Trackpad press as left click, middle click on B, and a second firmware with
  the two swapped.
- A one-handed firmware, with the mouse buttons on the face buttons.
- Start as Super, the kitty key and one-shot modifiers, replacing a
  [keyd](https://github.com/rvaiya/keyd) config.
- Y or B plus the D-pad as a mouse.
- The keyboard lock.
- No pointer jump when you lift your thumb, and pointer speed up 20%.
- Gentler scrolling that follows your finger speed: slow movement scrolls
  slowly.
- The trackpad light shows scroll mode.
- Bluetooth off.
- Fixes: Shift + Speaker no longer leaves Shift stuck, and the bootloader can
  no longer be entered by accident with `\`.

thoughtfix's own fixes are kept: see [`config/NOTES.md`](config/NOTES.md).
The changes to ZMK itself are in [jhob101/zmk](https://github.com/jhob101/zmk).

## Building

**You do not need to build anything.** Download a file from the
[latest release](https://github.com/jhob101/uconsole-bb9900-keyboard/releases/latest)
and go to [Flashing](#flashing).

### Which file

| File | What is different |
| --- | --- |
| `bb9900-zmk.uf2` | The standard firmware, as described above. |
| `bb9900-zmk-middle-click.uf2` | Pressing the trackpad is a middle click and a tap on B is a left click. |
| `bb9900-zmk-one-handed.uf2` | The [one-handed layout](#one-handed-layout). |

Build your own only to change the keymap or settings.

### On GitHub

1. Fork this repo and enable Actions on your fork.
2. Push a commit, or run the workflow from the **Actions** tab.
3. Download the `firmware` artifact from the finished run. It holds all three
   files.

### Locally

Needs Python 3, `cmake`, `ninja` and `git`. Tested on Linux x86-64.

```sh
# 1. west, the Zephyr build tool. If pip refuses to install system-wide:
#    python3 -m venv ~/zmk-venv && . ~/zmk-venv/bin/activate
pip install west

# 2. this repo, then the firmware source it builds against
git clone -b main https://github.com/jhob101/uconsole-bb9900-keyboard.git
cd uconsole-bb9900-keyboard
west init -l config
west update
west zephyr-export
pip install -r zephyr/scripts/requirements-base.txt

# 3. the ARM toolchain (Zephyr SDK 0.16.8)
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64_minimal.tar.xz
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/toolchain_linux-x86_64_arm-zephyr-eabi.tar.xz
tar xf zephyr-sdk-0.16.8_linux-x86_64_minimal.tar.xz
tar xf toolchain_linux-x86_64_arm-zephyr-eabi.tar.xz -C zephyr-sdk-0.16.8

# 4. build
cd ~/uconsole-bb9900-keyboard
west build -s zmk/app -d build -b bb9900 -- \
  -DZMK_CONFIG="$(pwd)/config" \
  -DZEPHYR_SDK_INSTALL_DIR="$HOME/zephyr-sdk-0.16.8"
```

The firmware is `build/zephyr/zmk.uf2`. Add `-p always` to `west build` after
changing a `.conf` file or `west.yml`.

### Publishing a release

Change the number in [`VERSION`](VERSION) on `main`. That builds the firmware
and publishes a GitHub Release with the files attached. The release text
comes from [`.github/release-notes.md`](.github/release-notes.md).

## Flashing

The keyboard cannot type while in its bootloader, so use SSH or a USB keyboard.

1. **Copy the `.uf2` to the uConsole**, for example with `scp`.

2. **Enter the bootloader.** On this firmware, hold Left Alt and Right Alt and
   press Start (or the [fallback](#bootloader)). On the vendor's stock
   firmware, Fn + `\`. A USB drive labelled `ADM840BOOT` appears.

3. **Mount the drive.**

   ```sh
   lsblk -o NAME,LABEL,SIZE        # look for ADM840BOOT, about 32M
   sudo mount /dev/sda /mnt        # use the name lsblk showed
   ```

4. **The first time, back up the current firmware.**

   ```sh
   cp /mnt/CURRENT.UF2 ~/keyboard-backup.uf2
   ```

5. **Copy the new firmware over.** The keyboard restarts on it by itself.

   ```sh
   sudo cp bb9900-zmk.uf2 /mnt/ && sync
   ```

To leave the bootloader without changing anything, or if the keyboard does not
come back, enter the bootloader again and copy your backup over.

If the drive reappears a few seconds after copying, that is a stuck unmount,
not a bad flash: reboot the uConsole and flash again.

## Desktop setup

Nothing to install. If you ran keyd for one-shot modifiers or a kitty key,
stop it, or the two will stack:

```sh
sudo systemctl disable --now keyd
```

## Customising

| File | What it holds |
| --- | --- |
| [`config/bb9900.keymap`](config/bb9900.keymap) | Every key on every layer, with comments. Start here. |
| [`config/bb9900.conf`](config/bb9900.conf) | Settings, each with a comment. |
| [`config/west.yml`](config/west.yml) | Which ZMK source to build against. |

Common changes:

| What | Where |
| --- | --- |
| Trackpad press and B tap | `TRACKPAD_PRESS` and `B_CLICK` at the top of the keymap: `LCLK`, `MCLK` or `RCLK` |
| Y, X, B and A | `Y_KEY`, `X_KEY`, `B_KEY` and `A_KEY` at the top of the keymap |
| Pointer speed | `CONFIG_TRACKPAD_SPEEDMULTIPLIER_HORIZONTAL` and `_VERTICAL`, in percent |
| Scroll speed | `CONFIG_TRACKPAD_SCROLL_SPEED`, in percent |
| Lift detection | `CONFIG_INPUT_A320_OFN_ENGINE`: `0xA0` on, `0x00` off |
| Trackpad light | `CONFIG_ZMK_TRACKPAD_SCROLL_LIGHT`: `n` leaves it alone |
| Backlight auto-off delay | `CONFIG_ZMK_IDLE_TIMEOUT`, in milliseconds |
| One-shot timeout | `release-after-ms` on `osm` and `osl` in the keymap |
| Kitty key list | `kitty_layer` in the keymap |

Avoid ZMK's `combos` feature: on this board it stops the Enter key working.

To see what a key sends: `sudo libinput debug-events --show-keycodes`

### Scroll switch

For a key that turns scrolling on and off, as on the stock firmware, bind it to
`&scroll_toggle`. For X on the standard layout, set `X_KEY` to
`&scroll_toggle`. Holding Select still scrolls, and the trackpad light is out
while the switch is on.

### Variants

<a name="example-a-one-handed-layout"></a>
The middle-click and one-handed firmware are the same keymap built with
`MIDDLE_CLICK_TRACKPAD` or `ONE_HANDED` defined, from
[`build.yaml`](build.yaml). The one-handed layout is the four `_KEY` values
under `#ifdef ONE_HANDED`, a worked example to copy for a layout of your own.

## Known limitations

- **Left Shift after Fn does nothing.** Press Shift before Fn, or use Right
  Shift.
- **Unlocking needs Fn released first.**
- **The Alt keys are one-shot**, so tapping both and then pressing Start also
  enters the bootloader.
- **The D-pad pointer and the trackpad share one mouse report**, so using both
  at once can feel uneven.
- **Based on a December 2023 ZMK.** Current ZMK documentation may not apply.

## Credits and licence

- [ZMK](https://github.com/zmkfirmware/zmk), the firmware underneath.
- [ZitaoTech](https://github.com/ZitaoTech/zmk), for the BB9900 keyboard and
  trackpad support.
- [Bill-lulu](https://github.com/Bill-lulu/uc9900), for the uConsole port.
- [Daniel Gentleman (thoughtfix)](https://github.com/thoughtfix/fix9900), for
  the fixes this fork starts from.
- [j1n6's qmk-uconsole](https://github.com/j1n6/qmk-uconsole), the model for
  gamepad mode, Select scrolling and the keyboard lock.

MIT licensed, the same as upstream. See [`LICENSE.txt`](LICENSE.txt).

The changes in this fork were written with the help of Claude (Anthropic).
