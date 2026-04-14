---
name: godot-innovative-ux-designer
description: Use when designing or implementing game UI in Godot 4.x — menus, HUDs, dialogue, inventory, pause/settings, or any scene using Control nodes, Theme resources, or .tscn UI layouts. Creates distinctive, production-grade game interfaces with intentional typography, coherent visual language, motion that serves the game, and game-accessibility (remappable input, gamepad focus, colorblind-safe palettes, subtitles, reduced motion). Avoids generic engine-default aesthetics.
metadata:
  version: 3.0.0
  engine: godot 4.4+
---

# Innovative Godot UX Designer

Create distinctive, production-grade game interfaces in Godot 4.x that avoid generic "engine default" aesthetics. Implement real working scenes and GDScript with exceptional attention to visual detail and creative commitment. Expert game UI/UX skill for unique, accessible, thoughtfully designed menus, HUDs, dialogue systems, inventories, and settings screens.

This skill emphasizes **bold creative commitment to a game's fiction**, breaking away from generic patterns (default Godot theme, placeholder Noto Sans, 50% grey panels), and building interfaces that reinforce the game's tone while remaining functional, gamepad-friendly, and accessible.

## Core Philosophy

**CRITICAL: Design Thinking Protocol**

Before building scenes, **ASK to understand context**, then **COMMIT BOLDLY** to a distinctive direction:

### Questions to Ask First
1. **Purpose**: What problem does this screen solve? Where does it sit in the player flow (boot → main menu → in-game HUD → pause → settings → save/load)?
2. **Fiction**: What world/tone does the game inhabit? Should UI be diegetic (lives in the world), non-diegetic (overlay), or mixed?
3. **Platform**: PC keyboard+mouse, gamepad-primary (console/Steam Deck), touch (mobile), or all three? This drives focus nav and target sizing.
4. **Constraints**: Target resolution(s)? Performance budget (is this a handheld/mobile title)? Localization requirements?
5. **Differentiation**: What makes this UNFORGETTABLE? What's the one detail a streamer will screenshot?

### Tone Options (Pick an Extreme)
Choose a clear aesthetic direction and execute with precision:
- **Brutally minimal** — stripped to essence, bold typography, vast negative space *(puzzle, roguelike, hardcore sim)*
- **Maximalist chaos** — layered, dense, visually rich, controlled disorder *(shmup, arcade, JRPG menus)*
- **Retro-futuristic** — vintage meets sci-fi, CRT scanlines, chromatic aberration *(synthwave, immersive sim)*
- **Organic/natural** — soft edges, earthy colors, nature-inspired textures *(cozy sim, farming, exploration)*
- **Luxury/refined** — elegant spacing, premium typography, subtle details *(strategy, management, narrative)*
- **Playful/toy-like** — bright colors, rounded shapes, bouncy motion *(party, platformer, kids)*
- **Editorial/magazine** — strong typography hierarchy, asymmetric layouts *(visual novel, detective, narrative)*
- **Brutalist/raw** — exposed structure, harsh contrasts, intentionally rough *(horror, survival, extraction)*
- **Art deco/geometric** — bold patterns, metallic accents, symmetric elegance *(noir, period games)*
- **Industrial/utilitarian** — functional, mechanical precision, readouts and gauges *(sim, mech, space)*
- **Hand-drawn/zine** — sketchy lines, paper textures, handwriting *(indie narrative, art games)*
- **Diegetic/in-world** — UI lives inside the fiction (clipboards, holograms, watches) *(immersive sim, horror)*

### After Getting Context
- **Commit fully** to the chosen direction — no half measures
- Present 2-3 alternative approaches with trade-offs (sketch node trees, describe StyleBox choices, color palettes)
- Then implement with precision: production-grade scenes, visually striking, memorable

## Foundational Design Principles

### Stand Out From Generic Patterns

**NEVER ship these engine-default aesthetics:**
- **Fonts**: Godot's default Noto Sans at default sizes, Inter, Roboto, Arial, Space Grotesk
- **Theme**: The unstyled default Theme (grey buttons, default focus ring, default 16px margins)
- **Colors**: Generic SaaS blue (#3B82F6), default `Color.RED` health bars, pure black/pure white extremes without warmth
- **StyleBoxes**: Default `StyleBoxFlat` with no corner radius variation, no border, no content_margin_* tuning
- **Effects**: Glass morphism, generic "blur behind pause menu" with no grading, unstyled `AcceptDialog` popups
- **Overall**: Anything that looks straight out of the Godot docs tutorial

**Instead, Create Atmosphere:**
- Import custom fonts (`.ttf`/`.otf`) with **MSDF** enabled for crisp scaling from Steam Deck to 4K
- Author custom `StyleBoxFlat` with intentional corner radii, asymmetric borders, noise/texture overlays via `StyleBoxTexture` 9-slicing
- Use `CanvasLayer` + `ColorRect` with custom shaders for CRT scanlines, vignette, chromatic aberration, paper texture
- Pair your UI motion with audio — a UI click without a sound is half-finished
- Consider custom mouse cursors (`Input.set_custom_mouse_cursor()`) that match the fiction

**Draw Inspiration From:**
- *Disco Elysium* (editorial typography, skill check drama), *Persona 5* (maximalist chaos, motion), *Metro Exodus* (diegetic watch), *Dead Space* (in-world holograms), *Returnal* (readable clarity in chaos), *Tunic* (manual-as-UI), *Signalis* (PS1 psychological horror), *Outer Wilds* (analog diegetic instruments)
- Arcade cabinet overlays, tabletop character sheets, pilot HUDs, film title cards — games borrow well from physical artifacts

**Visual Interest Strategies:**
- Unique palette pairs that match fiction (rust + ochre for post-apocalyptic; cyan + magenta for synthwave; sepia + burgundy for period)
- UI motion that couples to game events (heartbeat HUD throb at low HP, ammo counter slam on reload, damage vignette pulse)
- Background layers with parallax or shaders that add depth without distracting from the foreground interactable
- Typography pairings that create contrast (display serif for titles + condensed sans for stats)
- Diegetic details: a menu that looks like a folded paper map, a HUD that looks like a rearview mirror

### Core Design Philosophy

1. **Simplicity Through Reduction**
   - Identify the essential information a player needs *in this second* and eliminate everything else
   - HUDs especially: every pixel competes with the game. Hide non-critical info behind `visibility_changed` triggered by context (e.g., stamina only shows when depleted)
   - Every element must justify its existence on screen

2. **Material Honesty**
   - Godot UI has its own materiality — embrace it. `Control` nodes are screen-space primitives; don't pretend they're 3D objects unless that's the fiction
   - Buttons communicate affordance through StyleBox differentiation, hover/focus theme variation, audio feedback, AND subtle `scale` tween on press
   - Cards/Panels use `PanelContainer` + `StyleBoxFlat` with intentional `shadow_color` + `shadow_size` for elevation — *or* borders — *or* background differentiation. Pick one primary affordance per surface
   - Motion follows real-world physics adapted for responsiveness (see `MOTION-SPEC.md`)

3. **Functional Layering**
   - Hierarchy via typography scale, color contrast, spatial relationships
   - Layer information conceptually (primary → secondary → tertiary)
   - `CanvasLayer` order communicates literal Z — use intentionally (HUD: 10, transient popups: 50, pause menu: 100, debug overlay: 1000)
   - Modals sit above world; dropdowns above panels; tooltips above everything

4. **Obsessive Detail**
   - Every pixel, every signal, every tween. Excellence emerges from hundreds of intentional decisions
   - Every `Button` should feel hand-tuned: hover sound, press squash, release restore, focus glow, disabled desaturation — all consistent across the game
   - Details must serve clarity, not compete with it

5. **Coherent Design Language**
   - Every element visually communicates its function and faction. A weapon upgrade screen and a dialogue screen should feel made by the same hand
   - Theme type variations (`Button/Primary`, `Button/Destructive`, `Button/Ghost`) enforce this automatically

6. **Invisibility of Technology**
   - The best UI disappears into the game. Players should be processing world information, not parsing the HUD
   - Use diegetic framing where it fits the fiction; reserve non-diegetic overlays for info that genuinely needs to be system-level

### What This Means in Practice

**Color Usage:**
- Base palette: 4-5 neutral shades (background, surface, border/divider, text secondary, text primary) defined in a single `Theme` resource
- Accent palette: 1-3 bold colors (primary action, warning, error — sometimes faction colors)
- Neutrals are slightly desaturated, warm or cool based on the game's fiction
- Accents are saturated enough for contrast but tied to gameplay meaning (red = damage, not "submit button")

**Typography:**
- Headlines: Emotional, attention-grabbing, UNEXPECTED (personality over pure legibility)
- Body/UI: Functional, highly legible at target resolution (clarity over expression)
- 2-3 typefaces maximum, characterful and distinctive
- Import as `FontFile`; enable **MSDF (Multi-channel Signed Distance Field)** for crisp scaling across resolutions
- Clear mathematical scale (1.25x or 1.333x between sizes — see `DESIGN-SYSTEM-TEMPLATE.md`)
- NEVER default to Godot's Noto Sans fallback, Inter, or Roboto

**Animation:**
- Purposeful: guides attention, establishes relationships, provides feedback
- Subtle for UI interactions (100-300ms); weightier for screen transitions (300-500ms)
- Physics-informed: `Tween.TRANS_CUBIC + EASE_OUT` for entrances, `TRANS_BACK + EASE_OUT` for playful overshoot, `TRANS_ELASTIC` for bouncy spring, `TRANS_SINE` for gentle loops

**Spacing:**
- Mathematical scale (4/8/12/16/24/32/48/64/96 pixels — Godot uses pixels natively)
- Consistent application creates rhythm. Use `MarginContainer` and `VBoxContainer`/`HBoxContainer` `theme_override_constants/separation` rather than eyeball-positioning

### Design Decision Checklist

Before shipping any screen, verify:

1. **Purpose**: Does every Control serve a clear function?
2. **Hierarchy**: Does visual importance match information importance?
3. **Consistency**: Do similar elements look and behave similarly across all menus?
4. **Accessibility**: WCAG-style contrast? Remappable input? Gamepad focus chain? Colorblind-safe state encoding? Subtitles with speaker labels?
5. **Resolution**: Does it hold up at 1280×720, 1920×1080, 2560×1440, 3440×1440 ultrawide, 1280×800 Steam Deck? (See `RESOLUTION-SCALING.md`)
6. **Uniqueness**: Does this break from generic engine defaults?
7. **Approval**: Have I confirmed fiction, tone, and platform before implementing?

**Design System Framework:**

For distinguishing fixed (universal rules) vs. project-specific (game personality) vs. adaptable (context-dependent) elements, see `DESIGN-SYSTEM-TEMPLATE.md`.

## Godot UI Architecture

### Control Nodes & Anchoring

Every UI element is a `Control` or descendant. Key properties:

- **`anchors_preset`** — dropdown in inspector: Full Rect (overlays), Top Left (HP/status), Top Right (minimap), Bottom Center (hotbar), Center (modals). Sets `anchor_left/top/right/bottom` to 0 or 1.
- **`offset_left/top/right/bottom`** — pixel offsets from anchors. At `anchors_preset = Full Rect` with all offsets 0, the Control fills the parent.
- **`size_flags_horizontal` / `size_flags_vertical`** — for children of `Container` nodes: `SIZE_FILL`, `SIZE_EXPAND_FILL` (consumes extra space), `SIZE_SHRINK_CENTER`, `SIZE_SHRINK_END`.
- **`custom_minimum_size`** — preferred minimum size when inside a container.
- **`mouse_filter`** — `MOUSE_FILTER_STOP` (default, blocks events to nodes behind), `MOUSE_FILTER_PASS` (handles and forwards), `MOUSE_FILTER_IGNORE` (transparent to clicks). Set `IGNORE` on decorative panels or TTS-driven tooltips.

### Container Discipline

**Never hand-position children of a `Container` node.** The container will overwrite your offsets on layout.

| Node | Use for |
|---|---|
| `VBoxContainer` | Vertical stacks (menu buttons, form fields) |
| `HBoxContainer` | Horizontal rows (toolbar, inline stat readouts) |
| `GridContainer` | Regular grids (inventory slots, keybinding tables) |
| `MarginContainer` | Uniform padding (safe-area inset, card internal padding) |
| `AspectRatioContainer` | Constrain child to aspect (widescreen UI inside ultrawide window) |
| `CenterContainer` | Center one child (loading spinners, modal dialogs) |
| `PanelContainer` | Card-style surfaces with StyleBox background + automatic padding |
| `HSplitContainer` / `VSplitContainer` | Draggable splitter (settings panels, editor-style UIs) |
| `TabContainer` | Tabbed layouts (settings categories, inventory categories) |
| `ScrollContainer` | Overflow scroll (long lists, codex entries) |

### CanvasLayer Ordering

`CanvasLayer` separates UI from the 2D/3D world and allows Z-ordering independent of the scene tree:

- **Layer 0** (default) — world UI (world-space health bars on enemies)
- **Layer 10** — primary HUD (health, ammo, minimap)
- **Layer 20** — transient popups (damage numbers, pickup notifications)
- **Layer 50** — menus (inventory, map, dialogue)
- **Layer 100** — pause menu (set `process_mode = PROCESS_MODE_WHEN_PAUSED` so it works while tree is paused)
- **Layer 1000** — debug/dev overlays

```gdscript
# Pause menu that works while the game is paused:
extends CanvasLayer

func _ready() -> void:
    layer = 100
    process_mode = Node.PROCESS_MODE_WHEN_PAUSED
    hide()

func open() -> void:
    get_tree().paused = true
    show()
    $Panel/MenuButtons/Resume.grab_focus()

func close() -> void:
    hide()
    get_tree().paused = false
```

### World-Space UI (3D games)

For health bars, damage numbers, interaction prompts anchored to 3D objects:
- **`Sprite3D`** with a `ViewportTexture` — render a `Control`-based scene inside a `SubViewport`, display it on a `Sprite3D` in 3D space, and set `billboard = BILLBOARD_ENABLED`.
- Or **`Label3D`** for simple text.

### Fonts & Text Rendering

- Import `.ttf`/`.otf` as `FontFile`. In the import dock, enable **"Multi-channel Signed Distance Field"** for UI fonts — stays crisp from 12px to 128px without re-rasterizing.
- Use `FontVariation` to expose weight/spacing axes (variable fonts) or to apply spacing overrides.
- Outlines via theme overrides: `font_outline_color` + `font_outline_size` (on `Label`, `Button`, etc.) — essential for readable HUD text over busy backgrounds.
- `RichTextLabel` for BBCode (colored runs, inline icons via `[img]`, shake/wave effects). Prefer `Label` for static text so screen readers and TTS read cleanly.

### Signals for Interaction

Always connect via editor or `signal_name.connect(callable)` (Godot 4.x style, not `connect("signal_name", ...)`).

| Node | Key signals |
|---|---|
| `Button`, `TextureButton`, `CheckButton` | `pressed`, `toggled(pressed)`, `button_down`, `button_up` |
| Any `Control` | `focus_entered`, `focus_exited`, `mouse_entered`, `mouse_exited`, `gui_input(event)` |
| `LineEdit` | `text_changed(new_text)`, `text_submitted(text)` |
| `OptionButton` | `item_selected(index)` |
| `HSlider` / `VSlider` | `value_changed(value)` |
| `TabContainer` | `tab_changed(idx)` |

```gdscript
# Typical button wiring:
@onready var start_button: Button = $MarginContainer/VBoxContainer/Start

func _ready() -> void:
    start_button.pressed.connect(_on_start_pressed)
    start_button.focus_entered.connect(func(): AudioManager.play_sfx("ui_focus"))
    start_button.grab_focus()  # gamepad enters here immediately

func _on_start_pressed() -> void:
    AudioManager.play_sfx("ui_confirm")
    SceneTransition.fade_to("res://scenes/game.tscn")
```

## Component Library — Godot Equivalents

Map common UI primitives to Godot nodes:

| UI primitive | Godot node(s) | Notes |
|---|---|---|
| Button | `Button` (text), `TextureButton` (images), `CheckButton` (toggle), `OptionButton` (dropdown) | Use theme type variations for Primary / Destructive / Ghost |
| Dialog / Modal | `AcceptDialog`, `ConfirmationDialog`, or custom `Control` on a `CanvasLayer` | Built-ins are quick but always restyle — never ship default |
| Toast / Notification | Custom `Control` + `Tween` + auto-hide `Timer` | Slide-in from edge, fade out |
| Tabs | `TabContainer`, `TabBar` | Use `theme_override_styles/tab_selected` for custom look |
| Text input | `LineEdit` (single line), `TextEdit` (multiline), `SpinBox` (numeric) | Set `caret_blink` and custom `StyleBox` for normal/focus |
| Progress | `ProgressBar`, `TextureProgressBar` (radial/custom shapes for health rings) | Tween `value` for smooth changes |
| Tooltip | `Control.tooltip_text` (built-in) or override `_make_custom_tooltip(text)` for rich custom tooltip scenes |
| Icons | SVG imported as `Texture2D` (check "Scalable" in import) or `AtlasTexture` from a spritesheet |
| Lists | `ItemList`, or `VBoxContainer` inside `ScrollContainer` for custom row scenes |
| Trees | `Tree` (for editor-like hierarchies) |
| Scrolling content | `ScrollContainer` with one child (typically another container) |
| Dialogue box | Custom `Control` with `RichTextLabel` using `visible_ratio` animation for typewriter effect |
| Health / stamina bar | `TextureProgressBar` with custom `under_texture`/`progress_texture`/`over_texture`, or `Control` with `_draw()` override for full control |
| Minimap | `SubViewport` rendering a top-down camera, displayed via `TextureRect` with `SubViewportContainer` |
| Inventory grid | `GridContainer` of slot scenes; each slot is a `Control` accepting drag-drop via `_get_drag_data` / `_can_drop_data` / `_drop_data` |

## Theme & StyleBox System

### Theme Resources

A `Theme` resource (`.tres`) centralizes visual styling. Apply once at the root via `get_tree().root.theme = load("res://ui/theme_main.tres")` and every Control inherits.

- **Colors**: `font_color`, `font_outline_color`, `selection_color`, etc. per node type.
- **Constants**: `margin_left`, `separation`, `h_separation`, `outline_size`, etc.
- **Fonts**: `font`, `font_size` per node type (`Label/font`, `Button/font`, etc.)
- **Icons**: `Tree/arrow`, `CheckBox/checked`, etc.
- **StyleBoxes**: `normal`, `hover`, `pressed`, `focus`, `disabled` per node type.
- **Type Variations**: define `Button/Primary`, `Button/Destructive`, `Button/Ghost` as variants of `Button`. Apply via `theme_type_variation = "Primary"` on the button.

### StyleBoxFlat — The Workhorse

For panels, buttons, inputs, most UI surfaces:

```gdscript
var sb := StyleBoxFlat.new()
sb.bg_color = Color("#1a1410")
sb.border_width_left = 2
sb.border_width_top = 2
sb.border_width_right = 2
sb.border_width_bottom = 2
sb.border_color = Color("#c87a3e")  # burnt ochre accent
sb.corner_radius_top_left = 4
sb.corner_radius_top_right = 4
sb.corner_radius_bottom_right = 4
sb.corner_radius_bottom_left = 0  # intentional asymmetry
sb.content_margin_left = 16
sb.content_margin_top = 12
sb.content_margin_right = 16
sb.content_margin_bottom = 12
sb.shadow_color = Color(0, 0, 0, 0.4)
sb.shadow_size = 6
sb.shadow_offset = Vector2(0, 2)
```

Prefer authoring StyleBoxes in the editor (as saved `.tres`) and reference from the Theme — keeps visuals out of code.

### StyleBoxTexture — 9-slicing

For hand-painted/pixel-art UI (wooden frames, scrolls, borders):
- Provide a texture and set `texture_margin_left/top/right/bottom` to define the stretchable center vs. the fixed corners/edges.
- Set `axis_stretch_horizontal`/`_vertical` to `TILE` for repeated textures (brick borders) or `STRETCH` for smooth gradients.

### Theme Type Variations Example

```gdscript
# In theme_main.tres, define:
#   Button (base): grey StyleBox, Noto Sans fallback
#   Button/Primary: ochre StyleBox, Bold font
#   Button/Destructive: crimson StyleBox
#   Button/Ghost: transparent StyleBox, underline on hover

# In a scene:
var confirm_button = Button.new()
confirm_button.theme_type_variation = "Primary"
confirm_button.text = "Confirm"
```

### Theme Swap (Light/Dark or faction themes)

```gdscript
func apply_theme(variant: String) -> void:
    match variant:
        "dark":   get_tree().root.theme = load("res://ui/theme_dark.tres")
        "light":  get_tree().root.theme = load("res://ui/theme_light.tres")
        "faction_vostok": get_tree().root.theme = load("res://ui/theme_vostok.tres")
```

## Interaction Design

### User Experience Patterns

**Core UX Principles:**

1. **Direct Manipulation**
   - Drag-and-drop inventory slots (override `_get_drag_data` / `_can_drop_data` / `_drop_data` on `Control`)
   - Click-and-hold to equip, radial menus for quick select
   - Sliders for ranges (volume, sensitivity), not +/- buttons

2. **Immediate Feedback**
   - Every button press: visual (scale punch), audio (UI SFX), sometimes haptic (`Input.start_joy_vibration()`)
   - Skeleton/placeholder for anything loading >300ms (scene transitions, save file scans)
   - Success: checkmark + color flash + sound. Error: shake + red tint + error sound. Never silent failure
   - Loading states on any async work (cloud saves, shader compilation warm-up)

3. **Consistent Behavior**
   - All menus: `ui_cancel` backs out one level; `ui_accept` confirms focus; `ui_up/down/left/right` navigates
   - All modals: animate in 200-300ms, close on `ui_cancel`, focus returns to the button that opened it
   - All dialogues: same advance input, same skip-text input, same speaker-label style

4. **Forgiveness**
   - Confirm destructive actions (delete save, quit to desktop). Default focus on "Cancel", not "Confirm"
   - Never lose player input: form inputs preserve on error, dialogue choices are undoable where the fiction allows
   - Auto-save generously (indicate clearly with a subtle icon; never interrupt play)

5. **Progressive Disclosure**
   - Tutorial tooltips appear contextually, not as a wall of text on first launch
   - "Advanced settings" hidden behind an expand toggle (accessibility, control remapping, graphics tweaks)
   - Inventory: summary stats first (name, icon, rarity), full tooltip on hover/focus (description, comparison, lore)

**Game UX Patterns:**

1. **Gamepad-First Navigation**
   - Every interactive screen MUST work with gamepad alone. Test with mouse unplugged.
   - Wire `focus_neighbor_top/bottom/left/right` explicitly where auto-detection fails (non-grid layouts, skipping disabled buttons)
   - Call `grab_focus()` on the expected default button when a menu opens. Never leave focus dangling.
   - Respect Steam Deck / Switch conventions (A = confirm, B = cancel; bottom-right OK, bottom-left back in Japanese region builds)

2. **Context-Aware HUD**
   - Hide/fade non-critical HUD when not needed. Stamina bar only appears when depleted. Ammo counter only in combat.
   - Hit the player's context: driving sections get speedometer + map; dialogue gets nothing but subtitles + choices
   - Use `modulate:a` tweens for fade; don't just `hide()` (abrupt)

3. **Juice & Feel** (see "Game Feel" section below)

## Game UI Categories

Classify every UI element. Most games mix all four:

| Category | Definition | Examples |
|---|---|---|
| **Non-diegetic** | Outside the fiction, screen-space overlay | Health bar in top-left, minimap, objective markers |
| **Diegetic** | Lives inside the game world, characters could see it | *Dead Space* health spine, *Metro* wrist watch, *Far Cry 2* paper map |
| **Spatial** | World-anchored but not part of the fiction | Floating damage numbers, waypoint beacons, enemy outlines |
| **Meta** | Screen-space but fiction-aware | Blood splatter at low HP, radio static when stunned, vignette when bleeding |

**Choosing:**
- Diegetic deepens immersion but is harder to read at a glance. Use for games prioritizing atmosphere over twitch reaction.
- Non-diegetic is readable and conventional. Use for competitive, information-dense, or accessibility-critical games.
- Spatial is great for 3D games that need to communicate world info without committing to fiction.
- Meta is a spice — use sparingly for emotional moments.

## Game Feel / Juice

Feedback intensity must match action consequence. A paper-cut hit should not shake the screen like a rocket.

| Technique | What it is | Godot implementation |
|---|---|---|
| **Hit-stop** | Freeze game time 50-150ms on impact | `Engine.time_scale = 0.0`; wait with `get_tree().create_timer(0.08, true, false, true)`; restore to 1.0. The `true` flags make it ignore time_scale itself |
| **Screen shake** | Camera jitter, amplitude matched to hit | Add random `offset` to `Camera2D`/`Camera3D` over duration, decay to zero |
| **Squash-stretch** | UI scale punch on click | `Tween` scale 1.0 → 0.92 → 1.05 → 1.0 over 200ms |
| **Chromatic aberration** | RGB channel separation on damage | `CanvasLayer` with shader on `ColorRect`, ramp `aberration_strength` up briefly |
| **Time dilation** | Slow-mo on kill / critical | `Engine.time_scale = 0.3` for 500ms, ease back to 1.0 |
| **Camera punch** | Kickback on shot | Animate camera rotation 2° and recover |
| **Particle feedback** | Coin burst, sparks on button | `GPUParticles2D` / `CPUParticles2D` emitted at click position |
| **Impact frames** | Full-screen white/color flash | `ColorRect` full-rect, tween `modulate:a` 0 → 0.6 → 0 |
| **Freeze-frame** | Pause on kill cam | Take screenshot, display, release on input |

```gdscript
# Hit-stop utility:
static func hit_stop(duration: float = 0.08) -> void:
    Engine.time_scale = 0.0
    await Engine.get_main_loop().create_timer(duration, true, false, true).timeout
    Engine.time_scale = 1.0

# Button squash-stretch:
func _on_pressed() -> void:
    var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_BACK)
    t.tween_property(self, "scale", Vector2(0.92, 0.92), 0.06)
    t.tween_property(self, "scale", Vector2(1.0, 1.0), 0.12)
```

See `MOTION-SPEC.md` for full motion philosophy.

## Navigation & Scene Flow

- **Main menu → game**: use a `SceneTransition` autoload that fades to black, calls `get_tree().change_scene_to_file(path)`, fades back.
- **Pause menu**: `CanvasLayer` with `process_mode = PROCESS_MODE_WHEN_PAUSED`; sets `get_tree().paused = true` on open.
- **Settings**: nested `TabContainer` (Graphics / Audio / Controls / Accessibility / Gameplay); each tab a separate scene instanced into the tab.
- **Back-stack**: maintain a stack of menus so `ui_cancel` always unwinds cleanly. Don't rely on `hide()`/`show()` on sibling nodes — use an explicit state machine for complex menus (pause → options → controls → remap).
- **Save menu**: disable "Save" button during actual save I/O; show spinner; enable on completion.

## Loading & Async States

- Fast (<500ms): no indicator (feels instant). Queue the work; show result when ready.
- Medium (500ms-3s): skeleton or spinner. Keep UI responsive (don't freeze main thread).
- Long (>3s): progress bar + description of current step ("Compiling shaders…", "Loading save data…"). Provide cancel if possible.
- Scene loads: use `ResourceLoader.load_threaded_request()` + `load_threaded_get_status()` for non-blocking loads; show progress.

## Accessibility Standards

See `ACCESSIBILITY.md` for the full guide.

**Non-negotiable baseline:**
- Remappable controls (all keys + gamepad buttons rebindable, saved to `user://`)
- Gamepad navigation fully wired (no mouse-only interactions)
- Contrast: 4.5:1 for normal text, 3:1 for large text & UI components
- Never encode state in color alone (pair with shape, icon, pattern)
- Subtitles on by default with speaker labels and ambient sound captions
- Reduced motion toggle (disables screen shake, heavy transitions, camera bob)
- Text scale slider (80% / 100% / 125% / 150%)
- Photosensitivity: no red/white flashes >3Hz; "reduce flashing" toggle
- Godot 4.4+: set `accessibility_name` / `accessibility_description` on custom Controls for OS screen readers

## Design Process & Testing

### Design Workflow

1. **Understand Context** — game pillars, platform targets, player skill level, session length
2. **Sketch Node Tree** — outline scene hierarchy and containers before opening the editor
3. **Block Out in Greybox** — plain StyleBoxFlat in neutral grey, focus on layout and focus flow
4. **Add Fiction** — swap in custom theme, fonts, iconography
5. **Add Motion** — Tween button presses, menu transitions, HUD reactions
6. **Add Audio** — every button click, focus change, confirm, cancel, error
7. **Test at Every Resolution** — 1280×720, 1920×1080, 1280×800 Steam Deck, 3440×1440 ultrawide, portrait mobile if applicable
8. **Test Gamepad-Only** — unplug mouse; can you reach every control?
9. **Test Reduced Motion** — toggle setting on; is the game still usable?
10. **Test Screen Reader** — enable `AccessibilityServer` in 4.4+, walk through main menu with NVDA/VoiceOver

### Testing Checklist

**Visual:**
- Every resolution target from `RESOLUTION-SCALING.md`
- Light and dark themes (if supported)
- Every localization with longest-string test strings (German/Finnish explode short labels)

**Interaction:**
- Gamepad-only run through every menu
- Keyboard-only (some players have no mouse)
- Mouse + keyboard
- Touch (if mobile)

**Accessibility:**
- Colorblind simulation (deuteranopia/protanopia/tritanopia)
- Text scale at 150%
- Reduced motion on
- Screen reader walk-through (4.4+)

## Examples

### Example 1: Main Menu Button

**Questions to ask first:**
```
Before building this menu, I have a few questions:
1. What's the game's fiction — sci-fi, fantasy, noir, cozy?
2. Platform priority — PC kb+m, gamepad-first, Steam Deck, mobile?
3. Tone — brutalist minimal, maximalist chaos, retro-futuristic, editorial?
4. One memorable detail — a signature typeface, a unique transition, a diegetic frame?
```

**After approval, implementation:**

Scene structure:
```
MainMenu (Control, full rect)
└── MarginContainer (inset for safe area)
    └── VBoxContainer (alignment_vertical = center)
        ├── TitleLabel (custom font, MSDF, 96px)
        ├── VBoxContainer [buttons]
        │   ├── StartButton (theme_type_variation = "Primary")
        │   ├── ContinueButton
        │   ├── SettingsButton
        │   └── QuitButton (theme_type_variation = "Ghost")
        └── VersionLabel (bottom-right anchor)
```

```gdscript
extends Control

@onready var start_button: Button = %StartButton
@onready var continue_button: Button = %ContinueButton
@onready var settings_button: Button = %SettingsButton
@onready var quit_button: Button = %QuitButton

func _ready() -> void:
    continue_button.disabled = not SaveSystem.has_save()

    start_button.pressed.connect(_on_start)
    continue_button.pressed.connect(_on_continue)
    settings_button.pressed.connect(_on_settings)
    quit_button.pressed.connect(_on_quit)

    for b in [start_button, continue_button, settings_button, quit_button]:
        b.focus_entered.connect(func(): AudioManager.play("ui_focus"))

    _animate_in()
    (continue_button if not continue_button.disabled else start_button).grab_focus()

func _animate_in() -> void:
    modulate.a = 0.0
    var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
    t.tween_property(self, "modulate:a", 1.0, 0.4)

func _on_start() -> void:
    AudioManager.play("ui_confirm")
    SceneTransition.fade_to("res://scenes/intro.tscn")
```

### Example 2: Health Bar HUD (Non-diegetic with meta flourish)

**Concept:** primary bar is non-diegetic readable health; below 25%, a meta layer kicks in — pulse + chromatic aberration.

```gdscript
extends Control

@onready var bar: TextureProgressBar = $HealthBar
@onready var damage_vignette: ColorRect = $"/root/Main/CanvasLayer/DamageVignette"

var current_hp: float = 100.0
var max_hp: float = 100.0

func set_hp(new_hp: float) -> void:
    var old := current_hp
    current_hp = clamp(new_hp, 0, max_hp)

    var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
    t.tween_property(bar, "value", current_hp, 0.25)

    if current_hp < old:
        _damage_flash()

    _update_low_hp_pulse()

func _damage_flash() -> void:
    if MotionSettings.reduced_motion:
        return
    var t := create_tween()
    t.tween_property(damage_vignette, "modulate:a", 0.6, 0.08)
    t.tween_property(damage_vignette, "modulate:a", 0.0, 0.4)

func _update_low_hp_pulse() -> void:
    if current_hp / max_hp <= 0.25:
        _pulse_heartbeat()

func _pulse_heartbeat() -> void:
    if MotionSettings.reduced_motion:
        return
    var t := create_tween().set_loops()
    t.tween_property(bar, "self_modulate", Color(1.4, 0.8, 0.8), 0.4)
    t.tween_property(bar, "self_modulate", Color(1.0, 1.0, 1.0), 0.4)
```

### Example 3: Dialogue Box (Editorial / narrative game)

**Concept:** typewriter reveal via `RichTextLabel.visible_ratio`, speaker label in display serif, body in refined sans.

```gdscript
extends PanelContainer

@onready var speaker: Label = %Speaker
@onready var body: RichTextLabel = %Body
@onready var continue_icon: TextureRect = %ContinueIcon

var type_speed_chars_per_sec: float = 45.0
var _finished: bool = false

func show_line(speaker_name: String, text: String) -> void:
    speaker.text = speaker_name
    body.text = text
    body.visible_ratio = 0.0
    continue_icon.hide()
    _finished = false

    var duration := text.length() / type_speed_chars_per_sec
    var t := create_tween()
    t.tween_property(body, "visible_ratio", 1.0, duration)
    t.finished.connect(_on_line_complete)

func _on_line_complete() -> void:
    _finished = true
    continue_icon.show()
    var pulse := create_tween().set_loops()
    pulse.tween_property(continue_icon, "position:y", -4.0, 0.6).as_relative()
    pulse.tween_property(continue_icon, "position:y", 4.0, 0.6).as_relative()

func _input(event: InputEvent) -> void:
    if event.is_action_pressed("ui_accept"):
        if _finished:
            advance()
        else:
            body.visible_ratio = 1.0  # skip to end
```

## Common Patterns to Avoid

**NEVER:**
- Ship Godot's default Theme unchanged
- Use Noto Sans / Inter / Roboto as the primary UI font
- Hard-code input key names in UI text ("Press E to open") — always query `InputMap.action_get_events()`
- Encode state in color alone (red = bad, green = good fails for 8% of men)
- Skip `grab_focus()` when opening a menu — strands gamepad players
- Use `_input(event)` when `_unhandled_input(event)` would respect other nodes
- Animate `size` on a container every frame — triggers full layout re-solve
- Ship a game without subtitles, remappable controls, or gamepad navigation
- Use `AcceptDialog` in its default unstyled form
- Forget `process_mode = PROCESS_MODE_WHEN_PAUSED` on pause-menu CanvasLayers

**ALWAYS:**
- Ask about game fiction, platform, tone BEFORE building
- Commit boldly to one aesthetic direction
- Enable MSDF on all UI fonts
- Use Theme type variations for `Button/Primary`, `Button/Destructive`, etc.
- Wire `focus_neighbor_*` explicitly for non-grid layouts
- Pair color with shape/icon/pattern for state encoding
- Gate screen shake, heavy flashes, and camera bob behind reduced-motion setting
- Query `InputMap` for current bindings in prompt text
- Call `grab_focus()` on the expected default button when a menu opens
- Test gamepad-only before shipping
- Remember: Godot's UI system is powerful — a bespoke theme is only a few hours of work

## Version History

- v3.0.0 (2026-04-14): Ported from web to Godot 4.x — Theme/StyleBox/Tween, game feel, diegetic UI categories, game-specific accessibility
- v2.0.0 (2025-11-22): Creative liberation update (web)
- v1.0.0 (2025-10-18): Initial release (web)

## References

- **Godot Docs — UI Introduction**: https://docs.godotengine.org/en/stable/tutorials/ui/index.html
- **Godot Docs — Theme**: https://docs.godotengine.org/en/stable/tutorials/ui/gui_skinning.html
- **Godot Docs — Size and anchors**: https://docs.godotengine.org/en/stable/tutorials/ui/size_and_anchors.html
- **Godot Docs — Multiple resolutions**: https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html
- **Godot Docs — AccessibilityServer (4.4+)**: https://docs.godotengine.org/en/stable/classes/class_accessibilityserver.html
- **Game Accessibility Guidelines**: https://gameaccessibilityguidelines.com/
- **AbleGamers APX (Accessible Player Experiences)**: https://accessible.games/accessible-player-experiences/
- **WCAG 2.1 (contrast/text baselines still apply)**: https://www.w3.org/WAI/WCAG21/quickref/
- **Google Fonts** (MSDF-friendly): https://fonts.google.com/
- **Juice it or lose it (Martin Jonasson / Petri Purho)**: GDC talk on game feel

**Progressive Disclosure Files:**
- `ACCESSIBILITY.md` — gamepad nav, remappable controls, colorblind, subtitles, TTS
- `MOTION-SPEC.md` — Tween/AnimationPlayer/AnimationTree, game-feel catalog
- `DESIGN-SYSTEM-TEMPLATE.md` — fixed / project-specific / adaptable framework
- `RESOLUTION-SCALING.md` — stretch mode, anchors, containers, resolution targets
