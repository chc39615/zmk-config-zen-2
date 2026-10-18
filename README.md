# zmk-config-zen-2

ZMK firmware for **corne_epd**: a Corne 3×6 wireless split with a 1.02" e-paper
display (IL0323, 80×128) and WS2812 underglow (artwork: Corne-ish Zen R3 by lowprokb),
on a nice!nano-compatible controller ("52840NANO").

Built as a **shield** (`corne_epd`) on ZMK's **`nice_nano_v2`** board. The board
(maintained by ZMK) supplies the controller side: battery sensing on VDDH,
ext-power switch, DC/DC regulator, flash layout. This repo only describes the
keyboard PCB. Pins were checked against the controller's silkscreen and tested
on the keyboard (keys, split, Bluetooth, RGB, battery %).

## Layout of this repo

| Path | What it is |
| --- | --- |
| `config/boards/shields/corne_epd/` | Shield: key matrix, e-paper, RGB strip, status screen and icons |
| `config/corne_epd.keymap` | Keymap (Colemak + home-row mods; RGB controls on the FUNC layer) |
| `config/corne_epd.conf` | Settings (sleep, display, underglow, BLE power, DC/DC switch) |
| `config/west.yml` | Pins ZMK to **v0.3** |
| `build.yaml` | GitHub Actions build matrix |
| `docs/DISPLAY_RESEARCH.md` | Display replacement research notes |

## Build

Push to GitHub; the Actions workflow builds `nice_nano_v2` + `corne_epd_left`,
`corne_epd_right` and `settings_reset`. Flash the `.uf2` files by double-tapping
reset. On macOS, Finder may report error -36 after copying; the flash still works
(or use `cp -X file.uf2 /Volumes/NICENANO/`).

## Replacement display

Waveshare (微雪) 1.02" e-Paper, FPC marking WFT0102CZA2 (same glass as Good
Display GDEW0102T4, now EOL). Drop-in; no firmware change needed.
Full notes (diagnosis, buying options, battery, alternatives considered):
[docs/DISPLAY_RESEARCH.md](docs/DISPLAY_RESEARCH.md).
