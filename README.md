# ggbr minila on RMK v0.9

This branch ports the [ggbr ZMK configuration](https://github.com/Henvy-Mango/zmk-config) to [RMK v0.9](https://rmk.rs/docs/getting_started/introduction). The nRF52840 matrix uses the same 5 row pins, 14 column pins, diode direction, and 66 physical key positions as the ZMK board.

## Mapped behavior

- Four layers: base, Mac modifier swap, Fn, and macro/media.
- Grave/Escape, Caps Lock layer-tap, right Ctrl tap dance, right GUI tap dance, and two numeric macros.
- Shift + Shift + B enters the bootloader; Shift + Shift + Backspace reboots; Shift + Shift + U toggles preferred USB/BLE output.
- The macro layer has previous/next BLE profile keys. With the default three RMK profiles, `User4` is previous and `User3` is next. `User5` clears the current bond when assigned in Vial; it does not clear every profile.
- Caps Lock indicator, battery ADC divider, BLE transmit power, and 15-minute BLE idle sleep.
- The original external VCC control pin (`P1.09`, active low) is held inactive at startup so the unused RGB underglow rail remains off. This uses RMK's `[[output]]` configuration; the pin is not the `P0.13` used in the nice!nano example.
- The nRF52840 REG1 DC/DC converter is enabled to match ZMK's `BOARD_ENABLE_DCDC` setting. The separate high-voltage REG0 converter remains disabled; its required external LC circuit is not established by the ZMK configuration.

## Differences from ZMK

RMK's configuration-only v0.9 template does not drive this board's 66 WS2812 underglow LEDs, so the ZMK RGB toggle/effect positions on Fn are transparent. ZMK's keyboard soft-off and wake-up GPIO behavior is also not represented by a key in this configuration. The ZMK four-key `BT_CLR_ALL` combo has no equivalent that clears only all BLE bonds; it is intentionally omitted to avoid resetting the entire stored keymap. The custom ZMK `leds.c` listener is outside this migration; the RMK Caps Lock indicator uses its built-in `[light]` setting.

## Build and flash

Every push and manual workflow dispatch runs [Build RMK firmware](.github/workflows/build.yml), following the [official cloud compilation guide](https://rmk.rs/docs/user_guide/create_firmware/cloud_compilation.html). It calls RMK's `rmk-v0.9.0` reusable workflow with the `0.9` project template. Download the UF2 artifact from the successful run's **Artifacts** section.

Check the bootloader on the physical ggbr before flashing. The RMK nRF52840 template generates UF2 for the Adafruit nRF52 bootloader with application flash starting at `0x1000`. A successful cloud build confirms that the configuration compiles; it cannot confirm this board's bootloader or electrical behavior. If an earlier RMK keymap was saved through Vial, clear the stored layout so the new defaults can take effect.
