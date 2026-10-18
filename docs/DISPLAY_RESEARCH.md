# Corne-ish Zen Display Replacement — Research Notes

Oct 17, 2026 · @Yaoyuan

## Decision

Replace both failed panels with the **Waveshare (微雪) 1.02" e-Paper**, FPC marking **WFT0102CZA2**. It is the same panel as the original Good Display GDEW0102T4, so it plugs into the existing connector, fits the case window, and needs no firmware change.

Faster displays (memory LCD, OLED) were evaluated and are workable, but each needs an adapter between connectors, a mounting change, or a new driver. Those options are parked, not rejected.

## Diagnosis

The screen is a 1.02" IL0323 e-paper panel, not an OLED. The hardware rows below come from the devicetree overlays in `zmk-config-zen-2`, which this keyboard has been running. They were checked against the controller's silkscreen and tested on the keyboard (keys, split, Bluetooth, RGB, battery %) on Oct 17, 2026. The repo's artwork says "Corne-ish Zen R3 by lowprokb", but unlike a stock Zen it uses a plug-in nice!nano-compatible controller, so its pinout differs from ZMK's Zen v2.

| Item | Finding |
| --- | --- |
| Controller | `gooddisplay,il0323` (UC8175 family), SPI |
| Resolution | 80 × 128, portrait |
| Hardware | nice!nano-compatible "52840NANO" controller, normal Corne key matrix, 12 × WS2812 underglow, battery on nRF52840 VDDH |
| E-paper pins (both halves) | SCK P0.20, MOSI P0.17, CS P0.10, DC P0.09, BUSY P1.06, RST P1.04 |
| Firmware | `zmk-config-zen-2`, own `corne_epd` shield on ZMK's nice\_nano\_v2 board (DC/DC REG1 on), pinned to ZMK v0.3 |
| Right panel | Black blobs, moisture ingress or delamination; permanent |
| Left panel | Grey haze and ghosting; degrading |

Both panels need replacing. Neither fault is a firmware or connector problem.

## Replacement panel

The original [Good Display GDEW0102T4](https://www.good-display.com/product/207.html) is end-of-life, and no successor "GDEY0102T4" was found. Waveshare's 1.02" panel is the same glass. Its ribbon carries the same part marking as the panel removed from the Zen.

| Attribute | Original (GDEW0102T4) | Waveshare 1.02" e-Paper |
| --- | --- | --- |
| FPC marking | …102CZA2 (seen on removed panel) | WFT0102CZA2 |
| Resolution | 128 × 80 | 128 × 80 |
| Active area | 21.76 × 14 mm | 21.76 × 14 mm |
| Pixel pitch | 0.175 × 0.17 mm | 0.175 × 0.17 mm |
| Refresh (partial / full) | 0.3 s / 2 s | 0.3 s / 2 s |
| Colours | Black / white | Black / white |

Buying options:

- **Module (SKU 17575):** the panel on a blue 42 × 27 mm board with an 8-pin header. Remove the panel from the board's FPC socket. The board can serve as a bench test jig. In stock at [GroBotronics](https://grobotronics.com/e-paper-display-module-1.02-128x80-black-white.html?sl=en).
- **Raw panel (SKU 17574):** cheaper, but [marked discontinued by Waveshare](https://waveshare.com/1.02inch-e-paper.htm). Reseller stock may remain.
- **Taiwan:** search Taobao or 露天 for "微雪 1.02寸 墨水屏".

Avoid the [GDEW0102I4FC](https://robu.in/wp-content/uploads/2023/12/1.02-Flexible-Monochrome-ePaper-Display-with-128x80-Pixels.pdf). It is the same resolution, but flexible, with a 30-pin tail that must be ACF-bonded.

## Installation

- [ ] Order two panels, one per half, plus a spare if budget allows
- [ ] Optional: confirm each panel refreshes on the Waveshare board before fitting it
- [ ] Release the panel from the module's FPC latch and slide the ribbon out
- [ ] Note the old panel's ribbon orientation (contacts up or down) before removing it
- [ ] Insert the new ribbon the same way and close the latch
- [ ] Flash firmware built from zmk-config-zen-2 and check the status screen on both halves

Cautions: the glass cracks most easily where the ribbon meets it, so avoid sharp bends there. If faint ghosting appears, tune `pwr`, `cdi` and `tcon` in the `il0323` node. Part of the perceived lag is the 2 s full refresh versus the 0.3 s partial refresh. Check how often the firmware does a full refresh before blaming the panel.

## Battery

A 3.7 V 1500 mAh LiPo is electrically compatible; physical size is the limit.

- The battery feeds the nRF52840's VDDH pin and is measured there (`zmk,battery-nrf-vddh`). There is no fuel gauge and no capacity setting to change.
- A larger cell only charges longer; the on-board charge current is fixed.
- A typical 1500 mAh pouch (e.g. 103450, about 10 × 34 × 50 mm) is unlikely to fit under the PCB. Measure the stock cell and stay within its size.
- Use a cell with a protection circuit, and check connector type and polarity before plugging in.

## Alternatives considered

Faster displays are electrically possible because the Zen's display port is a plain SPI bus with spare control lines. Every option shares two costs: an adapter from the 24-pin e-paper socket, and a new mounting behind a window cut for a 1.02" portrait panel.

| Option | Speed | Power when on | Fits window | Driver in ZMK | Extra work |
| --- | --- | --- | --- | --- | --- |
| Waveshare 1.02" e-paper (chosen) | Slow (0.3 / 2 s) | \~0 between updates | Yes | Yes (`il0323`) | None |
| JDI LPM009M360A memory-in-pixel (MIP) LCD, 0.84", 72 × 144 | Instant | Microwatts | Yes (12.5 × 21 mm outline) | No, board-local driver written | 24→8-pin adapter, mounting |
| nice!view (Sharp LS011B7DH03), 160 × 68 | Instant | Tens of µA | No, landscape and too wide | Yes (`sharp,ls0xx`) | Adapter, new cover, widget re-layout |
| SPI OLED (SSD1306) | Instant | Several mA to \~20 mA | Depends on module | Yes | Adapter, re-layout, shorter battery life |
| Colour TFT (ST7735/ST7789) | Instant | \~10–30 mA plus backlight | Depends on module | Yes | Not sensible for a wireless split |

The LPM009M360A was the strongest upgrade path: fast, near-zero power, portrait, and small enough for the existing window.

## Parked: LPM009M360A driver

A Zephyr driver for the LPM009M360A was written and compile-tested on ZMK v0.2.1, but it is not part of the firmware and has not run on real hardware. Its files sit in `corniesh_zen/_to_delete/` (`drivers/`, `dts/`, `lpm009m360a_*.dtsi`). Copy them out before deleting that folder if this option may come back.

Its devicetree snippets used the stock Zen's pins. On corne\_epd the panel would map onto these nets, the same on both halves:

| Panel pin | corne\_epd net |
| --- | --- |
| SCLK | SCK P0.20 |
| SI | MOSI P0.17 |
| SCS (active-high) | CS P0.10 |
| DISP | old RST P1.04 |
| EXTCOMIN | old DC P0.09 |

Open items if this is revived:

- [ ] Move the driver into zmk-config-zen-2 as part of the `corne_epd` shield and rewrite the snippet with the pins above
- [ ] Get the LPM009M360A datasheet: 8-pin pinout, FPC pitch, EXTMODE, VDD maximum vs 3.3 V
- [ ] Map the 24-pin e-paper socket by continuity; leave charge-pump and high-voltage pins unconnected
- [ ] Bench-test the panel with fly-wires before designing an adapter flex
- [ ] Design the 24→8-pin adapter and a centring bracket

Protocol source: the [memory-lcd-spi](https://github.com/andelf/memory-lcd-spi) Rust driver, tested on this panel. Panel specs: [YouriTech listing](https://www.youritech.com/products/lpm009m360a-jdi-0-84-inch-mip-display-72x144-reflective-sunlight-readable-tft-lcd-display.html).

## Sources

- [Good Display GDEW0102T4 (EOL)](https://www.good-display.com/product/207.html)
- [Good Display 1.02" launch note](https://www.good-display.com/news/Good-Display-launched-1.02-inch-glass-substrate-e-paper!-95.html)
- [Waveshare 1.02inch e-Paper](https://waveshare.com/1.02inch-e-paper.htm)
- [GroBotronics: Waveshare 1.02" module](https://grobotronics.com/e-paper-display-module-1.02-128x80-black-white.html?sl=en)
- [Eckstein: GDEW0102T4](https://eckstein-shop.de/Neu_2__102_1)
- [GDEW0102I4FC spec (robu.in)](https://robu.in/wp-content/uploads/2023/12/1.02-Flexible-Monochrome-ePaper-Display-with-128x80-Pixels.pdf)
- [YouriTech: JDI LPM009M360A](https://www.youritech.com/products/lpm009m360a-jdi-0-84-inch-mip-display-72x144-reflective-sunlight-readable-tft-lcd-display.html)
- [andelf/memory-lcd-spi](https://github.com/andelf/memory-lcd-spi)
