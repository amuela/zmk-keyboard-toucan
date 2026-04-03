# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ZMK firmware configuration for the **beekeeb Toucan** — a wireless split 42-key keyboard based on the Seeeduino XIAO BLE (nRF52840). Features a Cirque Pinnacle trackpad, nice!view display, and BLE split connection.

## Build

Firmware is built via GitHub Actions using the official ZMK workflow (`zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`). There is no local build setup in this repo — builds are triggered by push/PR/manual dispatch.

The build matrix in `build.yaml` produces three artifacts:
- Left half: `toucan_left` + `rgbled_adapter` + `nice_view_gem` (+ studio-rpc-usb-uart)
- Right half: `toucan_right` + `rgbled_adapter`
- Settings reset image

For local builds (requires a west workspace with ZMK initialized):
```bash
west build -b seeeduino_xiao_ble -- -DSHIELD="toucan_left nice_view_gem rgbled_adapter"
```

No test or lint tooling exists in this repo.

## Architecture

### Dependency manifest
`config/west.yml` pulls in ZMK v0.3, the Cirque touchpad input module (toucan branch from geeksville/), and zmk-rgbled-widget v0.3.

### Shield definitions (`boards/shields/toucan/`)
- `toucan.dtsi` — shared hardware: 4x12 GPIO key matrix, matrix transform mapping 42 keys, physical layout with per-key rotation/stagger, SPI touchpad split input, and scroll processors
- `toucan_left.overlay` / `toucan_right.overlay` — per-side GPIO column mappings, SPI bus config. Right side owns the Cirque Pinnacle touchpad (1 MHz SPI, 2x sensitivity, X-inverted, DR on P0.02)
- `toucan_left.conf` / `toucan_right.conf` — Kconfig for BLE split roles, power management, pointing device. Left is central; right is peripheral
- `Kconfig.shield` / `Kconfig.defconfig` — shield declarations and defaults (SPI, split BLE, pointing, battery proxy)

### Display (`boards/shields/nice_view_gem/`)
Custom nice!view display shield with C widget code compiled via CMake. Widgets in `widgets/` render battery (central + peripheral), layer name, BLE profile, output type, and a sleep indicator. Font assets live in `assets/`.

### Keymap (`config/toucan.keymap`)
4 active layers: BASE (QWERTY), NAV (navigation + BT), SYM (symbols), ADJ (F-keys + media + Emacs workarounds). Tri-layer combo: NAV + SYM activates ADJ. ADJ provides LALT/LCTRL on outer pinky bottom row for modified F-key combos, plus macros for `M-?` (G position) and `M-S-RIGHT` (; position) — Emacs chords impossible as direct keypresses due to pinky-column collisions. Layout JSON for the keymap editor is in `config/toucan.json`.

### Power management
- Deep sleep after 60 min idle (`CONFIG_ZMK_SLEEP`)
- Soft-off support (`CONFIG_ZMK_PM_SOFT_OFF`)
- Touchpad activity timeout: 30 sec (drops from 2.9 mA to 1.7 mA)

### Split roles
Left = central (battery fetching/proxy, display). Right = peripheral (touchpad, battery reporting to central).
