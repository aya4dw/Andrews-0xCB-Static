# Andrew's 0xCB Static Vial

Custom Vial firmware for the 0xCB Static macro pad with Bongocat OLED animation.

## Features

- **Vial support** — remap keys and layers on the fly via [Vial](https://get.vial.today/)
- **Bongocat OLED animation** — fullscreen RLE-compressed cat animation on the 128×32 OLED, sourced from [filterpaper/qmk_oled_animations](https://github.com/filterpaper/qmk_oled_animations)
- **Custom OLED font** — layer names and NUM/CAP indicators rendered with custom glyphs
- **Encoder support** — volume control with click mute

## Bongocat Animation

The OLED displays a fullscreen Bongocat animation that reacts to typing activity:

| State | Trigger | Frames |
|-------|---------|--------|
| Idle  | > 1.6s since last keypress | 5 idle frames (loop) |
| Paws  | 0.4–1.6s since last keypress | 1 static stretch frame |
| Tap   | < 0.4s since last keypress | 2 tap frames (alternate) |

- Uses `last_matrix_activity_time()` for input detection — no custom `process_record_user` needed
- `#define CAT_SHIFT 30` shifts the animation right 30 pixels so NUM/CAP indicators overlay cleanly on the left
- RLE decoder (`decode_cat_frame`) writes shifted per-page via `oled_write_raw_byte`

Source: [filterpaper/qmk_oled_animations](https://github.com/filterpaper/qmk_oled_animations) (MIT license)
Filterpaper's RLE Bongocat uses ~150 bytes per frame vs ~512 bytes for raw bitmaps, saving considerable flash on the ATmega328P.

## Firmware

- **MCU:** ATmega328P (32 KB flash, 2 KB RAM)
- **Firmware size:** 26936 / 28672 bytes (93%, 1736 free)
- **Bootloader:** USBasp (ICSP required — bootloader entry is broken on this board)

### Flashing via USBasp

```
make 0xcb/static:vial:flash
```

ICSP header (J2) pinout for USBasp 10-pin → 6-pin:
- Pin 1 (MOSI) → J2-4
- Pin 2 (VCC)  → J2-2
- Pin 4 (GND)  → J2-6
- Pin 5 (RST)  → J2-5
- Pin 7 (SCK)  → J2-3
- Pin 9 (MISO) → J2-1

## Repository Structure

```
Andrews-0xCB-Static/
├── config.h                     # Keyboard-level config (OLED font, etc.)
├── gfxfont.c                    # Custom OLED font (layer glyphs 0x91-0xFF)
├── keyboard.json                # Matrix + layout definition
├── keymaps/vial/
│   ├── config.h                 # Vial keymap config (UID, unlock combo)
│   ├── keymap.c                 # Keymap + Bongocat OLED code
│   ├── rules.mk                 # Build rules (VIAL, ENCODER_MAP, OLED)
│   └── vial.json                # Vial keyboard definition
├── Andrew's_0xCB_Static.vil    # Vial layout export (4 layers)
├── 0xcb_static_vial.hex        # Pre-compiled firmware
└── readme.md
```

## Vial Layout

The included `.vil` file can be loaded in Vial (File → Load Saved Layout) to restore the 4-layer keymap:

- **Layer 0 (_HOME):** QWERTY alphas, MUTE on encoder press
- **Layer 1 (_FN2):** Number row + F-keys
- **Layer 2 (_FN3):** Numpad layer
- **Layer 3 (_FN4):** Media keys + navigation

## Credits

- [0xCB-dev/0xcb-static](https://github.com/0xCB-dev/0xcb-static) — original hardware and default firmware
- [filterpaper/qmk_oled_animations](https://github.com/filterpaper/qmk_oled_animations) — RLE-encoded Bongocat animation
- [Vial](https://get.vial.today/) — open-source keyboard configurator
