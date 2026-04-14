# Resolution Scaling & Layout (Godot 4.x)

Unlike the web (where the browser handles scaling), games run at arbitrary resolutions on arbitrary aspect ratios. Design once at a target resolution, then use Godot's stretch mode, anchors, and containers to handle everything else.

## Step 1 — Project Settings (do this once)

Navigate to **Project → Project Settings → Display → Window**.

### Stretch settings

| Setting | Recommended | Notes |
|---|---|---|
| **Viewport Width** | 1920 | Your design resolution (most common baseline) |
| **Viewport Height** | 1080 | |
| **Window Width Override** | 1280 | Default window opens at 1280×720 for testing |
| **Window Height Override** | 720 | |
| **Stretch Mode** | `canvas_items` | For most games — UI and 2D world scale crisply |
| **Stretch Aspect** | `expand` | Fills screen on any aspect; use anchors for edges. Choose `keep` if letterboxing is the aesthetic |
| **Scale Mode** | `fractional` | `integer` only for pixel-art games |

### Stretch Mode deep-dive

- **`disabled`**: Viewport resizes with window. Your UI at 4K will literally be tiny pixels. Don't use this for most games.
- **`canvas_items`** (recommended): Viewport stays at design size; content is scaled by the renderer. UI remains crisp if fonts are MSDF. Best balance for 99% of games.
- **`viewport`**: Everything renders at design resolution and is upscaled to window. Pixel-perfect for pixel art. Looks blurry for HD assets.

### Stretch Aspect deep-dive

- **`ignore`**: Distorts to fill. Never use.
- **`keep`**: Letterbox/pillarbox to preserve design aspect. Rigid but predictable. Good for cinematic or pixel-art games.
- **`keep_width` / `keep_height`**: Fills width/height, lets the other dimension be whatever the window is. The "extra" space becomes extended view. Useful for ultrawide support when you want to *see more*, not *see bigger*.
- **`expand`** (most common): Fills screen entirely. UI anchors handle edge cases (HUD in corner, menus centered). Requires anchor discipline.

## Step 2 — Resolution Targets

Design for 1920×1080 by default, but test across these targets:

| Target | Resolution | Aspect | Notes |
|---|---|---|---|
| 720p | 1280×720 | 16:9 | Older PCs, minimum spec |
| Steam Deck | 1280×800 | 16:10 | Handheld; 7" screen; text ≥20px readable |
| 1080p | 1920×1080 | 16:9 | Most common PC baseline; design resolution |
| 1440p | 2560×1440 | 16:9 | High-end desktop; MSDF fonts stay crisp |
| 4K UHD | 3840×2160 | 16:9 | Integer scale 2× from 1080p works cleanly |
| Ultrawide | 3440×1440 | 21:9 | Anchor HUD to safe zones, not screen edges |
| Super-ultrawide | 5120×1440 | 32:9 | Same as above, even more extreme |
| Console TV | 1920×1080 or 4K | 16:9 | **Respect 5% safe-area inset** for overscan |
| Switch handheld | 1280×720 | 16:9 | Lower GPU; minimize shader post-process |
| Switch docked | 1920×1080 | 16:9 | |
| Mobile landscape | ~2340×1080 | ~19.5:9 | Notch/cutout awareness |
| Mobile portrait | ~1080×2340 | ~9:19.5 | Bottom-anchored primary controls, thumb-reach |

### Aspect ratio strategy

**For 16:9 games on ultrawide:**
- Option A: letterbox/pillarbox (`keep` aspect) — preserves framing but wastes screen
- Option B: extend the view (`keep_height`) — world shows more on sides; HUD anchors to extended edges or stays in a `AspectRatioContainer` centered

**For mobile portrait/landscape switching:**
- Build two separate root UI scenes, swap on `get_viewport().size_changed`
- Or use `size_flags` + `alignment` Container settings that cope with both

## Step 3 — Anchors

Every `Control` has four anchors (`anchor_left`, `anchor_top`, `anchor_right`, `anchor_bottom`) — values 0.0 (left/top edge) to 1.0 (right/bottom edge). The Control's edges follow these anchors as the parent resizes.

### Use anchor presets (inspector dropdown "Layout")

| Preset | anchors | Use for |
|---|---|---|
| **Full Rect** | 0,0,1,1 | Overlays, damage vignettes, pause backdrop |
| **Top Left** | 0,0,0,0 | HP/status in corner, score counter |
| **Top Right** | 1,0,1,0 | Minimap, objective, alert icons |
| **Top Wide** | 0,0,1,0 | Top bar (nav, title strip) |
| **Bottom Left** | 0,1,0,1 | Tutorial hint, subtitle |
| **Bottom Right** | 1,1,1,1 | Version number, button prompts |
| **Bottom Center** | 0.5,1,0.5,1 | Hotbar, primary HUD cluster, dialogue box |
| **Bottom Wide** | 0,1,1,1 | Full-width dialogue/subtitle bar |
| **Center** | 0.5,0.5,0.5,0.5 | Modals, centered titles |
| **Left Wide** | 0,0,0,1 | Vertical nav column |
| **Right Wide** | 1,0,1,1 | Sidebar (inventory, details) |
| **Center Top** | 0.5,0,0.5,0 | Centered title |

### Programmatic anchor preset

```gdscript
control.set_anchors_and_offsets_preset(Control.PRESET_BOTTOM_CENTER)
```

## Step 4 — Containers (Prefer These Over Anchors for Layouts)

When you have multiple children that need to stack/align, use a `Container`. Containers manage their children's position and size — you don't hand-position inside a container.

```
Control (Full Rect)              ← root anchor
└── MarginContainer              ← safe-area inset
    └── VBoxContainer            ← vertical stack
        ├── Label "Settings"     ← child 1
        ├── HBoxContainer        ← horizontal row
        │   ├── Label "Volume"
        │   └── HSlider
        └── HBoxContainer
            ├── Label "Brightness"
            └── HSlider
```

### Container quick reference

| Container | Behavior |
|---|---|
| `VBoxContainer` | Stacks children vertically with `theme_override_constants/separation` gap |
| `HBoxContainer` | Stacks children horizontally |
| `GridContainer` | N-column grid. Set `columns`. Good for inventories, keybinding tables |
| `MarginContainer` | Adds padding via theme overrides (`margin_left/top/right/bottom`) |
| `AspectRatioContainer` | Forces child to a specific aspect ratio (16:9, 4:3, etc.) |
| `CenterContainer` | Centers one child |
| `PanelContainer` | StyleBox background + auto-padding around one child |
| `ScrollContainer` | Child can overflow; provides scrollbars |
| `HSplitContainer` / `VSplitContainer` | Draggable splitter, two children |
| `TabContainer` | Tabbed children; `tab_bar_align` for positioning |
| `FlowContainer` | Wraps children like text (good for tag clouds) |

### Size flags on children of containers

Children of containers respect `size_flags_horizontal` and `size_flags_vertical`:

| Flag | Meaning |
|---|---|
| `SIZE_FILL` | Fill available space (default; doesn't consume extra) |
| `SIZE_EXPAND_FILL` | Consume extra space (key flag for flexible layouts) |
| `SIZE_SHRINK_CENTER` | Center within allocated space |
| `SIZE_SHRINK_END` | Right/bottom-align |
| `SIZE_EXPAND` | Expand but don't fill (rare) |

**Common pattern — a row with left-aligned label + right-aligned value:**

```
HBoxContainer
├── Label "Volume"            size_flags_h = SIZE_EXPAND_FILL
└── HSlider                   size_flags_h = SIZE_FILL (fixed width via custom_minimum_size)
```

The label expands to consume left space; the slider keeps its minimum width on the right.

## Step 5 — Safe Area & Overscan

TVs crop ~5% of the image around the edges ("overscan"). Console titles *must* respect a safe area. PC is forgiving; mobile has notches/cutouts.

```gdscript
extends MarginContainer

func _ready() -> void:
    var viewport_size := get_viewport().get_visible_rect().size
    var inset := Vector2i()
    if _is_console():
        inset = Vector2i(viewport_size * 0.05)
    elif OS.has_feature("mobile"):
        var safe := DisplayServer.get_display_safe_area()
        # DisplayServer returns the safe rect; compute margins
        inset = Vector2i(
            safe.position.x,
            safe.position.y
        )
    add_theme_constant_override("margin_left",   inset.x)
    add_theme_constant_override("margin_right",  inset.x)
    add_theme_constant_override("margin_top",    inset.y)
    add_theme_constant_override("margin_bottom", inset.y)

func _is_console() -> bool:
    # Set your own platform detection; OS.has_feature("template") may help for exports
    return OS.has_feature("xbox") or OS.has_feature("playstation") or OS.has_feature("switch")
```

### Mobile notch/cutout

```gdscript
# iOS/Android safe area (Godot 4.x):
var safe: Rect2i = DisplayServer.get_display_safe_area()
var window: Vector2i = DisplayServer.window_get_size()
# safe.position is the top-left of the usable area
# safe.size is the usable dimensions
```

## Step 6 — Font Rendering for Multiple Resolutions

### Enable MSDF (Multi-channel Signed Distance Field)

For ANY font used in UI:

1. Select the font file in the FileSystem dock
2. Import tab → "Multichannel Signed Distance Field" → **enabled**
3. Reimport

MSDF fonts scale cleanly from 8px to 128px without re-rasterizing. They look crisp at 720p and 4K alike. One-time 10% larger file size; worth it every time.

### Fallback chain

For localization (CJK, Arabic, emoji):

```gdscript
# On your main font resource:
var main_font: FontFile = preload("res://ui/fonts/main.ttf")
var cjk_fallback: FontFile = preload("res://ui/fonts/notosans_cjk.ttf")
main_font.fallbacks = [cjk_fallback]
```

Godot automatically uses the fallback when a glyph isn't found.

### Dynamic font size (text scale slider)

```gdscript
# Player's accessibility setting:
func apply_text_scale(scale: float) -> void:
    var theme: Theme = get_tree().root.theme
    theme.default_font_size = int(16 * scale)  # base 16, adjust to your design
```

Design labels with `autowrap_mode = AUTOWRAP_WORD_SMART` and allow containers to re-solve. Always test at 150%.

## Step 7 — Input Target Sizing

Minimum input target size at design resolution:

| Input method | Minimum | Recommended |
|---|---|---|
| Mouse (PC) | 32×32 | 44×44 |
| Gamepad (focus) | n/a (focus visual handles it) | — |
| Touch (mobile) | 44×44 | 48×48 |
| Steam Deck touch | 48×48 | 56×56 |

Use `custom_minimum_size = Vector2(44, 44)` on `Button`, `TextureButton`. Let the StyleBox add visual padding on top.

## Step 8 — Runtime Resolution Changes

### Detect resize

```gdscript
func _ready() -> void:
    get_viewport().size_changed.connect(_on_viewport_resized)

func _on_viewport_resized() -> void:
    var size := get_viewport().get_visible_rect().size
    var aspect := size.x / size.y
    if aspect >= 2.0:
        _enable_ultrawide_layout()
    elif aspect <= 1.2:
        _enable_portrait_layout()
    else:
        _enable_standard_layout()
```

### In-game resolution change (settings)

```gdscript
# Apply from a settings menu:
DisplayServer.window_set_size(Vector2i(resolution_w, resolution_h))
# or for fullscreen:
DisplayServer.window_set_mode(DisplayServer.WINDOW_MODE_FULLSCREEN)
# or borderless fullscreen:
DisplayServer.window_set_mode(DisplayServer.WINDOW_MODE_EXCLUSIVE_FULLSCREEN)
```

Save preference to `user://settings.cfg`, restore on boot.

## Testing Protocol

In the Godot editor:

1. **Debug → Customize Run Instance** (or in the run config)
2. Set window size for each target, launch, and walk the main flow
3. Test every menu at each resolution
4. Test fullscreen AND windowed

Or quickly:

```gdscript
# Temporary test bindings:
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("debug_res_720p"):
        DisplayServer.window_set_size(Vector2i(1280, 720))
    elif event.is_action_pressed("debug_res_steamdeck"):
        DisplayServer.window_set_size(Vector2i(1280, 800))
    elif event.is_action_pressed("debug_res_1080p"):
        DisplayServer.window_set_size(Vector2i(1920, 1080))
    elif event.is_action_pressed("debug_res_ultrawide"):
        DisplayServer.window_set_size(Vector2i(3440, 1440))
    elif event.is_action_pressed("debug_res_portrait"):
        DisplayServer.window_set_size(Vector2i(1080, 1920))
```

## Testing Checklist

- [ ] Every menu legible at 1280×720 (minimum)
- [ ] Every menu legible at 1280×800 Steam Deck with actual-size preview
- [ ] Design resolution (1920×1080) is the baseline — no clipping, no scroll where not wanted
- [ ] 2560×1440 and 4K maintain crispness (MSDF fonts verified)
- [ ] 3440×1440 ultrawide: HUD not stranded in corners; main content in safe/comfortable viewing area
- [ ] 5% safe-area inset applied on console builds
- [ ] Mobile notch/cutout respected; critical UI not under notch
- [ ] Portrait mobile has bottom-anchored primary controls
- [ ] Text scale at 150% doesn't clip; wraps naturally
- [ ] All input targets ≥44×44 at design resolution
- [ ] Runtime resolution switching from settings works and persists

## Resources

- [Godot Docs — Multiple Resolutions](https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html) — definitive reference
- [Godot Docs — Size and Anchors](https://docs.godotengine.org/en/stable/tutorials/ui/size_and_anchors.html)
- [Godot Docs — Containers](https://docs.godotengine.org/en/stable/tutorials/ui/gui_containers.html)
- [Godot Docs — DisplayServer](https://docs.godotengine.org/en/stable/classes/class_displayserver.html)
- **Steam Deck UI guidelines** — Valve's Steam Deck developer docs cover text sizing and target sizing
- **Xbox/PlayStation TCR (Technical Certification Requirements)** — mandate specific safe-area rules for TV titles
