# Design System Template (Godot 4.x)

Meta-framework for understanding what's fixed, project-specific, and adaptable in your game's UI system.

## Purpose

Distinguish between:
- **Fixed Elements** — universal rules that never change (spacing, accessibility baseline, typography logic)
- **Project-Specific Elements** — filled in per game based on fiction and tone (colors, fonts, motion feel)
- **Adaptable Elements** — context-dependent (gamepad-first vs. mouse-first, platform safe zones, error patterns)

---

## I. FIXED ELEMENTS

These foundations remain consistent across all projects.

### 1. Spacing Scale

**Fixed system (pixels at design resolution):**
```
4, 8, 12, 16, 24, 32, 48, 64, 96
```

**Usage:**
- `MarginContainer` margins, `VBoxContainer`/`HBoxContainer` `theme_override_constants/separation`, `StyleBoxFlat.content_margin_*`
- Mathematical relationships ensure visual rhythm
- Use multiples of the 4px base unit

**Why Fixed:**
Consistent spacing creates visual harmony regardless of game personality. Steam Deck, 1080p, and 4K all scale proportionally when the stretch mode is `canvas_items`.

### 2. Grid Logic

**Fixed structure:**
- Think in implicit grids even when using free-form layouts
- Use `GridContainer` with `columns = 3/4/6` for inventory, keybindings, option tiles
- For menus: `VBoxContainer` with generous `separation` (24-32px)
- Safe-area inset: `MarginContainer` with margins of ~5% of design resolution for TV/console overscan

**Why Fixed:**
Grid provides structural order. Game fiction shows through color, typography, StyleBox choices — not grid structure.

### 3. Accessibility Standards

**Fixed requirements:**
- **Contrast**: 4.5:1 for normal text, 3:1 for large text and UI components
- **Input target**: Minimum 44×44 pixels at design resolution (48×48 on Steam Deck touch mode)
- **Gamepad navigation**: complete; mouse-free; `grab_focus()` on menu open
- **Remappable controls**: every keyboard/gamepad action rebindable, persisted to `user://`
- **Subtitles**: on by default, speaker-labeled, ambient-captioned
- **Reduced motion toggle**: disables shake, bob, flashing beyond UX needs
- **Screen-reader support**: Godot 4.4+ AccessibilityServer; `accessibility_name` on custom Controls

**Why Fixed:**
Accessibility is not negotiable. It's a baseline for ethical, legal, and commercially successful games. See `ACCESSIBILITY.md`.

### 4. Typography Hierarchy Logic

**Fixed structure:**
- **Mathematical scale**: 1.25x (major third) for moderate contrast, or 1.333x (perfect fourth) for dramatic
- **Hierarchy levels**: Display → H1 → H2 → H3 → Body → Small → Caption
- **Line height**: 1.5x font size for body, 1.2-1.3x for headlines
- **Line length**: 45-75 characters per line optimal (text wrap containers help enforce this)
- **MSDF enabled** for all UI fonts — crisp scaling, no re-rasterization

**Example scale at 1.25x (base 16px):**
```
Display: 49px    (×3.05)
H1:      39px    (×2.44)
H2:      31px    (×1.95)
H3:      25px    (×1.56)
Body:    16px    (×1.00)
Small:   13px    (×0.80)
Caption: 10px    (×0.64)
```

**Why Fixed:**
Mathematical relationships create predictable, harmonious hierarchy. Specific fonts change per project, but the ratio logic doesn't.

### 5. Component Architecture

**Fixed patterns (apply via Theme):**
- **Button states**: Normal, Hover, Pressed, Focus, Disabled — StyleBox + font_color variation for each
- **Form structure**: Label above input (or left, for compact settings), error/helper below
- **Modal pattern**: `CanvasLayer` + dim background (`ColorRect` with `modulate.a = 0.6`) + centered `PanelContainer` + explicit close (B/ui_cancel/X button)
- **Card structure**: `PanelContainer` → `MarginContainer` → `VBoxContainer` with `Header / Body / Footer`

**Why Fixed:**
Players expect consistent component behavior. Visual appearance is project-specific; architecture is not.

### 6. Animation Timing Framework

**Fixed profiles:**
- **Lightweight** (icons, badges, chips): 150ms
- **Standard** (cards, panels, list items): 300ms
- **Weighty** (modals, scene transitions): 500ms

**Fixed easing intents:**
- **Entrances**: `Tween.TRANS_CUBIC + EASE_OUT` (fast start, soft land)
- **Exits**: `TRANS_CUBIC + EASE_IN` (soft start, sharp leave)
- **Transitions**: `TRANS_CUBIC + EASE_IN_OUT` (smooth both ends)
- **Playful**: `TRANS_BACK + EASE_OUT` or `TRANS_ELASTIC`
- **Loops**: `TRANS_SINE` (breathing, pulsing)

**Why Fixed:**
Natural physics feels consistent across games. Duration and easing create that feeling. See `MOTION-SPEC.md`.

---

## II. PROJECT-SPECIFIC ELEMENTS

Fill these in per game based on fiction and personality.

### 1. Color System

**Template structure** (define as Theme colors in a `.tres` file):

```
NEUTRALS (4-5 colors):
- Background darkest / lightest: _______
- Surface (panels, cards):       _______
- Border / divider:              _______
- Text secondary:                _______
- Text primary:                  _______

ACCENTS (1-3 colors):
- Primary action / focus:        _______
- Faction / gameplay marker:     _______ (optional)
- Status colors:
    Success / heal:              _______
    Warning / low resource:      _______
    Error / damage:              _______
    Info / quest:                _______
```

**Questions to answer:**
- What emotion should the game evoke? (Dread, wonder, triumph, calm, unease)
- Warm or cool neutrals?
- Which palette supports the fiction without drowning out gameplay signal?
- Do colors need to *mean* something (faction, damage type, element)?

**Examples:**

**Project A: Post-apocalyptic survival**
```
Neutrals: Warm rust + charcoal (#1a1410 → #d4b89e)
Primary: Burnt ochre (#c87a3e) — rare hope, warnings
Success: Moss green (#6b8e5a) — scavenged medkit
Error: Bleached bone white on rust (#f5efe2 on #7a2a1a)
Why: Reinforces decay; cool colors feel wrong in this world
```

**Project B: Synthwave racer**
```
Neutrals: Deep blue-violet → magenta gradient (#1a0b2e → #f20a8f)
Primary: Cyan (#12faee) — UI highlights, speed lines
Success: Green-cyan (#0affb7)
Error: Hot magenta (#ff0055)
Why: Lean into the aesthetic; CRT scanlines + bloom complete it
```

**Project C: Cozy farming sim**
```
Neutrals: Soft cream + warm brown (#fdf6e8 → #4a3220)
Primary: Sage (#7fa072) — harvest, growth
Success: Honey gold (#e8a940)
Error: Soft coral (#e8735c) — never alarming red
Why: Welcoming; never punishing; low stakes visually
```

### 2. Typography Pairing

**Template:**

```
DISPLAY / HEADLINE FONT: _______
- Import: FontFile resource, MSDF enabled
- Weight(s): _______
- Use: title screen, chapter headers, dialogue speaker labels
- Personality: _______

BODY / UI FONT: _______
- Import: FontFile resource, MSDF enabled
- Weight(s): _______
- Use: paragraph text, button labels, menu items, tooltips
- Personality: _______

MONO / NUMERIC FONT (optional): _______
- Use: HUD readouts (ammo, HP numerics, timers), code displays
- Why: tabular numerals prevent digits jumping when values change
```

**Pairing logic:**
- Serif + Sans-serif (classic, editorial — narrative games)
- Display + System-equivalent (distinctive titles + readable UI)
- Single font family (efficient, cohesive — minimalist games)

**Examples:**

**Project A: Gothic RPG**
```
Display: IM Fell English SC (rough, period serif) — H1-H2 only
Body: Inter Tight Medium — readable at small HUD sizes
Mono: JetBrains Mono — damage numbers
Why: Old-world feel without sacrificing legibility
```

**Project B: Sci-fi shooter**
```
Display: Orbitron (geometric, techy) — headers only, used sparingly
Body: Exo 2 — wide-angle rounded sans, futuristic but legible
Mono: Space Mono — HUD readouts
Why: Reinforces technology fiction; MSDF handles bloom-scaling
```

**Project C: Cozy sim**
```
Display: Fraunces (warm humanist serif, curved) — title + dialogue speakers
Body: Nunito — rounded, friendly sans
No mono: all numbers are approximate anyway
Why: Handwritten-adjacent warmth, no technology feel
```

### 3. Tone of Voice

**Template:**

```
GAME PERSONALITY:
- Formal ↔ Casual: _______ (1-10)
- Serious ↔ Playful: _______ (1-10)
- Authoritative ↔ Conversational: _______ (1-10)
- In-fiction ↔ Out-of-fiction narrator: _______ (1-10)

MICROCOPY EXAMPLES:
- Pause menu title: _______
- Confirm-quit text: _______
- Save success: _______
- Save failure: _______
- Controls hint: _______
- Empty inventory: _______
- Death / retry: _______

MOTION PERSONALITY:
- Speed: _______ (snappy / standard / slow)
- Feel: _______ (precise / smooth / bouncy / brutalist)
```

**Examples:**

**Project A: Souls-like**
```
Personality: Formal (7), Serious (9), Authoritative (9), In-fiction (7)
Pause title: "Paused" (single word, austere)
Confirm quit: "Abandon this attempt?"
Save success: (no message; the game autosaves silently)
Death: "YOU DIED" (unchanged across decades; iconic)
Motion: Slow, weighty, brutalist easing (TRANS_CUBIC)
```

**Project B: Party game**
```
Personality: Casual (9), Playful (10), Conversational (9)
Pause title: "BRB?"
Confirm quit: "Give up? :("
Save: "Saved! Keep going!"
Death: "Wipeout! Try again?"
Motion: Fast, bouncy (TRANS_BACK + EASE_OUT)
```

### 4. Animation Speed & Feel

**Template:**

```
SPEED PREFERENCE:
- Button feedback:     _______ (80-120ms / 120-180ms / 180-250ms)
- Menu state changes:  _______ (150ms / 250ms / 350ms)
- Scene transitions:   _______ (300ms / 500ms / 700ms)

EASING STYLE:
- Default easing:      _______ (TRANS_CUBIC / TRANS_QUART / TRANS_EXPO)
- Accent easing:       _______ (TRANS_BACK for pop / TRANS_ELASTIC for wobble)

MOTION INTENSITY BUDGET:
- Screen shake on:     _______ (never / rarely / often / mechanic)
- Camera bob on:       _______ (off / subtle / pronounced)
- Screen flash on:     _______ (off / damage only / frequent)
```

---

## III. ADAPTABLE ELEMENTS

Context-dependent implementations.

### 1. Component Variations (Theme Type Variations)

Define in your Theme as `Button/Primary`, `Button/Destructive`, `Button/Ghost` etc.

- **Primary**: full accent background (main CTA, one per screen section)
- **Secondary**: outline-only (alternative action)
- **Ghost**: transparent background, underline on hover (navigation, toolbars)
- **Destructive**: red-family accent (delete, quit, abandon)
- **Icon**: `TextureButton` with transparent background, size 44×44 minimum

**Adaptation rule:** project-specific colors fill the palette, but hierarchy logic is fixed.

### 2. Platform & Resolution Adaptation

**Adaptable per target:**

**PC keyboard+mouse:**
- Dense UI okay; small click targets acceptable
- Hover states matter
- Mouse cursor visible; context menus via right-click

**Gamepad-first (console / Steam Deck):**
- Larger focus targets (48×48+)
- Focus visuals MUST be prominent (thick border, glow)
- No hover states (skip them or map to focus)
- Button prompts match the platform icon set (Xbox ABXY, PlayStation shapes, Switch letters, Steam Deck glyphs)
- Use `Input.get_joy_name(0)` or `Input.get_connected_joypads()` to detect and swap glyph assets

**Touch (mobile):**
- Minimum 48×48; prefer 64×64 for primary actions
- Thumb reach zones (bottom half of portrait, sides of landscape)
- No hover; press states must be visible
- Avoid focus navigation (no gamepad assumed; but include it if supporting controller)

**Adaptable responsive pattern:**

```gdscript
# Detect input method and swap UI:
func _ready() -> void:
    if Input.get_connected_joypads().size() > 0:
        _configure_for_gamepad()
    else:
        _configure_for_kbm()

func _input(event: InputEvent) -> void:
    # Swap on the fly when input method changes:
    if event is InputEventJoypadButton:
        _show_gamepad_glyphs()
    elif event is InputEventMouseMotion or event is InputEventKey:
        _show_kbm_glyphs()
```

### 3. Safe-Area & Overscan

**Adaptable by platform:**

| Platform | Safe area inset |
|---|---|
| PC windowed/fullscreen | 0-2% (optional, aesthetic) |
| Steam Deck | ~2% (screen is edge-to-edge) |
| Console TV | **5% on all sides** (for overscan on older TVs) |
| Mobile | Account for notch/cutout via `DisplayServer.get_display_safe_area()` |

Wrap your main UI in a `MarginContainer` whose margins adapt:

```gdscript
var margin := _safe_area_margin_for_platform()
$SafeAreaMargin.add_theme_constant_override("margin_left", margin.x)
$SafeAreaMargin.add_theme_constant_override("margin_right", margin.z)
$SafeAreaMargin.add_theme_constant_override("margin_top", margin.y)
$SafeAreaMargin.add_theme_constant_override("margin_bottom", margin.w)
```

### 4. Dark / Light / Faction Themes

Not a simple inversion — each theme needs adjusted contrast.

**Light Mode (cozy morning vibes):**
```
Background: #fdf6e8
Text primary: #2a1f12 (cream/espresso, 12:1 contrast)
Text secondary: #5a4a38 (softer, 7:1)
```

**Dark Mode (atmospheric):**
```
Background: #18141f (not pure black — gives depth)
Text primary: #e8e0d4 (warm off-white, 14:1 — softer than pure white)
Text secondary: #9a9280 (6:1, still AA)
```

**Faction Themes:**
Swap the entire Theme based on story allegiance, region, or chapter:
```gdscript
get_tree().root.theme = load("res://ui/theme_vostok.tres")  # industrial red
# later:
get_tree().root.theme = load("res://ui/theme_nordheim.tres")  # cold blue/white
```

### 5. Loading & Progress States

**Context-dependent:**

| Duration | Pattern |
|---|---|
| <500ms | No indicator; feels instant |
| 500ms-3s | Spinner or skeleton placeholder |
| 3-10s | Progress bar + description ("Compiling shaders…") |
| >10s | Progress + cancel option + estimated time |
| Scene load | Use `ResourceLoader.load_threaded_request()`; show progress from `load_threaded_get_status()` |

### 6. Error & Failure Strategy

**Context-dependent:**

**Save/load errors:**
```
Transient (disk full momentarily): retry button
Corruption: "Save file damaged — use backup?" with explicit action
Permissions: helpful message + link to platform help
```

**Connection errors (multiplayer):**
```
Matchmaking fail: clear reason + retry
Lost connection: attempt reconnect with progress indicator
Desync: return to main menu with explanation, NOT a crash
```

**Content errors:**
```
Missing asset: never crash — use fallback and log
Unknown state: return to last known safe state
```

---

## DECISION TREE

When implementing a UI feature, ask:

### Is this...

**FIXED?**
- Does it affect accessibility, structural logic, or universal UX expectations?
- Examples: spacing scale, grid, contrast ratios, component architecture, motion timing profiles
- **Action**: Use the fixed system, no variation

**PROJECT-SPECIFIC?**
- Does it express the game's fiction, tone, or personality?
- Examples: colors, typography, tone of voice, motion feel, StyleBox style
- **Action**: Fill in the template for this project

**ADAPTABLE?**
- Does it depend on platform, context, or player setting?
- Examples: component variants, input method, safe-area, loading patterns, error handling
- **Action**: Choose variation based on context

---

## EXAMPLE: Implementing a "Confirm" Button

### Fixed (Always the same):
- Minimum size: 44×44 at design resolution
- Padding via StyleBox: 16px horizontal, 12px vertical (from spacing scale)
- States: Normal, Hover, Pressed, Focus, Disabled — all defined in Theme
- Motion: 120ms squash-stretch on press (`TRANS_BACK + EASE_OUT`)
- Audio: UI confirm sound on `pressed`, UI focus sound on `focus_entered`

### Project-Specific (Filled per game):
- **Project A (Souls-like)**: Deep red StyleBox with rough border, carved-stone font, "Confirm"
- **Project B (Party)**: Bright yellow StyleBox with bouncy bevel, rounded friendly font, "Let's Go!"
- **Project C (Cozy)**: Sage StyleBox with soft shadow, warm humanist font, "Okay!"

### Adaptable (Context-dependent):
- **Main menu context**: Primary variant (full color)
- **Settings context**: Secondary variant (outline only)
- **Quit confirm context**: Destructive variant (red-family)
- **Toolbar context**: Ghost variant (transparent until hover)

---

## VALIDATION CHECKLIST

Before finalizing a screen:

### Fixed Elements
- [ ] Uses spacing scale (4/8/12/16/24/32/48/64/96px)
- [ ] Meets contrast (4.5:1 normal, 3:1 large/UI)
- [ ] Input targets ≥44×44 at design resolution
- [ ] Typography follows mathematical scale
- [ ] Components use standard state architecture (Normal/Hover/Pressed/Focus/Disabled)
- [ ] Motion profiles match weight (150/300/500ms)
- [ ] Full gamepad navigation with `grab_focus()` on open

### Project-Specific Elements
- [ ] Game color palette applied via Theme resource
- [ ] Typography pairing chosen and justified
- [ ] Tone of voice consistent across microcopy
- [ ] Motion feel matches game personality

### Adaptable Elements
- [ ] Component variants appropriate for context
- [ ] Platform-appropriate button prompts (ABXY / ⨯○△□ / ZL-ZR / etc.)
- [ ] Safe-area inset applied for console/TV
- [ ] Loading patterns match operation duration
- [ ] Error handling matches error type

---

## PROJECT KICKOFF TEMPLATE

```
GAME TITLE: _______________________
GENRE: ____________________________
FICTION / SETTING: ________________

PLATFORM TARGETS:
- Primary: _______ (PC / console / Steam Deck / mobile)
- Secondary: _______
- Input method(s): _______ (kb+m / gamepad / touch / all)

PILLARS:
- Pillar 1: _______
- Pillar 2: _______
- Pillar 3: _______

UI TONE:
- Primary emotion: _______
- Warm or cool: _______
- Formal or casual: _______
- Diegetic / non-diegetic emphasis: _______

COLORS (fill the template):
- Neutral palette: _______
- Primary accent: _______
- Status palette: _______ / _______ / _______ / _______

TYPOGRAPHY (fill the template):
- Display font: _______
- Body font: _______
- Mono font (optional): _______
- MSDF enabled: yes

MICROCOPY:
- Pause title: _______
- Confirm quit: _______
- Death / retry: _______

MOTION:
- Speed: _______ (snappy / standard / slow)
- Style: _______ (precise / smooth / bouncy / brutalist)
- Reduced-motion override: yes (default toggle in settings)

ACCESSIBILITY:
- Colorblind modes: deuteranopia / protanopia / tritanopia
- Text scale: 80-150%
- Subtitles: on by default, speaker-labeled
- Remappable input: all actions

RESOLUTION TARGETS (see RESOLUTION-SCALING.md):
- Design resolution: _______ (e.g., 1920×1080)
- Stretch mode: canvas_items / viewport
- Aspect: keep / expand / keep_width
```

---

## MAINTAINING CONSISTENCY

### Documentation
- Commit your Theme `.tres` to version control
- Document WHY colors/fonts were chosen, not just what — future you will thank you

### Tooling
- Use a single master Theme resource; variants inherit
- Save StyleBoxes as individual `.tres` files for reuse
- Define input actions in Project Settings → Input Map once; never hard-code

### Reviews
- Audit: does new UI follow fixed elements?
- Validate: are project-specific elements intentional?
- Question: are adaptations justified by platform/context?

---

## EXAMPLES OF COMPLETE SYSTEMS

### System A: Atmospheric horror (PC/console)

**Fixed**: Standard spacing, contrast 4.5:1 minimum, major third type scale, 300ms standard motion
**Project-Specific**:
- Colors: Warm charcoal + muted burgundy + cream
- Typography: IM Fell English (display) + Inter Tight (body) + JetBrains Mono (HUD numerics)
- Tone: Austere, in-fiction ("Awaken" instead of "Continue")
- Motion: Slow, weighty (TRANS_EXPO + EASE_OUT)
**Adaptable**:
- Console: 5% safe-area inset, Xbox/PS glyphs per platform
- Diegetic-heavy HUD (wrist watch for map)
- Errors are atmospheric ("The connection is lost…")

### System B: Fast arcade racer (all platforms)

**Fixed**: Same foundational structure
**Project-Specific**:
- Colors: Deep violet + magenta + cyan synthwave
- Typography: Orbitron (display) + Exo 2 (body) + Space Mono (speed readout)
- Tone: Loud, energetic ("GO!" not "Start")
- Motion: Snappy with aggressive TRANS_BACK overshoot
**Adaptable**:
- Steam Deck: enlarged HUD, Deck glyph set
- Mobile: simplified HUD, thumb-friendly lane-change buttons
- Reduced-motion: swap chromatic aberration for flat color shifts

### System C: Narrative visual novel (PC/mobile)

**Fixed**: Same baseline accessibility and layout
**Project-Specific**:
- Colors: Cream, sepia, ink-black (manuscript aesthetic)
- Typography: Fraunces (display) + Source Serif (body)
- Tone: Literary, speaker-labeled dialogue
- Motion: Gentle, TRANS_SINE for ambient pulses
**Adaptable**:
- Mobile portrait: bottom-anchored dialogue box
- PC landscape: side character portraits, larger body text
- TTS: offered as optional menu narration (4.4+)

---

## KEY TAKEAWAY

**The framework lets you:**
- Maintain consistency (fixed elements)
- Express unique fiction (project-specific)
- Adapt to platform & context (adaptable elements)

**Without it:**
- Spacing reinvented every screen
- UI feels inconsistent between menus
- Brand overrides accessibility
- Platform-blind implementations feel wrong

**With it:**
- Speed: start from proven foundations
- Consistency: fixed elements guarantee it
- Flexibility: express the game's unique voice
- Context: adapt without breaking the system
