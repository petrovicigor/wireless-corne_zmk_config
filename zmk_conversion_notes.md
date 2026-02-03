# QMK → ZMK Conversion Notes

Converted from: `vial_final.vil` (Vial QMK)

## Summary

| Category | Count | Status |
|----------|-------|--------|
| Layers used | 5 (0-4) | ✅ Fully converted |
| Macros | 11 (M0-M10) | ✅ Fully converted |
| Key overrides | 1 | ⚠️ Needs workaround |
| One-shot mods | 1 | ⚠️ Similar but different |
| Tap dance | 0 used | N/A |
| Combos | 0 used | N/A |

---

## Fully Converted Features

### Layer Switching
| QMK | ZMK | Notes |
|-----|-----|-------|
| `MO(2)` | `&mo 2` | Momentary layer |
| `MO(3)` | `&mo 3` | Momentary layer |
| `TG(1)` | `&tog 1` | Toggle layer |
| `LT4(KC_SPACE)` | `&lt4_space 4 SPACE` | Layer-tap (custom behavior) |

### Mod-Tap
| QMK | ZMK | Notes |
|-----|-----|-------|
| `LGUI_T(KC_ENTER)` | `&gui_enter LGUI RET` | Hold=Gui, Tap=Enter |

### Modifiers
| QMK | ZMK | Notes |
|-----|-----|-------|
| `LSFT(KC_1)` → `!` | `&kp EXCL` | Shifted symbols use named keys |
| `HYPR(KC_Q)` | `&kp LC(LA(LS(LG(Q))))` | All 4 mods + key |

### Macros
All macros converted to ZMK macro behaviors:
- `M0` → `&m_ctrl_a` (Ctrl+A)
- `M1-M10` → `&m_cmd_1` through `&m_cmd_0` (Cmd+1-0)

---

## Features Requiring Attention

### 1. One-Shot Meh Modifier

**QMK:**
```c
OSM(MOD_LCTL|MOD_LALT|MOD_LGUI)
```

**ZMK (current):**
```
&sk LC(LA(LGUI))
```

**Difference:** ZMK's sticky-key (`&sk`) works similarly but:
- Has configurable timeout (default 1000ms)
- Releases after next keypress OR timeout
- QMK's OSM has slightly different activation/release timing

**Options to match QMK more closely:**

1. **Configure sticky-key timing:**
```devicetree
&sk {
    release-after-ms = <1000>;
    quick-release;  // Release immediately after key-up
};
```

2. **Use macro for hold behavior:**
```devicetree
meh_hold: meh_hold {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    wait-ms = <0>;
    tap-ms = <0>;
    bindings = <&macro_press &kp LCTRL &kp LALT &kp LGUI>
             , <&macro_pause_for_release>
             , <&macro_release &kp LCTRL &kp LALT &kp LGUI>;
};
```

---

### 2. Key Override (GUI + CTRL → TAB)

**QMK:**
```json
{
  "trigger": "KC_LCTRL",
  "replacement": "KC_TAB",
  "trigger_mods": 8  // GUI modifier
}
```
When GUI is held and CTRL is pressed, sends TAB instead.

**ZMK:** No direct equivalent. Key overrides don't exist in ZMK.

**Workaround Options:**

#### Option A: Combo
```devicetree
combos {
    compatible = "zmk,combos";
    combo_gui_ctrl_tab {
        timeout-ms = <50>;
        // Positions of your GUI and CTRL keys
        key-positions = <...>;
        bindings = <&kp TAB>;
    };
};
```
⚠️ Triggers on simultaneous press, not "mod held + key pressed"

#### Option B: Mod-Morph (different trigger)
```devicetree
ctrl_or_tab: ctrl_or_tab {
    compatible = "zmk,behavior-mod-morph";
    #binding-cells = <0>;
    bindings = <&kp LCTRL>, <&kp TAB>;
    mods = <(MOD_LGUI)>;
};
```
Then use `&ctrl_or_tab` instead of `&kp LCTRL`.
⚠️ Changes CTRL key behavior when GUI is held.

#### Option C: OS-Level (Recommended)
Use Karabiner-Elements (macOS) or AutoHotkey (Windows) for this override.
This keeps keyboard firmware simple and is easier to modify.

---

### 3. Keypad vs Regular Numbers

**QMK used:**
```
KC_KP_1, KC_KP_2, ... KC_KP_0
KC_KP_ASTERISK, KC_KP_MINUS, KC_KP_EQUAL
```

**ZMK converted to:**
```
N1, N2, ... N0 (regular number row)
ASTRK, MINUS, EQUAL
```

**Why this matters:** Some applications distinguish between numpad and number row keys (e.g., some games, calculator apps).

**To use actual numpad codes in ZMK:**
```devicetree
&kp KP_N1    // Numpad 1
&kp KP_N2    // Numpad 2
...
&kp KP_MULTIPLY  // Numpad *
&kp KP_MINUS     // Numpad -
&kp KP_EQUAL     // Numpad =
```

---

### 4. Unknown Keycode 0x3734

**QMK:** `0x3734` appeared in the quote/apostrophe position.

**ZMK:** Converted to `&kp APOS` (apostrophe/single quote)

This is likely correct based on keyboard layout position. If it was something else (like a custom keycode), let me know and I'll adjust.

---

## Timing Settings

Your QMK config had these timing values:
```json
"settings": {
  "7": 200,   // Likely tapping term (200ms)
  ...
}
```

**ZMK equivalent (in hold-tap behaviors):**
```devicetree
tapping-term-ms = <200>;
```

For global behavior tuning, add to your `.conf` file:
```ini
# Debounce (if needed)
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=5
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=5
```

---

## ZMK Features Not in QMK

Consider adding these ZMK-specific features:

### Bluetooth Controls
```devicetree
&bt BT_SEL 0    // Select profile 0
&bt BT_SEL 1    // Select profile 1
&bt BT_CLR      // Clear current profile
&bt BT_NXT      // Next profile
```

### Output Toggle (USB/Bluetooth)
```devicetree
&out OUT_TOG    // Toggle between USB and BLE
&out OUT_USB    // Force USB
&out OUT_BLE    // Force Bluetooth
```

### Soft Off (Deep Sleep)
```devicetree
&soft_off       // Enter deep sleep
```

---

## Layer Summary

| Layer | Name | Purpose |
|-------|------|---------|
| 0 | Base | QWERTY with modifiers |
| 1 | Alt | Alternate (toggle from L3) - mostly transparent |
| 2 | Num/Sym | Numbers, symbols, brackets |
| 3 | Media | Macros (Cmd+1-0), volume, brightness, F-keys |
| 4 | Hyper | Hyper+key combos, arrow navigation |

---

## Files

- **Keymap:** `zmk_keymap.keymap`
- **Notes:** `zmk_conversion_notes.md` (this file)

## Next Steps

1. Copy `zmk_keymap.keymap` to your ZMK config repo
2. Adjust key positions if your physical layout differs
3. Decide on Key Override workaround (Option A, B, or C above)
4. Test and tune timing values if needed
5. Add Bluetooth controls if using wireless
