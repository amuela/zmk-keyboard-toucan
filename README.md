# ZMK config for beekeeb Toucan Keyboard

[The beekeeb Toucan Keyboard](https://beekeeb.com/toucan-keyboard/) is a wireless split 42-key column-stagger keyboard with a display and a trackpad, with an aggressive stagger on the pinky columns.

This configuration is optimized for **Emacs + i3wm + modern C++** on a US International (dead keys) layout.

## Layout

### Finger assignment

Standard touch typing with index fingers on F/J:

```
Left hand                              Right hand
x=0    x=1   x=2    x=3    x=4    x=5    x=8    x=9    x=10   x=11   x=12   x=13
o-pnk  pnk   ring   mid    INDEX  reach  reach  INDEX  mid    ring   pnk    o-pnk
```

### BASE

```
TAB        Q  W  E  R  T        Y  U  I  O  P       BSPC
ESC|META   A  S  D  F  G        H  J  K  L  td(;:)  '
LSHFT      Z  X  C  V  B        N  M  ,  .  /       RSHFT
           LGUI   NAV|TAB  LCTRL       SPC  SYM|RET  DEL|RALT
```

### NAV (hold left middle thumb)

```
___    1     2     3     4     5        6     7     8     9     0    DEL
LGUI  HOME  PGUP  PGDN  END  RCLK    MCLK  LEFT  DOWN   UP  RIGHT  ___
___   STUDIO BT0   BT1   BT2   BT3     BT4 BTCLR SCR_D SCR_U ___   ___
           ___   [NAV]   ___            ___  >>>   ___
```

### SYM (hold right middle thumb)

```
 ~    @    #    *    &    !        ?    _    +    $    %    `
 |    [    <    {    (    -        =    )    }    >    ]    \
___   //   &&   ->   ::   <<       >>   ==   <=   !=   >=  ___
           ___  >>>  ___              ___  [SYM]  ___
```

### ADJ (hold both middle thumbs — tri-layer)

```
 F1   F2   F3   F4   F5   F6      F7   F8   F9   F10  F11  F12
___  ___  ___  ___   ¡   ___      ___   ¿  VOL_D VOL_U ___   ^
___  ___  ___  ___  ___  ___      ___ PREV PLAY NEXT MUTE  ___
           ___  [NAV] ___            ___ [SYM] ___
```

## Design decisions

### Thumb cluster

| Position | Key | Rationale |
|----------|-----|-----------|
| L outer | LGUI | Dedicated Super for i3 `$mod` bindings — must be holdable without timing |
| L middle | NAV/TAB | tap = Tab (Emacs completion), hold = NAV layer |
| L inner | LCTRL | Dedicated Ctrl — zero latency for rapid Emacs sequences (C-x C-s, C-x C-f) |
| R inner | SPC | Dedicated Space |
| R middle | SYM/RET | tap = Enter, hold = SYM layer — both Space and Enter on right thumb |
| R outer | DEL/RALT | tap = Delete, hold = Right Alt (AltGr for Spanish: ñ, á, é, ó, ú, ¿, ¡) |

### Left pinky home row: ESC|META (mod-tap)

- tap = Escape (Emacs C-g alternative, mode exits)
- hold = Left Alt / Meta (M-x, M-f, M-b, M-d, M-w)
- Ctrl (more frequent in Emacs) gets the dedicated thumb position; Meta (less frequent) tolerates the mod-tap timing

### SYM layer: C++ optimized

**Home row — brackets by C++ frequency, same finger pairing:**

| Finger | Left (opening) | Right (closing) | C++ usage |
|--------|---------------|-----------------|-----------|
| Index home | `(` | `)` | Function calls, casts, conditions, decltype |
| Middle | `{` | `}` | Blocks, lambdas, initializer lists |
| Ring | `<` | `>` | Templates, `#include`, comparisons |
| Pinky | `[` | `]` | Lambda captures, subscripts, attributes |

**Top row — operators prioritized by C++ frequency on index fingers:**

- Left index: `&` (references: `const T&`, `T&&`)
- Right index: `_` (snake_case identifiers)
- Left middle: `*` (pointers, dereference)
- Right middle: `+` (addition, `operator+`)

**Bottom row — C++ multi-character combos as single keypresses (macros):**

| Left | | | | | | Right | | | | |
|------|------|------|------|------|------|------|------|------|------|------|
| `//` | `&&` | `->` | `::` | `<<` | | `>>` | `==` | `<=` | `!=` | `>=` |
| pnk | rng | mid | INDEX | reach | | reach | INDEX | mid | rng | pnk |

`::` (scope resolution) and `==` (equality) are on index home — the two most frequently typed combos in C++.

### NAV layer: i3 arrow mapping

Arrows match i3 default `$mod+j/k/l/;` directions, so the same physical key moves the same direction whether using i3 or the NAV layer:

| Key position | i3 action | NAV arrow |
|-------------|-----------|-----------|
| J (R index home) | `$mod+j` = focus left | LEFT |
| K (R middle) | `$mod+k` = focus down | DOWN |
| L (R ring) | `$mod+l` = focus up | UP |
| ; (R pinky) | `$mod+;` = focus right | RIGHT |

i3 workspace switching: hold LGUI (L outer thumb) + hold NAV (L middle thumb), then type 1-9 with either hand.

### Mouse controls (NAV layer)

The Toucan's Cirque Pinnacle touchpad automatically converts to scroll in NAV/SYM layers (configured in the device tree). Additionally:

- RCLK on left index reach (G position) — right-click with left hand while right hand uses touchpad
- MCLK on right index reach (H position) — middle click
- SCRL_DOWN / SCRL_UP below the DOWN/UP arrow keys — discrete scroll steps

### ADJ layer: Spanish characters

`¡` and `¿` are implemented as macros that send AltGr + key internally, so they work from the ADJ layer without needing to physically hold RALT. `^` (XOR) is also here, displaced from SYM by the more useful backtick.

### Tap-hold behavior

All mod-taps and layer-taps use **balanced** flavor (200ms tapping term). This means the hold triggers when another key is pressed AND released during the hold, preventing accidental modifier activation during fast typing.

Layer-taps have **quick-tap** (150ms) so double-tapping Tab or Enter sends the key twice without activating the layer.

### Tap-dance

Semicolon position (`;`): single tap = `;`, double tap = `:` (250ms window). Saves reaching for Shift when typing colons in C++ (ternary, range-for, labels).

## Building

Firmware builds via GitHub Actions on push/PR. Download UF2 artifacts from the Actions tab. Flash each half by double-tapping the XIAO BLE reset button and copying the UF2 to the mounted drive.

For local builds (requires a west workspace with ZMK):

```bash
west build -b seeeduino_xiao_ble -- -DSHIELD="toucan_left nice_view_gem rgbled_adapter"
west build -b seeeduino_xiao_ble -- -DSHIELD="toucan_right rgbled_adapter"
```

# License

The code in this repo is available under the MIT license.

The included shield nice_view_gem is modified from https://github.com/M165437/nice-view-gem licensed under the MIT License.

ZMK code snippets are taken from the ZMK documentation under the MIT license.

The embedded font QuinqueFive is designed by GGBotNet, licensed under under the SIL Open Font License, Version 1.1.
