# TACTICS — Visual Design Document

**Version:** 1.0  
**Date:** 2026-04-23  
**Status:** Source of Truth for Visual Implementation

---

## 1. Color Palettes

Three fully-specced palettes. All values are exact. CSS custom properties are listed at the end of each palette block — these drive the entire UI via variables.

---

### Palette A: Phosphor Green (Default)

The canonical green-screen CRT terminal. Think 1981 IBM 3101, DECwriter III.

| Token | Value | Notes |
|---|---|---|
| Background | `#0a0f0a` | Near-black with the faintest green tint |
| Gear fill | `#39ff14` | Neon green, slightly yellow-shifted for warmth |
| Gear stroke / outline | `#1a7a08` | Dark forest green, 1–2px outline |
| Glow color | `rgba(0, 255, 65, 0.4)` | Used in drop-shadow filter |
| Glow color (solid) | `#00ff41` | For text-shadow and border glow |
| Accent color | `#7fff00` | Chartreuse — used for header border, accent elements |
| Pawl — idle | `rgba(57, 255, 20, 0.4)` | Gear fill at 40% opacity |
| Pawl — active | `#ffffff` | Pure white flash on click |
| Text color | `#39ff14` | Matches gear fill |
| Header border | `#1a7a08` | Dark green, 1px solid |
| Scanlines overlay | `rgba(0, 0, 0, 0.15)` | 2px dark lines every 4px |
| Vignette | `rgba(0, 0, 0, 0.7)` | Radial gradient, transparent center → dark edge |
| Hub fill | `#2acc10` | Slightly darker/cooler than gear, distinct but same family |
| Center hole | `#0a0f0a` | Matches background — creates punch-through effect |
| Noise overlay | `rgba(0, 255, 65, 0.03)` | Extremely subtle green noise tint |

```css
:root[data-palette="green"] {
  --bg:           #0a0f0a;
  --gear-fill:    #39ff14;
  --gear-stroke:  #1a7a08;
  --gear-hub:     #2acc10;
  --glow-rgba:    rgba(0, 255, 65, 0.4);
  --glow-solid:   #00ff41;
  --accent:       #7fff00;
  --pawl-idle:    rgba(57, 255, 20, 0.4);
  --pawl-active:  #ffffff;
  --text:         #39ff14;
  --header-border:#1a7a08;
  --scanline:     rgba(0, 0, 0, 0.15);
  --vignette:     rgba(0, 0, 0, 0.7);
}
```

---

### Palette B: Amber Terminal

Warm amber phosphor. Think HP 2640A, Televideo 912. Evokes late-70s word processors.

| Token | Value | Notes |
|---|---|---|
| Background | `#0f0a00` | Near-black with warm amber tint |
| Gear fill | `#ffb300` | Saturated amber, slightly orange |
| Gear stroke / outline | `#7a4400` | Dark burnt sienna, 1–2px outline |
| Glow color | `rgba(255, 140, 0, 0.4)` | Orange-amber for drop-shadow |
| Glow color (solid) | `#ff8c00` | Dark orange, for text-shadow |
| Accent color | `#ffd700` | Gold — brighter than gear fill, used for highlights |
| Pawl — idle | `rgba(255, 179, 0, 0.4)` | Gear fill at 40% opacity |
| Pawl — active | `#ffffff` | Pure white flash |
| Text color | `#ffb300` | Matches gear fill |
| Header border | `#7a4400` | Dark burnt sienna |
| Scanlines overlay | `rgba(0, 0, 0, 0.18)` | Slightly denser than green — amber screens felt murkier |
| Vignette | `rgba(0, 0, 0, 0.72)` | Very slightly warmer dark |
| Hub fill | `#e69900` | Darker amber, noticeably distinct from teeth |
| Center hole | `#0f0a00` | Matches background |
| Noise overlay | `rgba(255, 140, 0, 0.025)` | Barely-there amber grain |

```css
:root[data-palette="amber"] {
  --bg:           #0f0a00;
  --gear-fill:    #ffb300;
  --gear-stroke:  #7a4400;
  --gear-hub:     #e69900;
  --glow-rgba:    rgba(255, 140, 0, 0.4);
  --glow-solid:   #ff8c00;
  --accent:       #ffd700;
  --pawl-idle:    rgba(255, 179, 0, 0.4);
  --pawl-active:  #ffffff;
  --text:         #ffb300;
  --header-border:#7a4400;
  --scanline:     rgba(0, 0, 0, 0.18);
  --vignette:     rgba(0, 0, 0, 0.72);
}
```

---

### Palette C: Neon Cyber

Cool cyan primary with magenta secondary. Think 1990s arcade cabinet, Tron, Cyberpunk 2020 sourcebook.

| Token | Value | Notes |
|---|---|---|
| Background | `#050010` | Near-black with deep violet tint |
| Gear fill | `#00d4ff` | Electric cyan |
| Gear stroke / outline | `#004466` | Dark teal, 1–2px outline |
| Glow color | `rgba(0, 212, 255, 0.35)` | Cyan glow, slightly less intense than green (cyan reads stronger) |
| Glow color (solid) | `#00d4ff` | Cyan, for text-shadow |
| Accent color | `#ff00ff` | Magenta — high contrast accent for header, pawl |
| Pawl — idle | `rgba(0, 212, 255, 0.4)` | Cyan at 40% |
| Pawl — active | `#ff00ff` | Magenta flash instead of white — more on-brand |
| Text color | `#00d4ff` | Cyan, matches gear |
| Header border | `#bf00ff` | Purple — bridges cyan and magenta |
| Scanlines overlay | `rgba(0, 0, 0, 0.12)` | Lighter scanlines — neon screens feel brighter |
| Vignette | `rgba(0, 0, 20, 0.75)` | Deep blue-black edges |
| Hub fill | `#008fb3` | Darker, desaturated cyan |
| Center hole | `#050010` | Matches background |
| Noise overlay | `rgba(191, 0, 255, 0.02)` | Barely-perceptible purple grain |

```css
:root[data-palette="cyber"] {
  --bg:           #050010;
  --gear-fill:    #00d4ff;
  --gear-stroke:  #004466;
  --gear-hub:     #008fb3;
  --glow-rgba:    rgba(0, 212, 255, 0.35);
  --glow-solid:   #00d4ff;
  --accent:       #ff00ff;
  --pawl-idle:    rgba(0, 212, 255, 0.4);
  --pawl-active:  #ff00ff;
  --text:         #00d4ff;
  --header-border:#bf00ff;
  --scanline:     rgba(0, 0, 0, 0.12);
  --vignette:     rgba(0, 0, 20, 0.75);
}
```

---

## 2. Gear Geometry Spec

### Canvas Setup

The gear renders on a `<canvas>` element using a **low-resolution internal canvas** scaled up via CSS to create natural pixel-art chunking. This is the spec's mandated approach (§3).

```
gearDiameterPx  = Math.min(window.innerWidth, window.innerHeight) * 0.80
internalSize    = Math.round(gearDiameterPx / 4)   // ~quarter resolution
canvas.width    = internalSize
canvas.height   = internalSize
canvas.style.width  = gearDiameterPx + 'px'
canvas.style.height = gearDiameterPx + 'px'
ctx.imageSmoothingEnabled = false
```

On HiDPI displays, do **not** multiply by `devicePixelRatio` — the pixel-art look is intentional and the low-res render achieves it. The CSS size stays at logical `gearDiameterPx`.

### Coordinate System

All measurements below are in **internal canvas pixels** (the low-res coordinate space). At a typical 800px logical screen, the internal canvas is ~160×160px.

```
internalCenter.x = internalSize / 2     // e.g. 80
internalCenter.y = internalSize / 2     // e.g. 80
```

### Gear Radii

| Component | Internal px (at internalSize=160) | As fraction of internalSize/2 |
|---|---|---|
| Tooth tip (outer radius) | 72px | 0.90 of half-size |
| Tooth root (inner radius) | 64px | 0.80 of half-size |
| Hub outer radius | 20px | 0.25 of half-size |
| Center hole radius | 7px | 0.09 of half-size |

Express as fractions so they scale automatically with canvas size:

```js
const R      = internalSize * 0.45    // tooth tip radius
const Rroot  = internalSize * 0.40    // tooth root radius
const Rhub   = internalSize * 0.125   // hub outer radius
const Rhole  = internalSize * 0.044   // center hole radius
```

### Tooth Geometry — 16 Teeth

**Count:** 16 teeth, evenly spaced → 22.5° (π/8 radians) per tooth.

**Duty cycle:** 50% — tooth width equals gap width. Each tooth occupies 11.25° (π/16 radians), each gap occupies 11.25°.

**Tooth profile:** Square-topped rectangle. No curves, no chamfers. Pixel art — all vertices must be integer coordinates after rounding.

```
Per tooth angular slot: π/8 radians (22.5°)
Tooth arc: π/16 radians (11.25°) — the tooth itself
Gap arc:   π/16 radians (11.25°) — the valley between teeth

Each tooth has 4 corners:
  - inner-left:  (Rroot × cos(θ_start), Rroot × sin(θ_start))
  - outer-left:  (R     × cos(θ_start), R     × sin(θ_start))
  - outer-right: (R     × cos(θ_end),   R     × sin(θ_end))
  - inner-right: (Rroot × cos(θ_end),   Rroot × sin(θ_end))

where θ_start = i × (π/8)
      θ_end   = θ_start + (π/16)
```

The gap (valley) connects adjacent inner-right and inner-left points at radius `Rroot`.

### Hub Design

- **Outer ring:** filled circle at `Rhub`, color `var(--gear-hub)` — slightly darker/cooler than gear fill
- **Center hole:** filled circle at `Rhole`, color = background (`var(--bg)`) — creates a punched-through look
- **Crosshair:** 2px lines (1px wide at low-res), horizontal and vertical, extending 4px from center in each direction, color = background. Sits on top of hub fill, below nothing else.

```
Crosshair lines (internal coords, centered at 0,0 after translate):
  Horizontal: (-4, 0) → (+4, 0)
  Vertical:   (0, -4) → (0, +4)
  strokeStyle = var(--bg)
  lineWidth = 1
```

### Stroke / Outline

- `strokeStyle = var(--gear-stroke)` (dark variant of gear color)
- `lineWidth = 1` at internal resolution (scales up to ~4px at CSS size — chunky, correct)
- Apply `ctx.stroke()` after the full gear path is closed
- Stroke the hub circle separately with same stroke settings
- Do **not** stroke the center hole — let it read as a clean void

---

## 3. Pawl / Reference Indicator Design

### What It Is

A fixed HTML element (not canvas) positioned at the 12 o'clock position above the gear. It represents the ratchet pawl — the static tooth the gear clicks against.

### Position

```
pawl.left = center.x - 4px          // horizontally centered on gear
pawl.top  = center.y - gearRadius - 20px   // 20px above top of gear
```

Gear center in viewport: `50vw`, `50vh` (gear is centered via flexbox). Gear radius = `gearDiameterPx / 2`. Use CSS to position pawl absolute relative to a centered wrapper.

### Shape

Downward-pointing triangle, 8px wide × 6px tall. Pixel art — rendered as a CSS clip-path or a single character (▼) or a small inline SVG.

**Recommended approach — inline SVG (most control):**

```html
<svg class="pawl" width="12" height="10" viewBox="0 0 12 10">
  <!-- Pixel-art triangle: 3 points, no anti-aliasing wanted -->
  <!-- shape-rendering: crispEdges -->
  <polygon points="0,0 12,0 6,10" 
           fill="currentColor" 
           shape-rendering="crispEdges"/>
</svg>
```

At actual render size (12×10px SVG), this is 12 pixels wide at base, 10px tall. Looks like a chunky downward arrow.

Alternative: use Unicode `▼` at 14px `Press Start 2P` — it will render as a pixel triangle due to the font's bitmap nature.

### States

| State | Color | Transform | Transition |
|---|---|---|---|
| Idle | `var(--pawl-idle)` (gear fill @ 40% opacity) | `translateX(0) translateY(0)` | None (instant) |
| Active (click) | `#ffffff` (or `--pawl-active` for Cyber palette) | `translateX(var(--snap-x)) translateY(4px)` | `none` — instant snap |
| Return | `var(--accent)` | `translateX(0) translateY(0)` | `80ms cubic-bezier(0.2, 0, 0.8, 1)` |

`--snap-x`: set to `+4px` for CW rotation, `-4px` for CCW rotation. Set via JS on each click.

### Snap Animation Implementation

```css
.pawl {
  color: var(--pawl-idle);
  transform: translateX(0) translateY(0);
  transition: transform 80ms cubic-bezier(0.2, 0, 0.8, 1), 
              color 80ms ease;
}

.pawl--clicked {
  color: var(--pawl-active);
  transform: translateX(var(--snap-x, 0px)) translateY(4px);
  transition: none;  /* instant snap */
}
```

```js
function animatePawl(direction) {
  pawlEl.style.setProperty('--snap-x', direction > 0 ? '4px' : '-4px')
  pawlEl.classList.add('pawl--clicked')
  setTimeout(() => {
    pawlEl.classList.remove('pawl--clicked')
    // CSS transition takes over for the return journey
  }, 30)  // color stays white 30ms, then transitions back over remaining 50ms
}
```

---

## 4. CRT Effects CSS Spec

All overlay elements use `position: fixed; inset: 0; pointer-events: none` so they cover the viewport and never intercept touch input.

### 4.1 Scanlines

2px dark horizontal bands every 4px, creating a 50% scanline pattern.

```css
.scanlines {
  position: fixed;
  inset: 0;
  background: repeating-linear-gradient(
    to bottom,
    transparent 0px,
    transparent 2px,
    var(--scanline) 2px,
    var(--scanline) 4px
  );
  pointer-events: none;
  z-index: 10;
  mix-blend-mode: multiply;
}
```

`--scanline` is palette-specific (see §1). The `mix-blend-mode: multiply` ensures the dark overlay interacts naturally with bright gear glow rather than just being layered on top.

### 4.2 Vignette

Simulates CRT screen curvature darkening edges. Elliptical gradient matching typical CRT aspect ratios.

```css
.vignette {
  position: fixed;
  inset: 0;
  background: radial-gradient(
    ellipse 70% 60% at center,
    transparent 0%,
    transparent 45%,
    rgba(0, 0, 0, 0.35) 70%,
    var(--vignette) 100%
  );
  pointer-events: none;
  z-index: 11;
}
```

Three-stop gradient: clean center → soft falloff begins at 45% → full dark at edge. The ellipse shape (wider than tall) matches CRT barrel geometry.

### 4.3 Phosphor Glow on Canvas

Applied directly to the `<canvas>` element via CSS filter. Two concentric drop-shadows: tight inner glow + diffuse outer aura.

```css
canvas {
  filter: 
    drop-shadow(0 0 4px var(--glow-solid))
    drop-shadow(0 0 12px var(--glow-rgba))
    drop-shadow(0 0 28px var(--glow-rgba));
}
```

Three layers:
1. `4px` — tight crisp edge glow, reinforces pixel boundaries
2. `12px` — medium aura, the main phosphor bloom
3. `28px` — wide diffuse glow, fills surrounding screen area

For Neon Cyber palette, add a second color glow using the accent:

```css
canvas[data-palette="cyber"] {
  filter:
    drop-shadow(0 0 4px var(--glow-solid))
    drop-shadow(0 0 12px var(--glow-rgba))
    drop-shadow(0 0 28px rgba(191, 0, 255, 0.2));
}
```

### 4.4 Screen Glow / Ambient Brightening

Applied to the body or a full-viewport wrapper to simulate the phosphor lighting up the surrounding bezel.

```css
.screen-wrapper {
  box-shadow: 
    inset 0 0 60px rgba(0, 0, 0, 0.8),  /* inner darken at edges */
    0 0 40px var(--glow-rgba);            /* external ambient glow */
  filter: brightness(1.05) contrast(1.1);
}
```

`brightness(1.05)` — slight overall lift to simulate phosphor emission  
`contrast(1.1)` — punches darks darker and brights brighter, increases CRT feel

### 4.5 Barrel Distortion (Optional)

Subtle CSS perspective warp that makes the screen appear slightly convex, like a curved CRT glass.

```css
.screen-wrapper {
  perspective: 800px;
  transform-style: preserve-3d;
}

.screen-inner {
  transform: 
    perspective(800px) 
    rotateX(0.5deg) 
    rotateY(0deg) 
    scale(1.01);
  /* 0.5deg is barely perceptible but creates a sense of depth */
}
```

This is cosmetic only. Do not apply to the canvas itself — the gear interaction math assumes a flat projection. Apply to a non-interactive wrapper div.

### 4.6 Background Noise Texture

Extremely subtle grain to break up the pure solid background color. Implemented as an SVG filter applied to the body background, or a tiny tileable base64 PNG.

```css
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
  background-size: 200px 200px;
  pointer-events: none;
  z-index: 1;
  opacity: 0.4;
  mix-blend-mode: screen;
}
```

Opacity is intentionally low (`0.4` on a `0.04` base SVG). The noise should be something felt rather than seen.

### 4.7 Screen Flicker (Optional)

0.5% chance per frame of a brightness pulse. Implemented in JS inside the render loop.

```js
if (Math.random() < 0.005) {
  canvas.style.filter = 'brightness(1.15) ' + baseCanvasFilter
  setTimeout(() => { canvas.style.filter = baseCanvasFilter }, 40)
}
```

40ms duration — long enough to register, short enough to read as a glitch not a strobe. Do not implement as a CSS animation (would be too regular and predictable).

---

## 5. Typography

### Font

**Primary:** `Press Start 2P` (Google Fonts)  
**Fallback:** `monospace`

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
```

```css
* {
  font-family: 'Press Start 2P', monospace;
}
```

`Press Start 2P` renders at approximately 1.5× its stated size in pixel height. A `10px` declaration renders roughly as a 15px-tall character block. Account for this in layout.

### TACTICS Title

```css
.title {
  font-size: 11px;
  letter-spacing: 0.25em;
  text-transform: uppercase;
  color: var(--text);
  text-shadow:
    0 0 8px var(--glow-solid),
    0 0 20px var(--glow-rgba);
}
```

At `11px Press Start 2P`, the title renders roughly 16–17px tall. The wide letter-spacing (0.25em) gives it an open, terminal-readout feel.

### Palette Toggle Button

```css
.palette-btn {
  font-size: 8px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  background: transparent;
  border: 1px solid var(--accent);
  padding: 4px 8px;
  cursor: pointer;
  image-rendering: pixelated;
}

.palette-btn:hover {
  background: var(--accent);
  color: var(--bg);
}
```

No border-radius. No transitions on hover — instant state change matches the CRT aesthetic.

### Stats Label (REV / RPM)

```css
.stats {
  font-size: 9px;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--text);
  opacity: 0.7;    /* slightly dimmed — secondary info */
  text-shadow: 0 0 6px var(--glow-rgba);
}
```

The `REV:` label text is static; only the number updates. Consider `font-variant-numeric: tabular-nums` equivalent — `Press Start 2P` is monospace by nature, so digits don't shift layout.

### Header Bar

```css
.header {
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  border-bottom: 1px solid var(--header-border);
  box-shadow: 0 1px 8px var(--glow-rgba);
}
```

---

## 6. Layout Wireframe (ASCII)

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  TACTICS                              [ GRN / AMB / CYB ] │  ← 40px header bar
│_________________________________________________________|  ← 1px border + glow
│                                                         │
│                                                         │
│                          ▼                              │  ← Pawl (12px above gear top)
│                        ╔═╧═╗                           │
│                      ██║   ║██                          │
│                   ███ ║     ║ ███                       │
│                  ██   ║  ⊕  ║   ██                     │  ← Gear (80vmin canvas)
│                   ███ ║     ║ ███                       │
│                      ██║   ║██                          │
│                        ╚═══╝                           │
│                                                         │
│                                                         │
│                       REV: 007                          │  ← Stats label
│                       RPM: 142                          │
│                                                         │
└─────────────────────────────────────────────────────────┘

Dimensions:
  Header:      40px fixed height, full width
  Gear canvas: 80vmin × 80vmin, centered
  Pawl:        ~12px above gear top edge
  Stats:       ~20px below gear bottom edge
  All layout:  flexbox column, items centered
```

**DOM structure:**

```
body (flex column, align-center, justify-center, height: 100vh, overflow: hidden)
├── header.header (fixed top, z-index 20)
│   ├── span.title — "TACTICS"
│   └── button.palette-btn — "[GRN]"
├── div.screen-wrapper (relative, houses gear + pawl)
│   ├── .pawl (absolute, centered above gear)
│   └── canvas (80vmin × 80vmin)
├── div.stats
│   ├── span — "REV: "
│   ├── span.rev-count — "000"
│   └── (optional) span.rpm — "RPM: 000"
├── div.scanlines (fixed overlay, z-10)
└── div.vignette (fixed overlay, z-11)
```

---

## 7. Animation Spec

### 7.1 Pawl Snap

| Phase | Duration | Easing | Property |
|---|---|---|---|
| Snap (to offset) | Instant (0ms) | n/a | `transform`, `color` |
| Color flash hold | 30ms | n/a | `color` stays `--pawl-active` |
| Return to idle | 80ms | `cubic-bezier(0.2, 0, 0.8, 1)` | `transform` back to 0 |
| Color fade | 80ms | `ease` | `color` back to `--accent` |

The cubic-bezier `(0.2, 0, 0.8, 1)` is a deceleration curve — starts fast, eases out. This mimics a spring return: the pawl snaps back quickly at first, then settles. Avoid `ease-in-out` — it starts too slowly and feels mushy.

Full animation timeline from click moment:
- `t=0ms`: class added, pawl jumps to offset position, turns white
- `t=30ms`: class removed, CSS transition begins, pawl starts returning
- `t=110ms` (30+80): pawl fully back to idle position and accent color

For rapid sequential clicks (fast spin), the 30ms timeout may fire before the 80ms return completes. This is intentional and looks correct — the pawl will interrupt mid-return and snap forward again. No animation queuing needed.

### 7.2 Gear Rotation

```js
// Applied inside render loop — no CSS transition, no easing
ctx.translate(cx, cy)
ctx.rotate(gearRotation)  // gearRotation updated directly from pointer delta
drawGear(ctx)
```

The gear transform is **never CSS-animated**. It is always the direct result of pointer math. No `transition` on the canvas element. No spring. No lerp. Direct 1:1 mapping.

The canvas element itself only has CSS `filter` applied — no CSS `transform`.

### 7.3 Screen Flicker

```
Each animation frame:
  roll = Math.random()          // 0.0 to 1.0
  if roll < 0.005:              // 0.5% chance — about once every 3 seconds at 60fps
    apply brightness(1.15) filter pulse to canvas
    revert after 40ms
```

Flicker is a brightness pulse only, never a color shift or geometric distortion. It should be surprising the first time you notice it, then unremarkable after.

### 7.4 Palette Transition

When cycling palettes (button click or `P` key):

```js
// Instant swap — no cross-fade
document.documentElement.setAttribute('data-palette', nextPalette)
localStorage.setItem('tactics-palette', nextPalette)
```

No transition between palettes. The instant switch reads as a CRT's brightness/color control being adjusted — which is exactly what it is.

### 7.5 Phosphor Persistence (Optional Frame Effect)

Instead of `clearRect()`:

```js
ctx.fillStyle = 'rgba(0, 0, 0, 0.85)'  // 85% opaque — previous frame bleeds through at 15%
ctx.fillRect(0, 0, canvas.width, canvas.height)
```

This creates a 1-frame trail. At 60fps with 85% fade per frame:
- Frame N: 100% opacity
- Frame N+1: 15% opacity
- Frame N+2: ~2% opacity (effectively gone)

The trail is only visible during fast rotation. During slow dragging it is imperceptible. Do not increase the trail persistence (lower the 0.85 value) — it will create smearing artifacts instead of a subtle phosphor effect.

---

## 8. Pixel Art Gear Drawing Algorithm

### Pseudocode

```
function drawGear(ctx, R, Rroot, Rhub, Rhole, numTeeth):
  
  TOOTH_ANGLE = (2 × π) / numTeeth          // arc for one full tooth slot (22.5°)
  TOOTH_FILL  = TOOTH_ANGLE / 2             // arc occupied by tooth (11.25°)
  GAP_FILL    = TOOTH_ANGLE / 2             // arc occupied by valley (11.25°)
  
  ctx.beginPath()
  
  // Build the gear outline as a single closed path
  // Alternate between outer radius (tooth tips) and inner radius (tooth valleys)
  
  for i = 0 to numTeeth - 1:
    
    θ_slot_start = i × TOOTH_ANGLE
    θ_tooth_start = θ_slot_start
    θ_tooth_end   = θ_slot_start + TOOTH_FILL
    θ_gap_start   = θ_tooth_end
    θ_gap_end     = θ_slot_start + TOOTH_ANGLE
    
    // Tooth: 4 corners traversed in order
    // Corner A (inner-left of tooth)
    ax = round(Rroot × cos(θ_tooth_start))
    ay = round(Rroot × sin(θ_tooth_start))
    
    // Corner B (outer-left of tooth — rise to tip)
    bx = round(R × cos(θ_tooth_start))
    by = round(R × sin(θ_tooth_start))
    
    // Corner C (outer-right of tooth — flat tooth top)
    cx = round(R × cos(θ_tooth_end))
    cy = round(R × sin(θ_tooth_end))
    
    // Corner D (inner-right of tooth — drop back down)
    dx = round(Rroot × cos(θ_tooth_end))
    dy = round(Rroot × sin(θ_tooth_end))
    
    if i == 0:
      ctx.moveTo(ax, ay)
    else:
      ctx.lineTo(ax, ay)    // arrive from previous gap
    
    ctx.lineTo(bx, by)      // rise: inner-left → outer-left
    ctx.lineTo(cx, cy)      // cross: outer-left → outer-right (flat tooth top)
    ctx.lineTo(dx, dy)      // fall: outer-right → inner-right
    
    // Gap: arc along inner radius from θ_gap_start to θ_gap_end
    // Use arc() for smoothness, or lineTo for extra pixel crunch (both acceptable)
    // ctx.arc(0, 0, Rroot, θ_gap_start, θ_gap_end)
    // OR, for harder pixel look, just lineTo the gap endpoints and let it chord:
    gx = round(Rroot × cos(θ_gap_end))
    gy = round(Rroot × sin(θ_gap_end))
    ctx.lineTo(gx, gy)      // gap floor (chord approximation at low-res is fine)
  
  ctx.closePath()
  
  // Fill gear body
  ctx.fillStyle = gearFillColor
  ctx.fill()
  
  // Stroke gear outline
  ctx.strokeStyle = gearStrokeColor
  ctx.lineWidth = 1
  ctx.stroke()
  
  // Draw hub
  ctx.beginPath()
  ctx.arc(0, 0, Rhub, 0, 2 × π)
  ctx.fillStyle = hubFillColor
  ctx.fill()
  ctx.stroke()
  
  // Draw center hole
  ctx.beginPath()
  ctx.arc(0, 0, Rhole, 0, 2 × π)
  ctx.fillStyle = backgroundColor
  ctx.fill()
  // No stroke on hole
  
  // Draw crosshair
  ctx.beginPath()
  ctx.moveTo(-4, 0)
  ctx.lineTo(4, 0)
  ctx.moveTo(0, -4)
  ctx.lineTo(0, 4)
  ctx.strokeStyle = backgroundColor
  ctx.lineWidth = 1
  ctx.stroke()
```

### Notes on Pixel Alignment

At low internal resolution (~160px canvas), `round()` on vertex coordinates is essential. Without rounding, sub-pixel coordinates will cause the `ctx.arc`-based operations to produce inconsistent pixel patterns when scaled up. The scale-up factor (~4×) amplifies any sub-pixel slop into visible jagged edges.

The gap floor uses a **chord** (straight `lineTo`) rather than a proper arc. At 11.25° and Rroot ~64px, the chord deviation from a true arc is `< 1px` at internal resolution — indistinguishable from an arc, and produces harder pixel edges when scaled up. This is the correct choice for the pixel-art aesthetic.

### Rotation Application

The draw function always draws the gear **centered at origin (0, 0)**. Rotation is applied via `ctx.rotate()` before drawing — never by rotating the vertex angles themselves. This keeps the math clean and the center guaranteed at canvas center.

```js
ctx.save()
ctx.translate(canvas.width / 2, canvas.height / 2)
ctx.rotate(gearRotation)
drawGear(ctx, R, Rroot, Rhub, Rhole, 16)
ctx.restore()
```

### Anti-aliasing

```js
ctx.imageSmoothingEnabled = false
```

Set once after canvas context creation. Does not need to be reset each frame. The gear will have hard aliased edges at internal resolution, which appear as chunky pixel-art steps when scaled 4× — exactly the intended look.

---

## Appendix: Quick Reference

### Full Color Token Summary

| Token | Phosphor Green | Amber Terminal | Neon Cyber |
|---|---|---|---|
| `--bg` | `#0a0f0a` | `#0f0a00` | `#050010` |
| `--gear-fill` | `#39ff14` | `#ffb300` | `#00d4ff` |
| `--gear-stroke` | `#1a7a08` | `#7a4400` | `#004466` |
| `--gear-hub` | `#2acc10` | `#e69900` | `#008fb3` |
| `--glow-solid` | `#00ff41` | `#ff8c00` | `#00d4ff` |
| `--glow-rgba` | `rgba(0,255,65,0.4)` | `rgba(255,140,0,0.4)` | `rgba(0,212,255,0.35)` |
| `--accent` | `#7fff00` | `#ffd700` | `#ff00ff` |
| `--pawl-idle` | `rgba(57,255,20,0.4)` | `rgba(255,179,0,0.4)` | `rgba(0,212,255,0.4)` |
| `--pawl-active` | `#ffffff` | `#ffffff` | `#ff00ff` |
| `--text` | `#39ff14` | `#ffb300` | `#00d4ff` |
| `--header-border` | `#1a7a08` | `#7a4400` | `#bf00ff` |
| `--scanline` | `rgba(0,0,0,0.15)` | `rgba(0,0,0,0.18)` | `rgba(0,0,0,0.12)` |
| `--vignette` | `rgba(0,0,0,0.7)` | `rgba(0,0,0,0.72)` | `rgba(0,0,20,0.75)` |

### Gear Size Quick Reference (at typical screen sizes)

| Screen vmin | gearDiameterPx (80vmin) | internalSize (÷4) | R (×0.45) | Rroot (×0.40) |
|---|---|---|---|---|
| 375px (mobile) | 300px | 75px | 33.75px | 30px |
| 667px (tablet) | 533px | 133px | 59.85px | 53.2px |
| 800px (desktop) | 640px | 160px | 72px | 64px |
| 1024px (desktop) | 819px | 204px | 91.8px | 81.6px |

---

DESIGN_DONE
