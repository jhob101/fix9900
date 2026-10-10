# Development notes

> **These are thoughtfix's notes from the original
> [fix9900](https://github.com/thoughtfix/fix9900) work, kept for the reasoning
> behind those fixes.** This fork has since changed some of what they describe:
>
> - Trackpad scroll mode is now hold-Select-to-scroll, not click-to-toggle, and
>   the trackpad press is a left click again. A click-to-toggle scroll switch
>   can be put on a key (see the README).
> - A tap on B is a mouse click, not F23.
> - The volume key behavior was reworked to cope with one-shot Shift.
> - Bluetooth is switched off.
> - The bootloader shortcuts changed. Fn+\\ and Ctrl+Alt+\\ are gone (the
>   single mod-morph described below fired on Ctrl or Alt alone). It is now
>   Left Alt + Right Alt + Start, or both Fn keys with \\.
> - The build instructions near the end are superseded by the
>   [README](../README.md#building).
>
> For what the firmware does today, read the [README](../README.md).

Notes from auditing and fixing `Bill-lulu/uc9900` (starting from `V1.2.1`) for the
`hack2you` uConsole keyboard/trackpad mod. Organized into three kinds of entry:

- **Implementation notes**: deliberate design decisions, not bug fixes.
- **Bugfixes**: things that were genuinely broken, with root cause and fix.
- **Investigations, warnings, and how-tos**: discoveries worth knowing about,
  a couple of false starts worth not repeating, and setup/testing instructions.

This firmware is built on ZMK rather than the original ClusterM uConsole's QMK
base. That difference explains a couple of design choices below, and comes up
more than once as the reason a fix here took more code than the equivalent QMK
change would.

## 1. Implementation notes

### CapsLock and the trackpad are now fully independent

Two unrelated things used to both hang off CapsLock state, both accidental:

- The trackpad backlight/power pin was wired through ZMK's built-in CapsLock
  LED indicator, leftover from the base board template, which had an actual
  CapsLock LED on that pin. Every CapsLock toggle silently drove the trackpad
  backlight as a side effect.
- Pressing CapsLock *or* ScrollLock silently switched the trackpad from
  cursor movement into scroll-wheel mode, via a check on the raw USB
  HID indicator bitmask the host reports back (`zmk_hid_indicators_get_current_profile()`).
  This had nothing to do with the LED bug above, just the same pattern
  (`{2, 3, 7}`/CapsLock-related bitmask values) copy-pasted into a second
  place, with ScrollLock's own separate trigger value (`4`) added in.

Both are now removed. CapsLock (`Fn+Tab`) toggles the keyboard modifier only,
with no LED and no effect on the trackpad. Trackpad scroll mode is now a
deliberate **click-to-toggle**: pressing straight down on the trackpad (its
center-click) flips between cursor mode and scroll mode. That position's
previous left/right-click bindings are removed, since the dedicated L/R
buttons already cover clicking. This was chosen over a hold-a-modifier
trigger (tried first) because the trackpad is operated by one hand (thumb),
so holding a modifier with the other hand for the whole gesture was awkward,
and polling-based hold-detection was also unreliable (see the Investigations
section).

### A second, safer way into the bootloader

`Fn+\` is the original way to drop into the UF2 bootloader, but it's a real
hazard: a stray finger near `\` while Fn is held for something else is
enough to trigger it. Added **Ctrl+Alt+\\** as a second, additional trigger,
implemented as a per-key modifier check (`zmk,behavior-mod-morph`) bound
directly to the Backslash position: normal `\` behaves exactly as before,
Ctrl+Alt+`\` sends `&bootloader` instead.

**TODO:** the old bare `Fn+\` is left in place on purpose. Check with
Bill-lulu before removing it, since other `uc9900` users may already rely on
it. Worth revisiting once this fix has had some real-world mileage.

## 2. Bugfixes

### Volume/Mute keys sent the wrong function entirely

**Was:** Speaker key alone sent Mute (should be Volume Down). Shift+Speaker
had no shift-handling at all, so it just sent Shift+Mute (should be Volume
Up). Both Fn layers sent Volume Up/Down (should be Mute on both).

**Fix:** base layer now uses a new custom behavior, `vol_shift`
(`app/src/behaviors/behavior_vol_shift.c` in the ZMK fork), that sends Volume
Down normally and Volume Up when Shift is held, explicitly releasing the
Shift modifier bit for the duration of the press and restoring it after.
That release-and-restore step matters: without it, Shift keeps reporting as
held throughout (Consumer-page HID reports have no modifier byte for a
simpler `zmk,behavior-mod-morph` to suppress it on), which is enough to stop
some window managers' keybind matching from firing at all. Confirmed on
this device via `libinput debug-events`, where the correct `KEY_VOLUMEUP`
event fired but nothing happened on-screen until this was fixed. See also
the Wayland/PipeWire note below: a *second*, separate issue also had to be
fixed before this was even testable.

Both Fn layers now send Mute.

### Right Alt / Right Ctrl went dead under either Fn layer

**Was:** bound to `&mo SYM`/`&none` on the bottom row of both Fn layers,
copy-paste residue, not intentional. Holding either Fn and pressing Right
Alt or Right Ctrl sent no HID event at all; any chord like Fn+RightCtrl+C
silently dropped the Ctrl.

**Fix:** both positions now bound to `&kp RALT`/`&kp RCTRL` on both layers,
matching how Left Ctrl and Right Shift already passed through correctly.

### Pause/Start sent garbage mouse-button clicks

**Was:** bound to `&mkp C_PAUSE`. `&mkp` expects a mouse-button bitmask, not
a consumer HID code, so the numeric value of `C_PAUSE` got reinterpreted as
mouse buttons, firing two simultaneous phantom clicks (`BTN_LEFT` +
`BTN_EXTRA`) instead of a Pause keypress.

**Fix:** `&kp C_PAUSE` instead of `&mkp C_PAUSE`, on both Fn layers.

### Enter key was completely dead

**Was:** the matrix-transform's `default_transform.map` (in both
`bb9900.overlay` and the duplicate copy in `boards/bb9900/bb9900.dts`) had
Enter's borrowed matrix position wrong: `RC(6,7)` instead of the physically
correct `RC(6,10)`, which also shifted Right Alt/Right Ctrl/Fn-R's own
borrowed positions down by one column. Present since `V1.2.1`; never
mattered until something made testing Enter specifically necessary.

**Fix:** corrected the transform's tail to `...RC(4,10) RC(6,10)` /
`RC(6,7) RC(6,8) RC(6,9)`, matching what the `V2.0` branch already had.

**Worth knowing if you hit something similar:** this class of bug, a wrong
matrix-transform entry for a key whose transform cell comes from a different
row than where it's listed in the keymap text, compiles clean, every other
key keeps working, and the affected key just produces zero HID events. That
looks exactly like a dead solder joint, a disabled Kconfig option, or a bad
keycode. The only reliable way we found it was a byte-level diff against a
known-good reference binary (`CURRENT.UF2`) plus a local build with real
debug symbols to identify which compiled table the differing byte belonged
to. If a key with an unusual transform entry goes dead on a fresh build,
check the matrix-transform `map` property against real hardware first.

## 3. Investigations, warnings, and how-tos

### Flashing: a stuck UF2 mount looks like a bricked board, isn't

Watch the trackpad light right after dragging the `.uf2` file onto the
drive:
- **Worked:** light goes out and *stays* out.
- **Stuck:** light comes back after 5 to 10 seconds, and the host's kernel
  log (`journalctl`) shows USB enumeration errors. This is a stuck unmount of
  the UF2 mass-storage drive, not a bad flash or firmware regression. Reboot
  the host (not the keyboard) and re-flash the same file.

### Wayland/PipeWire: volume keys can look broken even with a correct firmware

Not a firmware bug, but it blocked testing the volume fix above until we
found it, and it's likely to affect any uConsole CM5 running the stock
desktop image, not just this keyboard mod. This board's audio output has no
ALSA hardware mixer at all (`amixer scontrols` returns nothing on any card);
volume is a pure PipeWire software gain. The desktop's default volume
keybindings (`~/.config/labwc/rc.xml`, and the OSD popup script it calls)
run `amixer sset Master ...`, targeting a control that doesn't exist on this
hardware, so every volume keypress silently no-ops, and the on-screen
volume popup shows with no meter (its percentage read comes from the same
broken `amixer` call). Fix is `wpctl set-volume`/`set-mute
@DEFAULT_SINK@` instead of `amixer sset Master`, in both the keybind actions
and whatever script renders the OSD.

### Scroll works everywhere except VS Code and Firefox, open, not chased further

After the click-to-toggle scroll fix above, scrolling is clean in terminal,
text editor, and file manager. VS Code and Firefox receive genuinely valid,
well-formed wheel events (confirmed via `libinput debug-events`, the
firmware's own `zmk_hid_mouse_scroll_update()` call path, and Firefox's own
`wheel` DOM event with real deltas) but never visibly scroll. Ruled out:
wrong input type, Firefox's `smoothScroll` setting, per-report magnitude,
and event rate. Deepest lead: a `wev` capture showed `axis_source: wheel`
being re-announced on every individual tick instead of once per scroll
gesture, which, per the Wayland pointer protocol, a strict client
(Firefox, Chromium/Electron) could reasonably treat as constant
scroll-interaction resets, netting near-zero visible movement despite valid
deltas. Likely `libinput`'s own event-coalescing reacting to report timing,
not something controllable from the HID descriptor directly. Not pursued
further (real fallbacks exist in both apps: scrollbars, Page Up/Down), but
worth another look if there's appetite for more build/flash/test cycles
against an undocumented target.

### A reference photo turned out to be fake, verify hardware ground truth carefully

An early finding ("Enter and Backspace keycaps are swapped") was based on
comparing the firmware against a stock photo pulled from an online listing,
which turned out not to be a photo of the real device. A genuine high-res
photo of the actual hardware confirmed the firmware was correct all along.
Worth remembering before trusting any reference image as hardware ground
truth: check it's actually a photo of the unit in question.

### ABXY buttons send F21 to F24, not gamepad-style input, expected, not a bug

The D-pad sends normal arrow keys, but the ABXY circle buttons send
`F21` to `F24` on the base layer. Almost certainly intentional: those are
otherwise-unused keycodes specifically so they won't collide with any real
shortcut, which is what RetroPie/EmulationStation expects you to bind to
controller input yourself.

### Testing methodology

Every physical key was tested individually across the base layer and both Fn
layers before any fix work started (see `results_base/lfn/rfn.csv` in the
audit tooling). Worth doing the same after any keymap change, not just
trusting a source-level read.

To check exactly what a keypress sends, independent of what any particular
application does with it:

```
libinput debug-events --show-keycodes \
  --device /dev/input/by-id/<your-board>-event-kbd \
  --device /dev/input/by-id/<your-board>-event-mouse
```

This reads raw kernel evdev input, sitting below the audio/window-manager
stack entirely. It's the fastest way to confirm a firmware change actually
changed what's being sent, before troubleshooting anything OS-side. For
scroll/pointer-specific issues, `wev` (Wayland's own protocol-level event
dumper) is a useful complementary layer, one step below individual
applications and one above `libinput`.

### Building locally

```
python3 -m venv ~/zmk-venv && source ~/zmk-venv/bin/activate
pip install west

git clone https://github.com/thoughtfix/fix9900.git config
west init -l config
west update
west zephyr-export
pip install -r zephyr/scripts/requirements.txt

# nRF52840-capable Zephyr SDK, e.g. 0.16.9 (0.15.2 also verified to build
# identically): https://github.com/zephyrproject-rtos/sdk-ng/releases

west build -s zmk/app -d build -b bb9900 -- \
  -DZMK_CONFIG=$(pwd)/config \
  -DZEPHYR_SDK_INSTALL_DIR=/path/to/zephyr-sdk-<version>
```

`ZEPHYR_SDK_INSTALL_DIR` must be passed as a CMake `-D` define after `--`,
not as an environment variable. An exported env var is silently ignored.
Flash by copying `build/zephyr/zmk.uf2` onto the board's bootloader-mode
mass-storage drive (see the flashing note above for how to tell a good flash
from a stuck one).
