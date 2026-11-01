# zmk-config
This is the ZMK-Config repo for the Polarity works BT60 keyboard, with this you can customise your layout and keymap to suit your exact needs.
We have provided base keymaps for the following layouts on the soldered version: ANSI, ISO, Split backspace and right shift, Tsangan and all 1u bottom rows. These can be accessed by choosing the appropriate branch. The hotswap version has one fork which already includes support for split backspace however it is not bound to anything useful thus you will need to fork and customise to your taste

# Customisation
In order to change the keymap to meet your needs, you will need a GitHub account. Click the "fork" button in the top right to create your own version of the repo, then use the drop down menu to select the branch with your layout.
The keymap information is contained entirely within the file bt60.keymap. Download this file to your computer and open it in a text editor such as Atom or notepad++ then you can open and edit it. The full list of ZMK keycodes can be found [here](https://zmkfirmware.dev/docs/codes/keyboard-keypad/).
After editing the keymap file upload it to the original folder using the "Add file" dialog in GitHub, then click on "Actions" and wait for the build process to complete (The orange dot will go to either a red X or green tick). If your firmware doesn't compile you most likely made a mistake with the keys, double check the .keymap file and try again.

# Compilation and downloading
 If you have made any errors in the keymap the compilation will fail and there will be a red X in the actions dialog. Click on it and it will tell you where the problem is.
 If the build completes with no errors you will be able to find the firmware in the "Artifacts" section of the last action.
 Put the keyboard into bootloader (Either double tap the hardware reset button or use the behaviour, typically fn + enter) and drag and drop the .uf2 file onto the usb drive that shows up

 # More advanced
 If you want to exercise greater control over your BT60 and have a unique layout implemented you will need to change the matrix transform. The following diagram displays which keys connect to which row and column pins. Note that everything is zero indexed in the matrix transform
 	![alt text](rowcolmap.png)
  The matrix transform for this PCB lives in ZMK itself (`app/boards/polarityworks/bt60/bt60_nrf52840_zmk_2_0_0.overlay`); to override it, redefine it in `bt60.keymap`. more information on how to do the matrix transforms can be found in the official ZMK documentation [here](https://zmkfirmware.dev/docs/development/new-shield#optional-matrix-transform)
  We're always willing to accept pull requests if you've developed your own layouts :)

# This fork: what's configured and where

Firmware is built from **ZMK `main` (Zephyr 4.1)** using ZMK's own in-tree board
`bt60@2.0.0//zmk` (the BT60 CKP PCB). The old custom board files in
`config/boards/` were removed: ZMK now ships an identical board definition
(same matrix pins, LEDs, battery and ANSI matrix transform).

| File | Purpose |
|---|---|
| `build.yaml` | Build target (`bt60@2.0.0//zmk`), firmware artifact named `bt60.uf2` |
| `.github/workflows/build.yml` | Uses ZMK's standard `build-user-config` workflow |
| `config/west.yml` | ZMK version (tracks `main`) |
| `config/bt60.conf` | Kconfig: BLE, sleep/idle, lighting defaults, encoder driver off |
| `config/bt60.keymap` | Keymap, mod-tap tuning, encoders disabled |

## Layers

| Layer | How to reach it |
|---|---|
| 0 Default | — |
| 1 Raise | hold **Fn** |
| 2 Lower | hold **Fn**, then hold **Esc** |

The bottom row is in Mac order: **Ctrl, Option (Alt), Command (GUI)**.

## Lighting and output keys

New keys were only placed on positions that were previously empty (`&trans`)
on their layer, so no existing shortcut was replaced.

| Keys | Action |
|---|---|
| Fn + R | RGB underglow on/off |
| Fn + T | RGB effect (cycle) |
| Fn + Y | RGB hue + |
| Fn + U / Fn + O | RGB brightness + / − |
| Fn + J | Backlight on/off |
| Fn + K / Fn + L | Backlight − / + |
| Fn + Esc + U | Output → **U**SB |
| Fn + Esc + B | Output → **B**luetooth |
| Fn + Esc + ↓ (Menu key) | Toggle output USB ↔ Bluetooth |
| Fn + Esc + Tab/Caps/Shift/Ctrl | Select BT profile 0/1/2/3 (unchanged) |
| Fn + Esc + ← / → | Previous / next BT profile (unchanged) |
| Fn + Esc + R | Clear current BT profile (unchanged) |

Note: output selection only matters while USB is plugged in. Without USB the
keyboard always uses Bluetooth. By default ZMK prefers USB when connected.

## Mod-tap keys (right side of bottom rows)

`Right Shift / ↑`, `Right Alt / ←`, `Menu / ↓`, `Right Ctrl / →` are mod-taps
(set in `config/bt60.keymap`, `&mt { ... }`):

- **tap** → arrow key
- **hold > 200 ms** (`tapping-term-ms`) → modifier
- **tap, then press and hold again within 175 ms** (`quick-tap-ms`) → the
  arrow key auto-repeats instead of becoming a modifier.

On the Fn layer these positions are plain arrows, so **Fn + arrow** also
auto-repeats.

## Power saving (`config/bt60.conf`)

| Setting | Effect |
|---|---|
| `CONFIG_ZMK_IDLE_TIMEOUT=30000` | Keyboard is "idle" after 30 s without input |
| `CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y` | Underglow switches off while idle, back on at the next keypress |
| `CONFIG_ZMK_BACKLIGHT_AUTO_OFF_IDLE=y` | Same for the backlight |
| `CONFIG_ZMK_SLEEP=y`, `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000` | Deep sleep after 15 min; any key wakes it (that keypress is not sent), then BLE reconnects |
| `CONFIG_ZMK_RGB_UNDERGLOW_ON_START=n` | Underglow is no longer forced on at every boot (the last on/off state is restored) |
| `CONFIG_ZMK_BACKLIGHT_BRT_START/STEP=10` | Backlight starts at 10 %, 10 % per step |
| `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y` | BLE connection-stability tweaks (from upstream) |
| `CONFIG_ZMK_BLE_EXPERIMENTAL_SEC=y` | Lets macOS re-pair when stale pairing keys exist (fixes "Connect does nothing") |

None of these change key behaviour. Idle and sleep only react to time without
keypresses. Lighting toggled off by hand stays off after idle.

## Rotary encoders

No encoders are fitted. `encoder_1..3` and the keymap `sensors` node are
disabled at the top of `config/bt60.keymap`, and `CONFIG_EC11=n` in
`config/bt60.conf`. The build prints a harmless Kconfig warning about
`EC11_TRIGGER_GLOBAL_THREAD` (set by ZMK's board defaults, unused once the
driver is off). To use an encoder again, delete those lines and add
`sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN>;` to each layer.

## Building

Push to GitHub (or run the **Build** action manually) and download `bt60.uf2`
from the run's artifacts. Locally:

```bash
west init -l config && west update && west zephyr-export
west build -s zmk/app -b "bt60@2.0.0//zmk" -- -DZMK_CONFIG="$PWD/config"
```
