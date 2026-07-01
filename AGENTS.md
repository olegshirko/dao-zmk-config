# AGENTS.md

Information for AI coding agents working on this repository.

## Project overview

This repository is a [ZMK Firmware](https://zmk.dev/) user configuration for the **Dao** split mechanical keyboard. It does not contain application source code; instead, it provides keyboard-specific configuration that ZMK's reusable GitHub Actions workflow compiles into firmware images.

- The `main` branch targets the **Dao42** keyboard.
- The `dao44` branch targets the **Dao44** keyboard.
- The physical keyboard definition (PCB, matrix, layout aliases such as `dao_crkbd_layout`) is provided by the external module [`ergonautkb-zmk-module`](https://github.com/ergonautkb/ergonautkb-zmk-module).

## Repository layout

```
.
├── .github/workflows/build.yml   # CI workflow entry point
├── build.yaml                    # ZMK build targets and CMake options
├── README.md                     # User-facing flashing and keymap guide
└── config/
    ├── dao.conf                  # Kconfig overrides for the keyboard
    ├── dao.keymap                # Keymap: layers, bindings, and behaviors
    ├── info.json                 # Physical layout metadata for keymap editors
    └── west.yml                  # West manifest pinning ZMK and dependencies
```

## Technology stack

- **Firmware framework:** ZMK (runs on Zephyr RTOS).
- **Build system:** CMake + West, orchestrated by ZMK's reusable GitHub Actions workflow.
- **Target MCU:** Boards named `dao_left` and `dao_right` (defined in `ergonautkb-zmk-module`).
- **Programming language:** Devicetree (`.dts`/`.dtsi`) and Kconfig for configuration.
- **Firmware output:** `.uf2` files, one per keyboard half, plus Studio-enabled variants.

## Build process

Builds are performed remotely by GitHub Actions; there is no local build script in this repo.

1. `.github/workflows/build.yml` invokes `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`.
2. The reusable workflow reads `build.yaml` to determine targets.
3. `west.yml` pulls in the `zmk` repository and `ergonautkb-zmk-module`.
4. The resulting artifacts are packaged into `firmware.zip`.

### Build targets (`build.yaml`)

| Board/shield | Snippet | Extra CMake args | Artifact name |
|---|---|---|---|
| `dao_left` | — | — | `dao_left` |
| `dao_right` | — | — | `dao_right` |
| `dao_left` | `studio-rpc-usb-uart` | `-DCONFIG_ZMK_STUDIO=y` | `dao_left_studio` |
| `dao_right` | `studio-rpc-usb-uart` | `-DCONFIG_ZMK_STUDIO=y` | `dao_right_studio` |
| `nice_nano_v2` + `settings_reset` | — | — | `settings_reset` |

The `settings_reset` target produces a UF2 that clears Bluetooth pairings on a `nice_nano_v2`.

## Code organization and main files

- `config/dao.keymap`
  - Defines four layers (`DEF`, `LWR`, `RSE`, `ADJ`) with display names `MAIN`, `MAC`, `DEV`, `ADJ`.
  - `MAIN`: QWERTY with positional home-row mods (`A`=Ctrl, `S`=Alt, `F`=Shift, `J`=Shift, `L`=Alt, `;`=Ctrl) and `Command` on the right inner thumb (`Esc` hold). The outer top keys are plain `]` and `[`. A combo `, + .` switches the macOS input source (`Ctrl+Space`).
  - `MAC`: symbols/numbers on the left, one-handed macOS shortcuts on the right (Cut/Copy/Paste/Undo/Redo/Select All, Spotlight, App/Window switcher, Back/Forward, Find Action).
  - `DEV`: terminal and JetBrains GoLand shortcuts on the left, arrow navigation and media keys on the right.
  - `ADJ`: left half — Bluetooth profiles under left index/middle fingers (`E`/`D`/`C` and `R`/`F` = `BT 0`–`BT 4`), `BT CLR` on the outer pinky, plus bootloader, studio unlock, system reset; right half — centered digit layer (0-9).
  - Uses custom positional hold-tap behaviors `lmt` (left_mod_tap) and `rmt` (right_mod_tap) for the home-row modifiers, plus `&mt`, `&lt`, `&kp`, `&trans`, `&bt`, `&bootloader`, `&sys_reset`, `&studio_unlock`.
  - Selects the physical layout via the `zmk,physical_layout` chosen node (`dao_crkbd_layout` by default; `dao_full_layout` is commented out).
  - Includes `behaviors.dtsi`, `dt-bindings/zmk/bt.h`, and `dt-bindings/zmk/keys.h`.

- `config/dao.conf`
  - Sets `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` to increase Bluetooth transmit power.
  - Contains a commented-out `CONFIG_ZMK_USB_LOGGING=y` debug option.

- `config/info.json`
  - Vial/QMK-style layout descriptor used by visual keymap editors.
  - Describes 42 key positions arranged in four rows with split thumb clusters.

- `config/west.yml`
  - Pins the `zmkfirmware/zmk` repo at the `v0.3.0` release for build stability.
  - Pins `ergonautkb/ergonautkb-zmk-module` at the `main` branch.

- `.github/workflows/build.yml`
  - Uses the reusable `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3.0` workflow.
  - Pinned together with `west.yml` to avoid breaking changes from ZMK `main` (e.g., Zephyr 4.1 board-revision changes).

## Development conventions

- **Edit the keymap in `config/dao.keymap` only.** Do not modify `info.json` by hand unless you are intentionally updating the physical layout metadata.
- **Layer constants:** Use the existing defines (`DEF`, `LWR`, `RSE`, `ADJ`) when adding or modifying layers.
- **Home-row mods:** Use `&lmt` for left-hand home-row modifiers and `&rmt` for right-hand ones. Do not put `Command` on the home-row letter keys (`D`/`K`); `Command` lives on the outer top keys (`]` and `[`).
- **macOS shortcuts:** Prefer sending `LG(...)` (Left GUI / Command) combinations. Common clipboard/navigation shortcuts are centralized on the `MAC` layer; GoLand/terminal shortcuts live on the `DEV` layer.
- **Behavior tuning:** `quick_tap_ms = <200>` is used for `&lt`, `&mt`, `&lmt`, and `&rmt`. Home-row mods use `flavor = "tap-preferred"` and `require-prior-idle-ms = <125>` to avoid accidental modifiers during fast typing.
- **Physical layout:** If you switch the chosen `zmk,physical_layout`, leave the unused option commented rather than deleting it, to make the alternative explicit.
- **No source code formatting tools** are configured; follow the existing indentation and alignment in `dao.keymap`.

## Keymap position reference

Key positions are numbered sequentially by the order in which they appear in the `keymap` node, starting at `0`:

```
Row 0:  0  1  2  3  4  5      6  7  8  9 10 11
Row 1: 12 13 14 15 16 17     18 19 20 21 22 23
Row 2: 24 25 26 27 28 29     30 31 32 33 34 35
Thumbs:        36 37 38     39 40 41
```

Left-hand positions: `0 1 2 3 4 5 12 13 14 15 16 17 24 25 26 27 28 29 36 37 38`.  
Right-hand positions: `6 7 8 9 10 11 18 19 20 21 22 23 30 31 32 33 34 35 39 40 41`.

These lists are used by the `lmt` and `rmt` positional hold-tap behaviors so that home-row modifiers activate reliably for cross-hand combinations but do not fire during same-hand rolls.

## Testing instructions

There are no automated tests in this repository. Verification is done by building the firmware:

1. Push a commit or open a pull request.
2. Wait for the `build` workflow in the `Actions` tab to complete.
3. Download the `firmware.zip` artifact and confirm it contains the expected `.uf2` files.

For local-style validation, you can fork the repo and let GitHub Actions build it; ZMK local builds require a full Zephyr toolchain setup outside the scope of this repo.

## Deployment process

1. Download `firmware.zip` from the completed GitHub Actions run.
2. Extract `dao_left.uf2` and `dao_right.uf2` (or the `_studio` variants if using ZMK Studio).
3. For each half:
   - Slide the power switch to `OFF`.
   - Connect the half via USB-C.
   - Press the `RESET` button **twice** to enter DFU/UF2 bootloader mode.
   - Copy the corresponding `.uf2` file to the root of the mounted USB drive.
   - Disconnect the half.
4. Pair the halves by turning both on and pressing `RESET` once on each half simultaneously.

A "File Transfer Error" after copying the UF2 is expected and harmless; see the [ZMK troubleshooting docs](https://zmk.dev/docs/troubleshooting#file-transfer-error).

## Security considerations

- This repository contains only keyboard configuration; no secrets, credentials, or personal data are stored here.
- Do not add `.env` files, private keys, or API tokens.
- The firmware runs on a USB/BLE microcontroller. When editing `dao.conf`, only enable `CONFIG_ZMK_USB_LOGGING` for temporary debugging; leaving it on exposes key events over USB.
- Dependencies are fetched from public GitHub repositories (`zmkfirmware/zmk`, `ergonautkb/ergonautkb-zmk-module`) pinned to the `main` branch via West; be aware that upstream changes can affect builds.
