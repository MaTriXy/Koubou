---
name: koubou
description: Generate App Store screenshots using HTML/CSS templates with real device frames. Creates professional, localized screenshots for iPhone, iPad, Mac, and Watch. Use when user wants to create, design, or update App Store screenshots.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(kou *)
  - Bash(python *)
  - Bash(python3 *)
  - Bash(open *)
  - Bash(mkdir *)
  - Bash(ls *)
  - Bash(cp *)
user-invocable: true
---

# Koubou: App Store Screenshot Generator

Generate professional App Store screenshots using HTML/CSS templates with 100+ real device frames, xcstrings localization, and pixel-perfect Apple dimensions.

> Screenshots are advertisements, not documentation. Each slide sells one feeling, outcome, or pain-point solution.

**User instructions always take priority over defaults in this skill.**

**Creative direction is open by default.** Use the rules in this skill to enforce impact, readability, and canvas-aware scale, not to make every set look the same.

## Rule Hierarchy

Treat the guidance in this skill in this order:

1. **Hard constraints** — readability, scale, thumbnail clarity, truthful product marketing, and real canvas awareness
2. **Strong defaults** — variety, asymmetry, overlap, rhythm, and avoiding generic layouts
3. **Taste preferences** — anti-slop heuristics and stylistic defaults that can be broken when a stronger idea clearly improves the work

If breaking a default produces a better screenshot set, break it deliberately and explain the tradeoff to yourself before generating.

Reference files (read as needed, not upfront):
- `setup.md` — Installation and HTML runtime setup
- `design-guide.md` — Design principles, copywriting rules, CSS rules, HTML template examples
- `yaml-reference.md` — YAML config format, localization, assets, devices, sizes
- `capabilities-reference.md` — Full koubou capabilities (content mode, highlights, zoom, gradients)

## Phase 1: Setup (silent, automatic)

Check if `kou` is available first. Do not reinstall or touch Python packaging if `kou` already works.

Preferred order:

```bash
kou --version 2>/dev/null
```

If `kou --version` succeeds:
- treat koubou as installed
- do **not** run `pip`, `pip3`, or `playwright install`
- do **not** try to "upgrade" or "fix" the user's environment proactively
- always run `kou setup-html` before generating, because this skill is HTML-only and the command is designed to be safe to repeat

If `kou --version` fails, read `setup.md` and follow the least invasive path for the environment.

For HTML rendering support:
- always use `kou setup-html`
- do **not** skip it just because HTML worked in a previous run
- do **not** run `playwright install chromium` directly

Only inform the user if setup fails. Never mutate a working installation.

## Phase 2: Gather Information

Collect what you need through natural conversation. Do NOT dump all questions at once.

### Required (ask in order, naturally)

1. **App**: Name, what it does (1 sentence), main value proposition
2. **Screenshots**: Where are the app captures? Search automatically first:
   - Glob for: `.maestro/`, `screenshots/`, `AppStore/`, `fastlane/screenshots/`, `marketing/`
   - If found, show what you found and confirm
   - If not found, ask how the user generates them
   - Prefer clean simulator captures that show the app UI clearly. If the user can choose, prefer recent 6.1-inch iPhone captures as the source material
3. **Visual style**: Brand colors, preferred font, dark/light/colorful preference, and App Store references if any
   - If `Assets.xcassets`, `marketing/plan.md`, prior marketing, or brand assets exist, derive colors/font direction from there without asking
   - If the user gives directional feedback like "more premium", "more bold", "more editorial", "more airy", or "more dense", treat that as a real design constraint
4. **Features to highlight**: Prioritized list of features/benefits (recommend 3-5). Each slide = 1 feature
5. **Hero assets**: App icon or other brand asset. Search automatically before asking
6. **Slide count**: How many screenshots (recommend 3-5 to start, Apple allows up to 10)

### Optional (ask only if relevant)

7. **Device**: Default is `iPhone 16 Pro - Black Titanium - Portrait`. Only ask if iPad/Mac/other makes sense
8. **Localization**: Only if the project already has xcstrings or multiple language support
9. **Extra assets**: Floating UI elements, badges, supporting illustrations
10. **Additional constraints**: Any required claims, forbidden styles, or marketing constraints

### Derived (do NOT ask — figure out automatically)

- Background gradients (from brand colors)
- Layout distribution (hero → feature-top → feature-bottom → alternating)
- Headline copy (from features and value proposition)
- Secondary palette (variations of brand colors)
- Canvas class from `project.device` + `project.output_size` via `kou inspect-frame`
- Whether the app name should appear in the slide at all. Default: omit it unless it adds real brand value at readable size
- Whether the app icon should appear. Default: use it sparingly on hero or closing slides, not as a tiny decorative marker

**Principle**: If context is available (CLAUDE.md, existing configs, marketing/plan.md, Assets.xcassets), use it without re-asking.

## Phase 3: Generate

Read `design-guide.md` before generating templates. Read `yaml-reference.md` before writing config.

1. Create working directory (e.g., `AppStore/` or wherever makes sense for the project)
2. Run `kou setup-html`
3. Read `project.device` and `project.output_size` from the YAML
4. Run `kou inspect-frame "<device>" --output-size <size> --output json` and use that geometry before writing CSS
5. Draft the narrative arc and 2-3 headline/subtitle options per slide before touching layout. Pick the strongest one first. Copy quality comes before CSS
6. Create `templates/` with HTML templates — **minimum 3 distinct layouts** (read `design-guide.md` for templates)
7. Create `config.yaml` with koubou config (read `yaml-reference.md` for format)
8. Use CSS that adapts to the canvas and copy length: use `vw` or `clamp()` deliberately, prefer CSS Grid, overlap, and absolute-positioned layers, and verify the final computed text scale on the real canvas
9. **Before writing HTML**: plan text/device zones for each slide (which area owns what percentage of the canvas). Do not start CSS until zones are clear
10. Run: `kou generate config.yaml --verbose`
11. **Post-render QA** (mandatory — do not skip):
    - Review each generated slide visually
    - Check against the rejection checklist in `design-guide.md`
    - Verify: no emoji icons, no identical card grids, no decentered devices, no text below minimum scale, no unintentional device cropping (device top/notch must be visible on hero slides; screen content must be readable)
    - Apply the logo-swap test: would these slides work for a competitor?
    - If any slide fails, fix and regenerate before showing to the user
12. Open output folder: `open <output_dir>`
13. Ask if the user wants adjustments — iterate on specific slides without regenerating everything

### Iteration Rules

- When user asks to change a specific slide, only modify that template + config entry
- When user asks for a global style change (colors, fonts, mood, density, boldness), update all templates
- Re-run `kou generate config.yaml --verbose` after changes
- Use `kou live config.yaml` if user wants real-time preview while editing
- If a slide feels small, first increase scale or switch layout. Do not hide the problem with extra gradients or labels
- If copy forces tiny text, rewrite the copy or choose a more suitable layout; never accept a timid slide because "the text had to fit"
- Keep variety high: the goal is campaign consistency, not layout repetition
- Do not stop after the first successful render if the output still violates the design rules or rejection checklist
- If the user asks for a mood change, reinterpret the whole set inside the quality limits instead of defending the previous defaults

## Hard Rules

### Hard constraints
- **Never use emoji as icons** — use CSS shapes (accent bars, dots) or text-only
- **Never use banned copy phrases** — "revolutionary", "seamless", "unlock", "game-changing", etc.
- **Apply the logo-swap test** — if a competitor's name would fit, the set is too generic

### Scale
- On tall iPhone portrait (`iPhone6_9`, `iPhone6_7`), hero headlines must be `>=10vw`, side-layout headlines `>=10vw`, subtitles `>=4.5vw`
- Use `vw` or `clamp()` only if the computed size still clears the minimum on the target canvas
- On contrast or closing slides, the content cluster must occupy 60%+ of canvas height

### Strong defaults
- Do not use the same upright, centered phone composition on consecutive slides
- Do not leave large empty bands (>15% canvas height) between headline and device
- Avoid combining `rotate()` + `translate()` on centered devices unless the visual center still feels intentional after render
- Do not use a simple centered flex column when the slide needs overlap, edge-breaking crops, or layered composition
- Plan text/device zones before writing CSS (see design-guide.md Device-Text Zone Planning)
- If you use card grids, stagger, emphasize, or simplify them unless a strict grid genuinely serves the concept

### Content
- Do not add tiny top labels, category labels, or app-name headers by default
- Do not render the app name as small decorative text unless clearly readable and strategic
- Do not use app icons as tiny corner decorations; if used, they should be a real brand element
- Do not use awkward, literal, or unnatural headlines just to be short
- Do not start layout work before the slide narrative and copy are coherent

### Process
- If the first 3 slides do not already feel App Store-ready, keep iterating before presenting them
- Do not present the first technically successful render as final — always review against the rejection checklist in design-guide.md
- Do not let the rules flatten the creative direction. A strange but strong composition is valid if it stays readable and high-impact
- Defaults can be broken when the final result is stronger, clearer, and more specific to the app

## Key Technical Details

### How assets work in HTML templates

In template HTML, reference assets with `{{asset_name}}`. In the YAML config, map `asset_name` to a file path under `assets:`.

Koubou automatically pre-renders each image asset with the configured device frame before passing it to the HTML template. The template receives a composited image (screenshot inside device frame) — it just places it with `<img src="{{asset_name}}">`.

To disable frame for a specific screenshot: set `frame: false` in its YAML definition.

### Template variable substitution

- `variables:` in YAML → `{{key}}` in HTML → localizable text (extracted to xcstrings)
- `assets:` in YAML → `{{key}}` in HTML → image file paths (pre-rendered with device frame)

### Output structure

```
{output_dir}/{language}/{device_name}/{screenshot_id}.png
```

Non-localized projects skip the language directory.

### Device frame names

Use exact names from `kou list-frames`. Common ones:
- `iPhone 16 Pro - Black Titanium - Portrait`
- `iPhone 16 Pro Max - Black Titanium - Portrait`
- `iPad Pro 13 - M4 - Space Gray - Portrait`

Search with: `kou list-frames "iPhone 16"`

### Frame inspection for layout decisions

Use `kou inspect-frame "<device>" --output-size <size> --output json` to get:
- real frame size
- screen bounds and screen bbox
- safe margins
- coverage ratio
- orientation
- `canvas_class`

Use this data to choose typography scale, crop aggressiveness, and layout density.

### Output sizes (App Store dimensions)

| Name | Dimensions | Devices |
|------|-----------|---------|
| `iPhone6_9` | 1320x2868 | iPhone 16 Pro Max, 15 Pro Max |
| `iPhone6_7` | 1290x2796 | iPhone 15/14/13 Pro Max, Plus |
| `iPhone6_5` | 1242x2688 | iPhone 11 Pro Max, XS Max |
| `iPhone6_1` | 1179x2556 | iPhone 16/15/14/13 Pro |
| `iPhone5_5` | 1242x2208 | iPhone 8 Plus, 7 Plus |
| `iPadPro13` | 2064x2752 | iPad Pro 13" M4 |
| `iPadPro12_9` | 2048x2732 | iPad Pro 12.9" |
| `iPadPro11` | 1668x2388 | iPad Pro 11" |

Custom: `output_size: [1320, 2868]`
