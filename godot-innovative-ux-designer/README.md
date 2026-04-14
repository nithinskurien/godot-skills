# Godot Innovative UX Designer

A Claude Code skill that turns Claude into a senior game UI/UX designer for **Godot 4.x**. It produces distinctive, production-grade game interfaces — menus, HUDs, dialogue, inventories, pause/settings screens — while enforcing accessibility (remappable input, gamepad focus, colorblind-safe palettes, subtitles, reduced motion) and resolution scaling across PC, Steam Deck, console, and mobile.

No generic "engine default theme" slop. Every screen should feel intentionally crafted for *your* game.

## What you get

When the skill activates, Claude:

1. **Asks first** — clarifies the game's fiction, platform targets, tone, and one memorable visual hook before writing any scene
2. **Commits boldly** — picks one aesthetic direction (brutalist, retro-futuristic, cozy, editorial, etc.) and executes it with precision
3. **Writes real Godot 4.x code** — GDScript with proper `Control` / `Container` node hierarchies, `Theme` resources with `StyleBoxFlat` variations, `Tween` and `AnimationPlayer` for motion, signals wired correctly
4. **Bakes in accessibility** — gamepad focus navigation, remappable `InputMap`, `AccessibilityServer` (4.4+) exposure, colorblind modes, subtitles with speaker labels, reduced-motion toggle, text scale slider
5. **Scales across resolutions** — stretch mode, anchor presets, container discipline, safe-area insets for console overscan, Steam Deck sizing

## When it triggers

The skill activates automatically when you ask Claude to work on Godot UI. Examples:

- "Design a pause menu for my roguelike"
- "Build a HUD with a health bar, stamina, and minimap"
- "Create a dialogue system with speaker labels and typewriter text"
- "Make the main menu feel more editorial"
- "Style this settings screen — it looks like default Godot"
- Editing any `.tscn` or `.gd` file involving `Control` nodes

It will also activate implicitly when your working directory contains Godot project files and you discuss UI work.

## Installation

The skill is structured as a Claude Code plugin bundle:

```
godot-innovative-ux-designer/
├── README.md                    # ← you are here
└── skills/
    └── godot-innovative-ux-designer/
        ├── SKILL.md             # Main handbook (Claude reads this first)
        ├── ACCESSIBILITY.md     # Gamepad nav, TTS, colorblind, subtitles
        ├── DESIGN-SYSTEM-TEMPLATE.md  # Fixed / project / adaptable framework
        ├── MOTION-SPEC.md       # Tween / AnimationPlayer / game-feel catalog
        └── RESOLUTION-SCALING.md  # Stretch mode, anchors, containers, targets
```

### Option A — Install as a user-level skill

Copy (or symlink) the inner skill folder into Claude Code's skills directory:

**Windows (PowerShell):**
```powershell
Copy-Item -Recurse `
  ".\godot-innovative-ux-designer\skills\godot-innovative-ux-designer" `
  "$env:USERPROFILE\.claude\skills\"
```

**macOS / Linux:**
```bash
cp -r ./godot-innovative-ux-designer/skills/godot-innovative-ux-designer \
      ~/.claude/skills/
```

Claude Code picks up new skills automatically on the next prompt. No restart needed.

### Option B — Install per-project

Drop the inner skill folder into your Godot project's `.claude/skills/` directory:

```
your-godot-project/
├── .claude/
│   └── skills/
│       └── godot-innovative-ux-designer/
│           └── SKILL.md ...
├── project.godot
└── ...
```

Project-level skills apply only when Claude Code runs in that directory. Useful when different games want different UX rules.

### Option C — Load as a Claude Code plugin

If you manage skills via Claude Code's plugin system, this bundle's top-level layout (`skills/<skill-name>/SKILL.md`) is already plugin-compatible. Reference it from your plugin configuration.

### Verify the install

Ask Claude: *"What skills are available? Do you have a Godot UX skill?"* Claude will list it if discovered. You can also ask *"What does the godot-innovative-ux-designer skill cover?"* to get a summary before using it.

## How to use it

Once installed, just describe what you want. The skill handles the rest.

### Example prompts

```
Build a main menu for a cozy farming sim. Gamepad-first, warm palette,
hand-drawn serif display font. Include a continue button that grays out
when no save exists.
```

```
I need a pause menu that works during game pause. Keep the world blurred
behind it. Accessibility: reduced-motion option should skip the fade.
```

```
Design a HUD: health bar top-left, ammo counter bottom-right, objective
tracker top-right. Must work at 1280x800 Steam Deck and 3440x1440 ultrawide.
Use MSDF fonts.
```

```
Build an inventory with drag-and-drop slots, tooltips on focus, and a
keyboard-navigable grid. 6x4 layout.
```

```
Create a dialogue box with typewriter reveal, speaker labels, and ambient
sound captions. Brutalist horror tone.
```

```
Audit this menu scene for accessibility — gamepad focus chain, contrast,
color-only state encoding.
```

### What Claude will ask you first

Before producing code, Claude typically asks:

1. **Fiction & tone** — what world, what feel?
2. **Platform** — PC kb+m, gamepad-first, Steam Deck, mobile, or all?
3. **Resolution targets** — which sizes must this hold up at?
4. **Signature detail** — the one thing someone screenshots?

Answer even briefly ("retro-futuristic, Steam Deck first, 1280×800 and 1920×1080") and Claude will proceed. If you skip answering, Claude will commit to a reasonable default and tell you what it chose so you can redirect.

## What's in each file

| File | When Claude reads it |
|---|---|
| `SKILL.md` | Always. Main handbook: design philosophy, Godot UI architecture, component library, game feel |
| `ACCESSIBILITY.md` | When accessibility is discussed: gamepad nav, TTS, colorblind, subtitles, remappable input |
| `DESIGN-SYSTEM-TEMPLATE.md` | When scoping a new project or setting up a Theme resource: fixed/project-specific/adaptable framework |
| `MOTION-SPEC.md` | When animating UI or building game feel: Tween/AnimationPlayer/AnimationTree patterns, juice catalog |
| `RESOLUTION-SCALING.md` | When designing for multiple resolutions: stretch mode, anchors, containers, safe-area |

The files use Claude's progressive-disclosure pattern: `SKILL.md` is read on every activation; the others are consulted when the relevant topic comes up.

## Target Godot version

- **Baseline**: Godot **4.4+** (stable `AccessibilityServer` + screen reader exposure, improved Theme system)
- **Notes called out**: features requiring 4.5/4.6 are flagged where relevant
- **Degrades gracefully**: 4.3 and earlier lose the AccessibilityServer; everything else (Theme, Tween, AnimationPlayer, Control nodes, focus system) works identically back to 4.2

Code examples are GDScript only. C# users will recognize the APIs — the concepts map 1:1.

## Key Principles

### 1. Design Thinking Protocol
Ask first. Commit boldly. No half-measures.

### 2. Anti-Generic Aesthetics
**Avoid**: Godot's default Theme, Noto Sans, 50% grey panels, SaaS blue (#3B82F6), generic health-bar red, glass morphism.

**Embrace**: Custom MSDF fonts, hand-tuned `StyleBoxFlat`, palettes that match fiction, intentional diegetic/non-diegetic choices.

### 3. Game UI is Not Web UI
`Control` nodes, `Theme` resources, `StyleBoxFlat`, Container discipline, `CanvasLayer` ordering, signal-driven interaction. Gamepad-first focus nav; mouse/touch are additions, not assumptions.

### 4. Accessibility Baseline
Remappable input, gamepad focus chains, colorblind-safe state encoding, subtitles with speaker labels, reduced-motion toggle, text scale slider, photosensitivity safeguards.

## Tone Options

Pick an extreme and commit. Genre hints are suggestions, not rules:

- **Brutally minimal** — stripped to essence *(roguelikes, puzzle games, hardcore sims)*
- **Maximalist chaos** — layered, dense, controlled disorder *(shmups, JRPGs, arcade)*
- **Retro-futuristic** — vintage meets sci-fi *(synthwave racers, immersive sims, cyber-noir)*
- **Organic/natural** — earth tones, flowing shapes *(cozy sims, farming, exploration)*
- **Editorial/magazine** — grid-based, typographic *(narrative games, visual novels, detective)*
- **Brutalist/raw** — exposed structure, monospace *(horror, survival, extraction shooters)*
- **Luxury/refined**, **Playful/toy-like**, **Art deco/geometric**, **Industrial/utilitarian**, **Hand-drawn/zine**, **Diegetic/in-world** — see `SKILL.md` for the full list

## Credits

This skill is a Godot 4.x port of the original **`bencium-innovative-ux-designer`** skill, a web-focused UX designer skill built for React / Tailwind / shadcn / Framer Motion stacks. The foundational design philosophy — Design Thinking Protocol, anti-generic aesthetics, bold creative commitment, the fixed/project-specific/adaptable framework, the six tone options, and the accessibility-as-baseline ethos — all originate from that skill.

The port replaces web implementation details with their Godot equivalents:

| Original (web) | Port (Godot 4.x) |
|---|---|
| Tailwind / CSS / shadcn | `Theme` resources + `StyleBoxFlat` / `StyleBoxTexture` + `Control` node tree |
| React / JSX / TSX | `.tscn` scenes + GDScript + signals |
| Framer Motion | `Tween` + `AnimationPlayer` + `AnimationTree` |
| ARIA + `sr-only` + screen readers | `focus_neighbor_*` + `AccessibilityServer` (4.4+) + TTS via `DisplayServer` |
| Tailwind breakpoints / media queries | Project stretch mode + anchor presets + Container nodes |
| Web fonts (`@font-face`) | `FontFile` resources with MSDF enabled |
| Phosphor icons / Lucide | SVG `Texture2D` (Scalable import) + `AtlasTexture` |

New material added for games: diegetic/non-diegetic/spatial/meta UI categories, the game-feel catalog (hit-stop, screen shake, squash-stretch, chromatic aberration, time dilation, camera punch, freeze-frame), gamepad focus wiring, safe-area/overscan for console, Steam Deck / ultrawide / mobile resolution targets, colorblind mode toggles, subtitle/caption conventions, and TTS integration.

**Credit to the original `bencium-innovative-ux-designer`** for the design philosophy this port stands on. Bugs, oversimplifications, or Godot-specific misunderstandings in this port are not the original author's responsibility.

## External references

- [Godot Docs — UI Introduction](https://docs.godotengine.org/en/stable/tutorials/ui/index.html)
- [Godot Docs — Multiple Resolutions](https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html)
- [Godot Docs — AccessibilityServer](https://docs.godotengine.org/en/stable/classes/class_accessibilityserver.html)
- [Game Accessibility Guidelines](https://gameaccessibilityguidelines.com/)
- [AbleGamers APX](https://accessible.games/accessible-player-experiences/)

## Version History

- **v3.0.0** (2026-04-14) — Godot 4.x port
  - Folder renamed to `godot-innovative-ux-designer`
  - Tailwind/shadcn/React replaced with `Theme`/`StyleBoxFlat`/`Control` nodes
  - Framer Motion replaced with `Tween` / `AnimationPlayer` / `AnimationTree`
  - ARIA/`sr-only` replaced with Godot focus system + `AccessibilityServer` (4.4+) + TTS
  - `RESPONSIVE-DESIGN.md` → `RESOLUTION-SCALING.md` (stretch mode, Steam Deck, ultrawide, console safe zones)
  - Added game UI categories (diegetic / non-diegetic / spatial / meta)
  - Added game-feel catalog (hit-stop, screen shake, squash-stretch, chromatic aberration, time dilation, camera punch)
  - Added game-specific accessibility: remappable input, colorblind modes, subtitles, photosensitivity
- **v2.0.0** (2025-11-22, `bencium-innovative-ux-designer`) — Creative liberation update (pre-port)
- **v1.0.0** (2025-10-18, `bencium-innovative-ux-designer`) — Initial release

## License

Match the license of the original `bencium-innovative-ux-designer` skill. If you redistribute this port, retain the credits section.
