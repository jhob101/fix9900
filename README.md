# uConsole trackpad keyboard firmware

Custom firmware for the [hack2you uConsole trackpad keyboard kit](https://hack2you.tech/products/uconsole-trackpad-keyboard):
the replacement uConsole keyboard with an optical trackpad in place of the
trackball.

**It is for the V2.1 keyboard, and it does not work on V1.1.** If you have a
V1.1 keyboard, use [noodleboy91/uconsole-bb9900-keyboard-v1.1](https://github.com/noodleboy91/uconsole-bb9900-keyboard-v1.1),
a fork of this firmware for that version. Other versions are untested.

It carries over the features of the [qmk-uconsole](https://github.com/jhob101/qmk-uconsole)
firmware for the original keyboard (gamepad mode, hold-Select scrolling, the
keyboard lock) and adds a few of its own, all inside the firmware. Nothing
needs to run on the uConsole itself: no keyd, no remapping daemon.

> **Flash at your own risk.** This is a spare-time project, developed and tested
> on one V2.1 keyboard. Keep a copy of the firmware you are running now before
> you flash anything (see [Flashing](#flashing)).

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

The firmware is [ZMK](https://zmk.dev). The keyboard connects over USB only;
Bluetooth is switched off.

### Keyboard mode

This is the normal mode. Letters, numbers and punctuation are as printed.

| Control | What it does |
| --- | --- |
| Trackpad | Moves the pointer. Pressing it is a left click. |
| L / R buttons | Left click / right click. |
| D-pad | Arrow keys. |
| Select | Hold it and move the trackpad to scroll. A tap on its own sends the Select key (`KEY_FRONT` on Linux). |
| Start | Super. |
| A | The [kitty key](#kitty-key). |
| Y, B | Hold either and the D-pad moves the pointer. |
| Y tap | F21. |
| B tap | Middle click. It is sent as you let go, so it clicks but cannot drag. |
| X | F22. |
| Speaker | Volume down. With Shift, volume up. With Fn, mute. |
| Shift, Ctrl, Alt, AltGr | [One-shot modifiers](#one-shot-modifiers). |

The [middle-click firmware](#which-file) swaps two of these: pressing the
trackpad is a middle click, and a tap on B is a left click.

### One-shot modifiers

Tap Shift, Ctrl, Alt or AltGr and let go, and it applies to the next key only.
Tap Shift, then `a`, and you get `A`. Holding a modifier down works as normal.
A tap that is never used expires after 60 seconds.

### Kitty key

The A button is a one-shot Ctrl+Shift for the [kitty](https://sw.kovidgoyal.net/kitty/)
terminal, whose shortcuts are all Ctrl+Shift+something. Tap A, then C, and the
keyboard sends Ctrl+Shift+C. Holding A works too.

It covers these keys:

- Letters `Q W E R T P S F G H J K L X C V B N M`
- `Enter`, the arrow keys, `[ ] / - = , .`
- `1` to `9`, which send Ctrl+Shift+F1 to Ctrl+Shift+F9

Any other key is sent unchanged and ends the one-shot.

### Fn keys

Either Fn key works unless a row says otherwise.

| Keys | What it does |
| --- | --- |
| Fn + `1` to `0` | F1 to F10 |
| Fn + `-`, Fn + `=` | F11, F12 |
| Fn + Backspace | Delete |
| Fn + Tab | Caps Lock (there is no indicator light) |
| Fn + U, Fn + K | Page Up, Page Down |
| Fn + H, Fn + J | Home, End |
| Fn + I | Insert |
| Fn + `,`, Fn + `.` | Screen brightness down, up |
| Fn + Space | Key backlight on/off. When on, it switches itself off after 30 seconds without a key press and comes back on the next one. |
| Fn + Speaker | Mute |
| Fn + Select | Print |
| Fn + Start | Media pause |
| Fn + G | [Gamepad mode](#gamepad-mode) on/off |
| Fn + Esc | [Lock](#lock) / unlock |
| Left Fn + D-pad up / down | Volume up / down |
| Left Fn + Left Alt | Super |
| Right Fn, then Left Fn | Restart the keyboard firmware |
| Left Alt + Right Alt + Start | Enter the [bootloader](#bootloader) (no Fn needed) |
| Hold Left Fn, then also Right Fn, then press `\` | Enter the bootloader, fallback route |

### Gamepad mode

Fn + G toggles it. The keyboard shows up to the uConsole as a joystick as well
as a keyboard and mouse (`/dev/input/js0` on Linux), and in gamepad mode:

| Control | Joystick |
| --- | --- |
| D-pad | X and Y axes. If two opposite directions are held, the last one pressed wins. |
| A, B, X, Y | Buttons 1, 2, 3, 4 |
| Select | Button 5. Holding it still scrolls with the trackpad. |
| Start | Button 6 |

Shift, Ctrl and Alt are plain modifiers here, not one-shot. Everything else
(typing, the trackpad, the Fn keys) works as in keyboard mode.

Games will see a new controller, so button mappings in SDL or RetroArch need
setting once.

To watch the axes and buttons live, use
[sdl-jstest](https://github.com/Grumbel/sdl-jstest). It needs `cmake` and the
SDL2 and ncurses development packages to build:

```sh
git clone https://github.com/Grumbel/sdl-jstest.git
cd sdl-jstest && mkdir build && cd build
cmake .. -DBUILD_SDL_JSTEST=OFF -DBUILD_SDL3_JSTEST=OFF   # SDL2 version only
make
./sdl2-jstest --list       # find the keyboard's joystick number
./sdl2-jstest --test 0     # test joystick 0
```

### Lock

Fn + Esc locks the keyboard, for carrying the uConsole around. Locking:

1. switches the trackpad off,
2. turns the key backlight off,
3. ignores every key and both mouse buttons.

Fn + Esc again unlocks: the trackpad comes back and the backlight returns to
however it was. Release Fn between locking and unlocking.

The lock is inside the keyboard only. It sends nothing to the uConsole, so it
does not blank the screen. It is not remembered across a power cycle, so the
keyboard always starts unlocked.

### Bootloader

Two shortcuts restart the keyboard into its bootloader for
[flashing](#flashing):

- **Left Alt + Right Alt + Start**, the same shortcut as the qmk-uconsole
  firmware. Hold both Alt keys and press Start.
- **Left Fn + Right Fn + `\`**, as a fallback. Hold Left Fn, then also hold
  Right Fn, then press `\`. The order matters: Right Fn first restarts the
  firmware instead.

Neither works while the keyboard is locked, and the first does not work in
gamepad mode, where Start is a joystick button.

Because the Alt keys are [one-shot](#one-shot-modifiers), tapping both and then
pressing Start also counts. The Fn route cannot be set off that way, as Fn keys
are never one-shot.

## Changes in this fork

This is a fork of [thoughtfix/fix9900](https://github.com/thoughtfix/fix9900),
which is itself a fork of the vendor's [Bill-lulu/uc9900](https://github.com/Bill-lulu/uc9900).

Added here:

- **Gamepad mode** on Fn + G, as a real USB joystick.
- **Hold Select to scroll**, replacing fix9900's click-the-trackpad toggle.
- **Trackpad press is left click** again, with **middle click on B**. A
  second firmware has the two swapped.
- **An optional scroll switch**, for anyone who prefers the stock firmware's
  toggle. No key has it by default: see [Scroll switch](#scroll-switch).
- **Start is Super** (it was media Play).
- **The kitty key** on A, and **one-shot modifiers**. These replace a
  [keyd](https://github.com/rvaiya/keyd) config on the uConsole.
- **Y / B plus the D-pad as a mouse.**
- **The keyboard lock** on Fn + Esc, covering the keys, trackpad and backlight.
- **Bluetooth switched off.** The right Fn pairing keys (Esc, 1 to 4) now match
  left Fn.
- **Trackpad pointer speed** raised by 30%.
- **Volume key fix**: Shift + Speaker no longer leaves Shift stuck on.
- **Bootloader shortcuts made safe.** In fix9900, Ctrl + `\` or Alt + `\`
  alone entered the bootloader, as did Fn + `\`. The backslash key is now
  just a backslash, and the shortcuts are Left Alt + Right Alt + Start or
  both Fn keys with `\`.

Kept from thoughtfix's fix9900:

- Speaker key as volume down, Shift for up, Fn for mute.
- Caps Lock no longer interferes with the trackpad.
- Working Enter key, right Alt and right Ctrl under Fn, and Fn + Start.
- Smoother trackpad scrolling, locked to one axis at a time.

thoughtfix's notes on those fixes are in [`config/NOTES.md`](config/NOTES.md).

Several of these needed changes to ZMK itself. Those live in
[jhob101/zmk](https://github.com/jhob101/zmk), which this repo builds against.

## Building

**You do not need to build anything to use the firmware.** Each
[release](https://github.com/jhob101/uconsole-bb9900-keyboard/releases/latest)
has ready-made firmware attached. Download the file you want and go straight to
[Flashing](#flashing).

### Which file

The two files differ only in where left click and middle click sit:

| File | Pressing the trackpad | Tapping B |
| --- | --- | --- |
| `bb9900-zmk.uf2` | Left click | Middle click |
| `bb9900-zmk-middle-click.uf2` | Middle click | Left click |

The L and R buttons are left and right click in both.

Build it yourself if you want to change the keymap or settings. The output of
either route below is one file, `bb9900-zmk.uf2` (GitHub) or `zmk.uf2` (local
build).

### On GitHub (no tools to install)

1. Fork this repo and enable Actions on your fork.
2. Push a commit, or open the **Actions** tab, pick the workflow and choose
   **Run workflow**.
3. Open the finished run and download the `firmware` artifact. It is a zip
   containing both files from the [table above](#which-file).

A build takes about four minutes.

### Locally

You need Python 3, `cmake`, `ninja` and `git`. These steps were run on Linux
x86-64.

```sh
# 1. west, the Zephyr build tool. On Debian or Raspberry Pi OS, where pip
#    refuses to install system-wide, do this in a virtual environment first:
#    python3 -m venv ~/zmk-venv && . ~/zmk-venv/bin/activate
pip install west

# 2. this repo, then the firmware source it builds against
git clone -b main https://github.com/jhob101/uconsole-bb9900-keyboard.git
cd uconsole-bb9900-keyboard
west init -l config
west update
west zephyr-export
pip install -r zephyr/scripts/requirements-base.txt

# 3. the ARM toolchain (Zephyr SDK 0.16.8), unpacked to ~/zephyr-sdk-0.16.8
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

The firmware is `build/zephyr/zmk.uf2`.

Two things that are easy to get wrong:

- Run `west init -l config` from the repo root. The `config` argument is the
  folder inside the repo, not a name for the clone.
- Pass `ZEPHYR_SDK_INSTALL_DIR` with `-D` after the `--`. Setting it as an
  environment variable is ignored.

After changing a `.conf` file or `west.yml`, add `-p always` to `west build`
for a clean rebuild.

### Publishing a release

Change the number in the [`VERSION`](VERSION) file on `main`, for example from
`1.0.0` to `1.0.1`. You can edit it straight on GitHub. That builds the
firmware and publishes a GitHub Release named `v1.0.1` with the `.uf2`
attached.

The release is built against whatever `jhob101/zmk` `main` is at that moment.

The text on the release page comes from
[`.github/release-notes.md`](.github/release-notes.md). Editing that file on
its own updates the text of the current release without touching its firmware
file.

## Flashing

The keyboard stops working as a keyboard while it is in its bootloader, so you
need another way to type on the uConsole. SSH from another computer is the
easiest. A USB keyboard also works.

1. **Copy the new `.uf2` to the uConsole**, for example with `scp`. Get it
   from the [latest release](https://github.com/jhob101/uconsole-bb9900-keyboard/releases/latest)
   or from your own build.

2. **Put the keyboard into its bootloader.**
   - On this firmware: hold Left Alt and Right Alt and press Start. If that
     ever fails, hold Left Fn, then also Right Fn, then press `\`.
   - On the vendor's stock firmware: Fn + `\`.

   A small USB drive labelled `ADM840BOOT` appears.

3. **Find and mount the drive.**

   ```sh
   lsblk -o NAME,LABEL,SIZE        # look for ADM840BOOT, about 32M
   sudo mount /dev/sda /mnt        # use the name lsblk showed
   ```

4. **The first time, back up what is on the keyboard now.**

   ```sh
   cp /mnt/CURRENT.UF2 ~/keyboard-backup.uf2
   ```

5. **Copy the new firmware onto the drive.**

   ```sh
   sudo cp bb9900-zmk.uf2 /mnt/ && sync
   ```

   Use `bb9900-zmk-middle-click.uf2` here if that is the one you chose.

   The drive disappears by itself and the keyboard restarts on the new
   firmware a second or two later.

### Leaving the bootloader without flashing

Copy any firmware `.uf2` onto the drive, including your backup. The keyboard
restarts into whatever you copied.

### If something goes wrong

- **The keyboard does not come back.** The bootloader shortcuts are handled by
  the keyboard itself, so try Left Alt + Right Alt + Start again (or Left Fn,
  Right Fn, `\`) and copy your backup over.
- **The drive reappears a few seconds after copying,** with USB errors in
  `journalctl`. That is a stuck unmount, not a bad flash. Reboot the uConsole
  and flash the same file again.

## Desktop setup

The firmware needs nothing installed on the uConsole. One thing to undo if you
are coming from an earlier setup:

### Remove keyd, if you used it

If you ran keyd for one-shot modifiers or a kitty key with an earlier keyboard,
stop it, or the two will stack:

```sh
sudo systemctl disable --now keyd
```

## Customising

| File | What it holds |
| --- | --- |
| [`config/bb9900.keymap`](config/bb9900.keymap) | Every key on every layer, with comments. Start here. |
| [`config/bb9900.conf`](config/bb9900.conf) | Settings: pointer speed, backlight, gamepad, Bluetooth. |
| [`config/bb9900.overlay`](config/bb9900.overlay) | The key matrix wiring. Leave it alone unless a key is dead. |
| [`config/west.yml`](config/west.yml) | Which ZMK source to build against. |

Common changes:

- **Trackpad press and B tap:** `TRACKPAD_PRESS` and `B_CLICK` near the top of
  the keymap, each `LCLK`, `MCLK` or `RCLK`. Change the pair after the
  `#else`. The middle-click firmware is the same keymap with
  `MIDDLE_CLICK_TRACKPAD` defined from [`build.yaml`](build.yaml), which is
  also where to add another variant.
- **Pointer speed:** `CONFIG_TRACKPAD_SPEEDMULTIPLIER_HORIZONTAL` and
  `_VERTICAL` in `bb9900.conf`, in percent. Above about 133 the smallest
  pointer step becomes two pixels.
- **Scroll speed:** `CONFIG_TRACKPAD_SCROLL_INTERVAL` in `bb9900.conf`. Higher
  is slower.
- **Backlight auto-off delay:** `CONFIG_ZMK_IDLE_TIMEOUT` in `bb9900.conf`, in
  milliseconds.
- **One-shot timeout:** `release-after-ms` on `osm` and `osl` in the keymap.
- **Kitty key list:** the `kitty_layer` block in the keymap.

The files under `config/boards/bb9900/` are not used by the build. The board
definition comes from the ZMK fork.

Avoid ZMK's `combos` feature on this board: thoughtfix found that adding any
combo stopped the Enter key working.

To check what a key actually sends, on the uConsole:

```sh
sudo libinput debug-events --show-keycodes
```

### Scroll switch

The stock firmware had a scroll switch: press a key once and the trackpad
scrolls, press it again and it moves the pointer. This firmware scrolls while
Select is held instead, but the switch is there if you want it. It is already
defined in the keymap as `&scroll_toggle`, and no key uses it.

To put it on X, find `default_layer` in `config/bb9900.keymap` and change
`&kp F22` to `&scroll_toggle`. Any other key works the same way. Holding Select
still scrolls as well.

Then build and flash as in [Building](#building).

### Example: a one-handed layout

A worked example, for using the uConsole with the right hand only: the mouse
buttons and the scroll switch all go on the four face buttons.

| Control | Wanted | Change |
| --- | --- | --- |
| Trackpad press | Left click | None, it already is. |
| Y | Middle click | `&dpad_mouse MOUSE F21` becomes `&mkp MCLK` |
| X | Scroll switch | `&kp F22` becomes `&scroll_toggle` |
| B | Right click | `&dpad_mouse_click MOUSE B_CLICK` becomes `&mkp RCLK` |
| A | F11 | `&osl KITTY` becomes `&kp F11` |
| Select | Hold to scroll | None, it already is. |
| Start | Super (the app menu) | None, it already is. |

All four changes are in the first two rows of `default_layer`. The keys on
those rows run Left, L button, trackpad press, Y, X, then Up, Down, Right,
R button, B, A. Before:

```
            &kp LEFT                      &mkp LCLK         &mkp TRACKPAD_PRESS  &dpad_mouse MOUSE F21   &kp F22
&kp UP   &kp DOWN &kp RIGHT               &mkp RCLK                       &dpad_mouse_click MOUSE B_CLICK  &osl KITTY
```

After:

```
            &kp LEFT                      &mkp LCLK         &mkp TRACKPAD_PRESS  &mkp MCLK   &scroll_toggle
&kp UP   &kp DOWN &kp RIGHT               &mkp RCLK                       &mkp RCLK  &kp F11
```

Leave the other layers alone. Two things are given up: Y and B no longer move
the pointer with the D-pad, and there is no [kitty key](#kitty-key). Y and B
become ordinary mouse buttons, so they click on press and can be held to drag.
Gamepad mode is unaffected, since its layer sits on top.

## Known limitations

- **V2.1 keyboard only.** It does not work on V1.1: use
  [noodleboy91/uconsole-bb9900-keyboard-v1.1](https://github.com/noodleboy91/uconsole-bb9900-keyboard-v1.1)
  for that. Other versions of the kit are untried.
- **Left Shift after Fn does nothing.** Press Shift before Fn, or use Right
  Shift. This is inherited from the vendor keymap.
- **Unlocking needs Fn released first.** Holding Fn and tapping Esc twice locks
  but does not unlock.
- **The gamepad is USB only**, and Bluetooth is off altogether.
- **The D-pad pointer and the trackpad share one mouse report**, so using both
  at once can feel uneven.
- **Based on a December 2023 ZMK.** Current ZMK documentation and modules may
  not apply.

## Credits and licence

- [ZMK](https://github.com/zmkfirmware/zmk), the firmware underneath.
- [ZitaoTech](https://github.com/ZitaoTech/zmk), for the BB9900 keyboard and
  trackpad support.
- [Bill-lulu](https://github.com/Bill-lulu/uc9900), for the uConsole port.
- [Daniel Gentleman (thoughtfix)](https://github.com/thoughtfix/fix9900), for
  the fixes this fork starts from.
- [j1n6's qmk-uconsole](https://github.com/j1n6/qmk-uconsole), whose gamepad
  mode, Select scrolling and keyboard lock were the model for this one.

MIT licensed, the same as upstream. See [`LICENSE.txt`](LICENSE.txt).

The changes in this fork were written with the help of Claude (Anthropic).
