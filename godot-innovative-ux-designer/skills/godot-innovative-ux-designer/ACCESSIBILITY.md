# Game Accessibility Essentials (Godot 4.x)

Accessibility enables creativity and reaches more players. It's a foundation, not a limitation. Target Godot 4.4+ for `AccessibilityServer` screen-reader exposure; graceful degradation for 4.3 and earlier.

## Core Principles (POUR)

- **Perceivable**: Visual info has audio/text alternatives; subtitles; colorblind-safe state encoding; high-contrast modes.
- **Operable**: Gamepad navigation wired end-to-end; all controls remappable; no time-only inputs; reduced-motion available.
- **Understandable**: Consistent menu flow; clear prompts; localized; explain state changes.
- **Robust**: Works with `AccessibilityServer` + OS screen readers (NVDA/JAWS/VoiceOver in 4.4+); survives text scaling and container re-wrap.

## Contrast Requirements

| Element | Minimum ratio |
|---|---|
| Normal text | 4.5:1 |
| Large text (≥18pt / 24px) | 3:1 |
| UI components (button outline, focus ring, icons) | 3:1 |

**Check with**: online WCAG contrast tools against your Theme's `font_color` vs. `bg_color` on StyleBoxes. Also verify HUD over *actual gameplay backgrounds*, not just flat swatches.

## Gamepad & Keyboard Navigation

Every menu MUST be fully navigable without a mouse. Test by unplugging it.

### Built-in actions

Godot ships with `ui_accept`, `ui_cancel`, `ui_up`, `ui_down`, `ui_left`, `ui_right`, `ui_focus_next`, `ui_focus_prev`, `ui_page_up`, `ui_page_down`, `ui_home`, `ui_end`. Analog stick → d-pad mapping is handled automatically in 4.x.

### Focus wiring

```gdscript
# In a non-grid layout, auto focus_neighbor detection often fails.
# Wire explicitly:
start_button.focus_neighbor_bottom = continue_button.get_path()
continue_button.focus_neighbor_top = start_button.get_path()
continue_button.focus_neighbor_bottom = settings_button.get_path()
# ...etc

# For a grid, use GridContainer and let Godot handle it.
```

### Grab initial focus on open

```gdscript
func open_menu() -> void:
    show()
    # Defer by a frame so containers finish laying out:
    await get_tree().process_frame
    default_button.grab_focus()
```

### Focus visuals are mandatory

Every focusable Control needs a visible focus StyleBox. In your Theme, define `Button/focus` (and variations) as a StyleBoxFlat with:
- Thick border (2-4px) in your accent color, *or*
- Outer glow via `shadow_color` + `shadow_size`, *or*
- Background shift + inset border for low-vision clarity

Never rely on color alone — the focus ring should also add shape/size delta.

### Focus mode

- `FOCUS_ALL` (default on Button) — reachable by keyboard/gamepad and click
- `FOCUS_CLICK` — mouse only (use rarely, e.g. decorative labels that trigger something)
- `FOCUS_NONE` — never focusable (decorative elements, static labels)

## Remappable Controls

**Every** action bindable via in-game settings. Never hard-code key names in UI text.

### Query current binding dynamically

```gdscript
# Returns readable text like "E" or "Right Trigger"
static func get_action_prompt(action: String) -> String:
    var events := InputMap.action_get_events(action)
    if events.is_empty():
        return "?"
    var ev := events[0]
    if ev is InputEventKey:
        return OS.get_keycode_string(ev.physical_keycode)
    elif ev is InputEventJoypadButton:
        return _joy_button_name(ev.button_index)
    elif ev is InputEventMouseButton:
        return "Mouse %d" % ev.button_index
    return "?"

# In UI:
interact_prompt.text = "Press %s to open" % InputHelper.get_action_prompt("interact")
```

### Rebinding UI

Typical pattern:
1. Player clicks a binding row → row enters "listening" state.
2. Next `InputEvent` captured in `_input()`; validate it's not reserved (e.g., never bind system/menu actions away).
3. `InputMap.action_erase_events(action)`, `InputMap.action_add_event(action, new_event)`.
4. Save to `user://bindings.cfg` via `ConfigFile`.

```gdscript
func save_bindings() -> void:
    var cfg := ConfigFile.new()
    for action in ["move_up", "move_down", "move_left", "move_right", "jump", "interact"]:
        var events := InputMap.action_get_events(action)
        cfg.set_value("bindings", action, events)
    cfg.save("user://bindings.cfg")
```

### Reserved / system actions

Never let players rebind `ui_cancel` to the same key as `ui_accept`. Validate on save and offer "reset to defaults".

## Colorblind-Safe Design

~8% of men and 0.5% of women have some form of color vision deficiency. Deuteranopia (red-green) is most common.

### Rules

- **Never encode state in hue alone.** Always pair with one of: icon, shape, pattern, label, position.
- Red/green is the worst pair. Use blue+orange, or red+blue, or pair red with a crossed-out icon.
- Friendly/hostile markers: shape matters (circle vs. triangle), not just color.
- Status effects: icon + color + brief label in tooltip.

### Colorblind mode toggle

Provide three sim modes as a settings option:
- Deuteranopia simulation / shift
- Protanopia simulation / shift
- Tritanopia simulation / shift

Implement via a full-screen color-matrix shader on a `CanvasLayer` `ColorRect` at the top, or swap the Theme to a pre-adjusted palette.

```gdscript
# Colorblind filter CanvasLayer (top of scene tree, layer = 500):
extends CanvasLayer

@onready var rect: ColorRect = $ColorRect
@onready var shader_mat: ShaderMaterial = rect.material

func set_mode(mode: String) -> void:
    match mode:
        "none": rect.hide()
        "deuteranopia":
            rect.show()
            shader_mat.set_shader_parameter("matrix_mode", 1)
        "protanopia":
            rect.show()
            shader_mat.set_shader_parameter("matrix_mode", 2)
        "tritanopia":
            rect.show()
            shader_mat.set_shader_parameter("matrix_mode", 3)
```

## Subtitles & Captions

- **On by default.** Opt-out, not opt-in.
- Speaker labels: `BOB: "Don't go in there."` (color per speaker helps, but also use the name — colorblind-safe).
- **Ambient captions**: `[footsteps approach]`, `[door creaks open]`, `[distant explosion]`. Critical for players who can't hear.
- Minimum size: 24px at 1080p baseline; scale with the text-size slider.
- Background: semi-opaque dark panel behind caption, not just a text outline. Outlines alone fail on busy scenes.
- Line length: ≤42 characters per line, max 2 lines on screen at once.
- Duration: minimum 1.5 seconds even for very short lines.

```gdscript
extends PanelContainer  # the caption box

@onready var label: RichTextLabel = %Label

func show_caption(speaker: String, text: String, duration: float = 2.5) -> void:
    var prefix = "[b]%s:[/b] " % speaker if speaker != "" else ""
    label.text = "[center]%s%s[/center]" % [prefix, text]
    show()
    var t := create_tween()
    t.tween_interval(duration)
    t.tween_callback(hide)
```

## Reduced Motion

Some players experience motion sickness or vestibular disorders from screen shake, fast camera moves, parallax, and chromatic aberration.

### Settings toggle

Expose a single "Reduced Motion" toggle (or split into several for fine control):

- Screen shake: on/off
- Camera bob (first-person): on/off
- Damage vignette / chromatic aberration: on/off
- Menu transition intensity: full / reduced / instant
- Parallax backgrounds: on/off

### Global autoload pattern

```gdscript
# motion_settings.gd — autoload
extends Node

var reduced_motion: bool = false
var screen_shake_enabled: bool = true
var camera_bob_enabled: bool = true
var flashing_reduced: bool = false

func shake(camera: Node, amplitude: float, duration: float) -> void:
    if reduced_motion or not screen_shake_enabled:
        return
    # perform shake...
```

Always consult this singleton before kicking off decorative motion.

## Text Scaling

Offer 80% / 100% / 125% / 150% text size slider in accessibility settings.

### Implementation

```gdscript
# Apply by modifying the Theme's default_font_size:
func apply_text_scale(scale: float) -> void:
    var theme: Theme = get_tree().root.theme
    theme.default_font_size = int(16 * scale)  # base 16px
    # Containers will re-layout automatically on next frame.
```

Design menus to tolerate 150% scale without clipping:
- Use `size_flags_horizontal = SIZE_EXPAND_FILL` on labels
- Allow `autowrap_mode = AUTOWRAP_WORD_SMART` on `Label` nodes that might wrap
- Test all menus at 150% scale regularly

## Photosensitivity

~3% of the population has photosensitive epilepsy. Avoid triggering seizures.

### Rules

- No red/white flashes faster than 3Hz (three times per second)
- No large high-contrast patterns flashing
- No rapid saturation/brightness oscillation across >25% of the screen

### "Reduce flashing" toggle

Provide a setting that:
- Disables strobe effects (lightning, explosion flashes)
- Reduces brightness of screen flashes (cap at ~40% alpha)
- Lengthens flash durations so the flicker rate stays under 3Hz
- Softens chromatic aberration pulses

## Screen Reader Support (Godot 4.4+)

Godot 4.4 introduced the `AccessibilityServer`, which automatically exposes focusable `Control` nodes to the OS accessibility API (NVDA, JAWS, VoiceOver, Narrator).

### Enable and configure

In Project Settings → Accessibility, enable the server (default on in 4.4+).

### Custom accessibility labels

Standard Controls (`Button`, `Label`, `CheckBox`) are exposed automatically using their text. For custom controls or icon-only buttons, set:

```gdscript
# Icon-only button (e.g., a close "X"):
close_button.tooltip_text = "Close menu"
close_button.accessibility_name = "Close menu"  # 4.4+

# Custom health bar Control:
health_bar.accessibility_name = "Player health"
health_bar.accessibility_description = "Current HP: %d of %d" % [current_hp, max_hp]
```

Update `accessibility_description` when state changes (HP drops, etc.) so screen readers announce.

### TTS (text-to-speech)

`DisplayServer.tts_*` APIs let you speak arbitrary text via the OS:

```gdscript
# Speak a notification:
if DisplayServer.tts_is_speaking():
    DisplayServer.tts_stop()
var voices := DisplayServer.tts_get_voices_for_language("en")
var voice_id: String = voices[0] if not voices.is_empty() else ""
DisplayServer.tts_speak("Quest updated", voice_id)
```

Use for:
- Reading menu items aloud (optional "menu narration" toggle in settings)
- Announcing critical HUD changes (low HP, incoming objective)
- Reading dialogue aloud (optional, for players who can't/don't want to read)

### Fallback for older Godot

On 4.3 and earlier, `AccessibilityServer` isn't available. Document this limitation and provide:
- Rich tooltips with full context
- Consistent `Label` text (screen readers can OCR-read if using platform screen magnifiers)
- Full keyboard/gamepad nav (works regardless of screen reader)

## Quick Checklist

**Input & Nav:**
- [ ] Gamepad-only completes every menu
- [ ] Keyboard-only completes every menu
- [ ] All actions remappable (keyboard + gamepad) and persisted
- [ ] Focus visible on every Control (border, glow, or background shift)
- [ ] `grab_focus()` called on menu open

**Visuals:**
- [ ] Contrast ≥4.5:1 for text, ≥3:1 for large text/UI
- [ ] State never encoded in color alone (pair with icon/shape/label)
- [ ] Colorblind mode toggle (deuteranopia/protanopia/tritanopia)
- [ ] Text scale slider (80-150%) and menus survive 150%

**Audio:**
- [ ] Subtitles on by default, with speaker labels
- [ ] Ambient sound captions for critical audio
- [ ] Caption background panel (not outline alone), min 24px at 1080p

**Motion:**
- [ ] Reduced motion toggle disables shake, bob, aberration
- [ ] No flashes >3Hz; "reduce flashing" toggle available
- [ ] Menu transitions have an instant option

**Screen Reader (4.4+):**
- [ ] Icon-only buttons have `accessibility_name`
- [ ] Custom Controls have `accessibility_name` + `accessibility_description`
- [ ] TTS available for menu narration (optional setting)

**Localization:**
- [ ] All UI strings in translation files (CSV/PO)
- [ ] Longest-string test (Finnish/German) doesn't break layouts
- [ ] RTL support if targeting Arabic/Hebrew (set `layout_direction` on Controls)

## Resources

- [Game Accessibility Guidelines](https://gameaccessibilityguidelines.com/) — comprehensive checklist, categorized Basic/Intermediate/Advanced
- [AbleGamers APX](https://accessible.games/accessible-player-experiences/) — design patterns for disabled players
- [Xbox Accessibility Guidelines](https://learn.microsoft.com/en-us/gaming/accessibility/xbox-accessibility-guidelines)
- [Godot AccessibilityServer docs](https://docs.godotengine.org/en/stable/classes/class_accessibilityserver.html)
- [Godot InputMap & InputEvent](https://docs.godotengine.org/en/stable/tutorials/inputs/inputevent.html)
- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/) — contrast ratios and perception baselines transfer to games
- [Can I Play That?](https://caniplaythat.com/) — disabled-player game reviews; see what real players ask for
