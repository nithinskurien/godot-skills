# Motion Specification (Godot 4.x)

Motion should surprise and delight while serving function. Animation is a creative tool — and in games, feedback intensity must match action consequence.

## Easing Intent → Godot Mapping

Godot's `Tween` uses `TransitionType` + `EaseType` enums. Pick by intent, not by looking up Bezier curves.

| Intent | `Tween.TransitionType` | `Tween.EaseType` | Use for |
|---|---|---|---|
| Standard entrance | `TRANS_CUBIC` | `EASE_OUT` | Panels sliding in, fades on appear |
| Sharp exit | `TRANS_CUBIC` | `EASE_IN` | Panels closing, fades on disappear |
| Smooth transition | `TRANS_CUBIC` | `EASE_IN_OUT` | Camera moves, state changes |
| Playful overshoot | `TRANS_BACK` | `EASE_OUT` | Button pops, item pickups, celebratory motion |
| Bouncy spring | `TRANS_ELASTIC` | `EASE_OUT` | Physics-y wobble, rubber-band snaps |
| Sudden stop | `TRANS_EXPO` | `EASE_OUT` | Dramatic landings, weighty arrivals |
| Gentle loop | `TRANS_SINE` | `EASE_IN_OUT` | Breathing highlights, ambient pulses |
| Linear | `TRANS_LINEAR` | — | Spinners, continuous progress |

```gdscript
var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
t.tween_property(panel, "position:y", 0, 0.3).from(40)
t.parallel().tween_property(panel, "modulate:a", 1.0, 0.3).from(0.0)
```

## Duration by Element Weight

| Weight | Duration | Examples |
|---|---|---|
| **Lightweight** | 150ms | Icons, chips, tooltips, button hover |
| **Standard** | 300ms | Cards, panels, list items, menu tabs |
| **Weighty** | 500ms | Modals, pause menu open, map screen |
| **Cinematic** | 800-1500ms | Scene transitions, cutscene intros |

## Duration by Interaction

| Interaction | Duration | Notes |
|---|---|---|
| Button press feedback | 100-120ms | Quick punch; release tween 150-200ms |
| Hover enter | 120-150ms | Should feel responsive, not delayed |
| Focus change | 150-200ms | Border glow / slide |
| Tooltip appear | 200ms | Slight delay on entry (300-500ms) prevents nuisance pop |
| Tab switch | 200-300ms | Cross-fade content |
| Menu open/close | 300ms | Weight matters — heavier menus slower |
| Scene transition | 400-600ms | Paired with audio swell |
| Chapter/act transition | 800-1500ms | Rare; cinematic pause |

## Tween — The Workhorse

**When to use:** one-shot code-driven animation. Button press, panel fade-in, HP bar smoothing, damage flash, modal entry.

**Lifecycle:**
- `create_tween()` returns a new tween owned by the node it's called on
- Tween auto-frees when finished unless you `set_loops(N)` or attach
- Tween pauses automatically when the owning node is paused

**Common patterns:**

```gdscript
# Fade in a panel:
func fade_in() -> void:
    modulate.a = 0.0
    var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
    t.tween_property(self, "modulate:a", 1.0, 0.3)

# Slide + fade (parallel):
func slide_in_from_bottom() -> void:
    position.y = 40
    modulate.a = 0.0
    var t := create_tween().set_parallel(true).set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
    t.tween_property(self, "position:y", 0, 0.35)
    t.tween_property(self, "modulate:a", 1.0, 0.35)

# Sequential: fade out, then change content, then fade in
func crossfade_content(new_text: String) -> void:
    var t := create_tween()
    t.tween_property(label, "modulate:a", 0.0, 0.15)
    t.tween_callback(func(): label.text = new_text)
    t.tween_property(label, "modulate:a", 1.0, 0.15)

# Staggered list (like children appearing one after another):
func stagger_in(items: Array[Control]) -> void:
    for i in items.size():
        items[i].modulate.a = 0.0
        items[i].position.y = 20
        var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
        t.tween_interval(i * 0.05)  # 50ms stagger
        t.parallel().tween_property(items[i], "modulate:a", 1.0, 0.3)
        t.parallel().tween_property(items[i], "position:y", 0, 0.3)

# Looping breathing / pulse:
func start_pulse(control: Control) -> void:
    var t := create_tween().set_loops().set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)
    t.tween_property(control, "modulate", Color(1.2, 1.2, 1.2), 0.8)
    t.tween_property(control, "modulate", Color(1.0, 1.0, 1.0), 0.8)

# Stop + kill a tween you stored:
if active_pulse:
    active_pulse.kill()
    active_pulse = null
```

## AnimationPlayer — For Choreographed Sequences

**When to use:** multi-track keyframed animations authored in the editor. Menu open sequences, title screen reveals, complex HUD introductions, cutscene UI beats.

**Advantages over Tween:**
- Visual editor (timeline with keyframes)
- Multiple tracks animate together (position + rotation + audio + call methods)
- Reusable across scene instances
- Can trigger method calls at keyframe times (`@onready func _on_sting_played(): ...`)
- Can trigger audio
- Persists in scene file, no code needed

**Pattern:**

```gdscript
# Scene has an AnimationPlayer with animations "open", "close":
@onready var anim: AnimationPlayer = $AnimationPlayer

func open() -> void:
    show()
    anim.play("open")
    await anim.animation_finished
    default_button.grab_focus()

func close() -> void:
    anim.play("close")
    await anim.animation_finished
    hide()
```

Great for: title screen logo reveal + camera pan + audio cue + button fade-ins — one coordinated timeline.

## AnimationTree — For State Machines & Blending

**When to use:** UI elements that react to multiple game states with blending, or complex state-machine choreography.

**Examples:**
- Character portrait that blends between "idle / talking / angry / hurt" expressions based on dialogue state
- HUD compass rose that rotates + scales based on proximity to objective
- Boss health bar that transitions between phase visuals

**Pattern:** use an `AnimationNodeStateMachine` sub-resource; transition on conditions; blend trees for sliders (e.g., portrait morph 0.0 → 1.0 between two expressions).

Overkill for most UI — prefer Tween (one-shots) or AnimationPlayer (choreographed) unless you genuinely have state-machine-like UI logic.

## Decision Guide: Tween vs AnimationPlayer vs AnimationTree

| Use case | Pick |
|---|---|
| Button press, panel fade, simple code-driven motion | **Tween** |
| Menu open sequence with multiple elements, audio sync | **AnimationPlayer** |
| Title screen cinematic reveal | **AnimationPlayer** |
| HUD element reacting to single state change | **Tween** |
| Character portrait blending between emotions | **AnimationTree** |
| Boss health bar with phase transitions | **AnimationTree** |
| Quick prototyping, iteration | **Tween** (code) |
| Handing animation authoring to an animator | **AnimationPlayer** (editor) |

## Performance Rules

**Cheap to animate (do it freely):**
- `modulate` and `modulate:a` (color + alpha — GPU blend)
- `position` (`Control`, `Node2D`, `Node3D`)
- `rotation`
- `scale`
- `self_modulate` (affects only this node, not children)
- Shader parameters

**Expensive to animate (avoid per-frame):**
- `size` on a `Container` node — triggers full layout re-solve each frame
- `custom_minimum_size` on container children — same layout re-solve
- Theme property overrides — reloads theme
- Text content on a `Label` with autowrap — triggers text measurement

**If you must animate size:** animate the child's `scale` instead (keeps the container layout stable) and mask with a parent `Control` that has `clip_contents = true`.

**Respect paused state:** Tweens owned by a node inherit its `process_mode`. A pause-menu Tween should be on a node with `PROCESS_MODE_WHEN_PAUSED` so it actually runs when the game is paused.

## Reduced Motion

Respect the player's reduced-motion setting. Route decorative motion through a central singleton:

```gdscript
# motion_settings.gd — autoload
extends Node

var reduced_motion: bool = false
var screen_shake_enabled: bool = true
var flashing_reduced: bool = false

func shake(camera: Node2D, amplitude: float, duration: float) -> void:
    if reduced_motion or not screen_shake_enabled:
        return
    var start := camera.offset
    var t := create_tween()
    var steps := max(1, int(duration / 0.02))
    for i in steps:
        t.tween_property(camera, "offset", start + Vector2(
            randf_range(-amplitude, amplitude),
            randf_range(-amplitude, amplitude)
        ), 0.02)
    t.tween_property(camera, "offset", start, 0.1)

func maybe_flash(rect: ColorRect, alpha: float, duration: float) -> void:
    if flashing_reduced:
        alpha *= 0.4
        duration *= 1.5
    var t := create_tween()
    t.tween_property(rect, "modulate:a", alpha, duration * 0.3)
    t.tween_property(rect, "modulate:a", 0.0, duration * 0.7)
```

In menu transitions, offer an "instant" mode that short-circuits tweens:

```gdscript
func open_menu() -> void:
    show()
    if MotionSettings.reduced_motion:
        modulate.a = 1.0
        position.y = 0
        return
    # ... regular tweened entrance
```

## Game Feel / Juice Catalog

Feedback techniques to make UI and gameplay feel weighty and responsive. Pair multiple for maximum effect; gate all of them behind reduced-motion.

### Hit-stop (frame freeze on impact)

Freeze simulation 50-150ms on a hit to convey weight. Heavy hits longer, light hits shorter.

```gdscript
static func hit_stop(duration: float = 0.08) -> void:
    Engine.time_scale = 0.0
    await Engine.get_main_loop().create_timer(duration, true, false, true).timeout
    Engine.time_scale = 1.0
```

The `create_timer(duration, true, false, true)` flags: `ignore_time_scale = true` (so the timer itself runs despite `time_scale = 0`), `process_always = false`, `process_in_physics = true`.

### Screen shake

Random camera offset decaying to zero. Amplitude matches impact magnitude.

| Weight | Amplitude | Duration |
|---|---|---|
| Light hit | 2-4 pixels | 100-150ms |
| Medium hit | 6-10 pixels | 200-300ms |
| Heavy hit / explosion | 15-25 pixels | 400-600ms |

```gdscript
func shake(amplitude: float, duration: float) -> void:
    if MotionSettings.reduced_motion:
        return
    var origin := camera.offset
    var elapsed := 0.0
    while elapsed < duration:
        var decay := 1.0 - (elapsed / duration)
        camera.offset = origin + Vector2(
            randf_range(-amplitude, amplitude) * decay,
            randf_range(-amplitude, amplitude) * decay
        )
        await get_tree().process_frame
        elapsed += get_process_delta_time()
    camera.offset = origin
```

### Squash-stretch (UI click punch)

Scale 1.0 → 0.92 → 1.05 → 1.0 over 200ms. Makes buttons feel tactile.

```gdscript
func _on_pressed() -> void:
    var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_BACK)
    t.tween_property(self, "scale", Vector2(0.92, 0.92), 0.06)
    t.tween_property(self, "scale", Vector2(1.05, 1.05), 0.08)
    t.tween_property(self, "scale", Vector2(1.0, 1.0), 0.12)
```

Set `pivot_offset = size * 0.5` or the button will scale from the top-left corner.

### Chromatic aberration flash (damage)

RGB channel separation for a few frames on hit. Implement as a post-process shader on a full-screen `ColorRect` in a `CanvasLayer`. Ramp `aberration_strength` 0 → 0.02 → 0 over 200ms.

### Time dilation (slow-mo on kill)

Drop `Engine.time_scale` to 0.3 for 500ms, ease back to 1.0. Pair with audio low-pass filter.

### Camera punch (recoil)

Rotate camera 1-3° on shot, tween back over 150ms. Layer with screen shake for heavy weapons.

### Impact frames (full-screen flash)

`ColorRect` covering the screen, tween `modulate:a` 0 → 0.6 → 0 over 150ms. Color matches the impact (white for ice, red for fire).

### Particle feedback on UI events

Emit `GPUParticles2D` / `CPUParticles2D` at button click position. Coin bursts, sparks, confetti — scale with action importance.

### Freeze-frame on kill cam

Take a screenshot via `DisplayServer.screen_get_image()` or copy to a `TextureRect`. Display for 1-2s with audio sting. Release on input.

### Tween-sync audio

Every juicy visual gets an audio partner. Button squash → click sound. Menu open → woosh. Damage flash → low thud. Silence is the enemy of feel.

## Common Patterns

### Typewriter text reveal

```gdscript
@onready var body: RichTextLabel = $Body
var chars_per_sec: float = 45.0

func reveal(text: String) -> void:
    body.text = text
    body.visible_ratio = 0.0
    var duration := text.length() / chars_per_sec
    var t := create_tween()
    t.tween_property(body, "visible_ratio", 1.0, duration)
```

### Number counter (score tick-up)

```gdscript
func set_score(new_score: int, duration: float = 0.4) -> void:
    var old := current_score
    current_score = new_score
    var t := create_tween().set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_CUBIC)
    t.tween_method(_update_score_label, old, new_score, duration)

func _update_score_label(val: float) -> void:
    score_label.text = str(int(val))
```

### HP bar with damage lag (white ghost)

Common in fighting games: main bar tweens fast, a second bar lags behind showing damage taken.

```gdscript
func set_hp(new_hp: float) -> void:
    var t := create_tween()
    t.tween_property(front_bar, "value", new_hp, 0.1)  # fast
    t.parallel().tween_property(back_bar, "value", new_hp, 0.6).set_delay(0.15)  # lags
```

### Focus glow pulse

```gdscript
func _on_focus_entered() -> void:
    var t := create_tween().set_loops().set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)
    t.tween_property(focus_glow, "modulate:a", 1.0, 0.6)
    t.tween_property(focus_glow, "modulate:a", 0.4, 0.6)
    _focus_pulse = t

func _on_focus_exited() -> void:
    if _focus_pulse:
        _focus_pulse.kill()
    focus_glow.modulate.a = 0.0
```

## Resources

- [Godot Docs — Tween](https://docs.godotengine.org/en/stable/classes/class_tween.html)
- [Godot Docs — AnimationPlayer](https://docs.godotengine.org/en/stable/tutorials/animation/index.html)
- [Godot Docs — AnimationTree](https://docs.godotengine.org/en/stable/tutorials/animation/animation_tree.html)
- **Game Feel (Steve Swink)** — foundational book on juice and feedback
- **"Juice it or lose it"** — Martin Jonasson & Petri Purho GDC talk
- **"The art of screenshake"** — Jan Willem Nijman (Vlambeer) GDC talk
- [easings.net](https://easings.net/) — visual reference; map the curves to Godot's `TransitionType` enum
