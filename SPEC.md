# TACTICS — Product Spec

**Version:** 1.0  
**Date:** 2026-04-23  
**Status:** Source of Truth

---

## Overview

Tactics is a single-screen web app that simulates a tactile clicky fidget gear. A large stylized gear fills most of the viewport. The user drags it to spin it. Every tooth edge that passes a fixed reference point fires a haptic click. The entire value proposition is tactile feel — it should feel indistinguishable from a physical ratchet mechanism.

There is no game. There is no score. There is no goal. There is only the click.

---

## 1. Core Interaction Model

### Input Handling

- **Pointer Events API** — not Touch Events, not Mouse Events. Use `pointerdown`, `pointermove`, `pointerup`, `pointercancel`. This gives unified mouse + touch + stylus handling with no duplication.
- **Single-finger only.** On `pointerdown`, capture the pointer ID via `element.setPointerCapture(event.pointerId)`. Ignore any subsequent `pointerdown` events while a pointer is already tracked. First finger wins, second finger is ignored entirely.
- **Finger down** → begin tracking. Record initial angle from pointer position to gear center.
- **Finger move** → compute new angle from pointer to gear center. Delta from previous angle = rotation increment. Apply to gear. Check for tooth crossings. Render.
- **Finger up / cancel** → freeze gear instantly. Zero velocity. No coast. No easing. The gear stops the moment contact breaks.

### Rotation Math

```
angle = Math.atan2(pointer.y - center.y, pointer.x - center.x)
delta = angle - previousAngle
gearRotation += delta
previousAngle = angle
```

Normalize delta to handle the ±π wraparound:
```
if (delta > Math.PI)  delta -= 2 * Math.PI
if (delta < -Math.PI) delta += 2 * Math.PI
```

This prevents the gear from snapping 360° when the pointer crosses the -π/+π boundary.

### Dead Zone

If the pointer is within **20px of the gear center**, pause angle tracking for that move event. Do not update `previousAngle`. This prevents wild rotation flips when the finger is dragged across the center point. Resume tracking normally once the pointer exits the dead zone.

### Why No Momentum

Momentum / inertia / coasting would break the core feel entirely. A ratchet mechanism does not coast — it stops when you let go. Inertia also makes haptics unpredictable and unearned. The clicks should be a direct 1:1 consequence of the user's physical gesture, not a byproduct of a simulation. Momentum is explicitly forbidden. Do not add it. Do not add it "just a little." Do not add it as an option.

---

## 2. Haptics / Vibration Spec

### Tooth Click Trigger

The gear has **16 teeth**, evenly spaced → **22.5° per tooth** (2π/16 radians per tooth).

Each tooth has two edges: leading and trailing. For simplicity, fire one click per tooth per pass — i.e., fire when the gear rotation crosses any multiple of 22.5°. Direction does not matter for the trigger; clicks fire on both CW and CCW rotation.

```
toothAngle = (2 * Math.PI) / 16   // 22.5° in radians

// After applying delta:
previousTooth = Math.floor(previousRotation / toothAngle)
currentTooth  = Math.floor(currentRotation  / toothAngle)

if (previousTooth !== currentTooth) {
  fireClick()
}
```

For rapid multi-tooth crossings in a single move event (e.g. a fast swipe that skips 3 teeth), fire one click per tooth crossed:
```
teethCrossed = Math.abs(currentTooth - previousTooth)
for (let i = 0; i < teethCrossed; i++) fireClick()
```

### Platform Haptics

**Android Chrome / Desktop:**
```js
if (navigator.vibrate) {
  navigator.vibrate(8)  // 8ms — short, sharp, mechanical
}
```

**iOS (vibration API blocked):**
Use Web Audio API to play a synthesized click sound as the haptic proxy. iOS users will hear the click rather than feel it, which is acceptable — the sound itself provides the feedback loop.

```js
function createAudioClick(audioCtx) {
  const buffer = audioCtx.createBuffer(1, audioCtx.sampleRate * 0.02, audioCtx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < data.length; i++) {
    // Sharp transient: high amplitude spike decaying to zero
    data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / data.length, 8)
  }
  return buffer
}
```

Play via `AudioBufferSourceNode` with gain ~0.4 to avoid harshness. Initialize `AudioContext` on first `pointerdown` (required by browser autoplay policy — cannot init on page load).

### Click Timing Rules

- **No throttle.** Do not debounce or rate-limit clicks. If a fast swipe crosses 8 teeth in 40ms, fire 8 clicks as fast as the browser allows.
- **8ms vibration duration.** Do not increase this. Longer durations create a buzz instead of a click. The goal is a discrete, mechanical snap.
- **No overlap management.** If a new vibrate() call fires while a previous one is still active, the browser handles it (typically resets the timer). This is acceptable behavior for rapid spins.

---

## 3. Visual Design — Pixel/CRT Aesthetic

### Philosophy

The visual language is a 1980s CRT terminal or arcade cabinet. Sharp pixel edges, phosphor glow, scanlines, dark backgrounds. No gradients, no shadows, no rounded corners, no smooth animations. Everything looks like it was rendered on a green screen in 1983 and is being viewed through a slightly curved CRT.

### Color Palette — Three Options

The developer should implement all three and provide a way to cycle between them (a small button or keyboard shortcut). **Phosphor Green** is the default.

| Palette | Background | Gear Color | Glow | Accent |
|---|---|---|---|---|
| **Phosphor Green** | `#0a0f0a` | `#39ff14` | `#00ff41` | `#7fff00` |
| **Amber Terminal** | `#0f0a00` | `#ffb300` | `#ff8c00` | `#ffd700` |
| **Neon Cyber** | `#050010` | `#00d4ff` | `#bf00ff` | `#ff00ff` |

### Gear Rendering — Canvas

- Gear rendered on an HTML5 `<canvas>` element.
- `ctx.imageSmoothingEnabled = false` — pixel-crisp, no anti-aliasing.
- Gear diameter: **~80vmin**. Compute in pixels from `Math.min(window.innerWidth, window.innerHeight) * 0.80`.
- **16 teeth**, chunky and prominent. Tooth height: ~8% of gear radius. Tooth width: roughly equal to gap width (50% duty cycle). Teeth should look mechanical and industrial, not decorative.
- Gear body: filled with the palette's gear color, **no fill gradient**. The glow is applied via CSS `filter: drop-shadow()` on the canvas element, not drawn in canvas.
- Tooth rendering: draw gear as a polygon path. Outer tip of each tooth should be flat (square top), not pointed. This gives it the chunky pixel-ratchet look.
- Optional: a small circular **hub** at center (15% gear radius), same color, with a pixel cross-hair indicator.

### Pixel Rendering Approach

To achieve true pixel-art style on the gear, render at a lower internal resolution and scale up:
- Internal canvas resolution: `gearDiameterPx / 4` (quarter resolution)
- CSS size: full `gearDiameterPx`
- This naturally pixelates the gear teeth into blocky, chunky segments

### CRT Effects — CSS Layer

Apply as CSS overlays on top of the canvas, using pointer-events: none so they don't interfere with input:

**Scanlines:**
```css
.scanlines {
  position: fixed;
  inset: 0;
  background: repeating-linear-gradient(
    to bottom,
    transparent 0px,
    transparent 2px,
    rgba(0,0,0,0.15) 2px,
    rgba(0,0,0,0.15) 4px
  );
  pointer-events: none;
  z-index: 10;
}
```

**Vignette (screen curvature simulation):**
```css
.vignette {
  position: fixed;
  inset: 0;
  background: radial-gradient(ellipse at center,
    transparent 60%,
    rgba(0,0,0,0.7) 100%
  );
  pointer-events: none;
  z-index: 11;
}
```

**Phosphor Glow (on canvas element):**
```css
canvas {
  filter: drop-shadow(0 0 8px var(--glow-color))
          drop-shadow(0 0 20px var(--glow-color));
}
```

### Phosphor Persistence (Optional, Low Priority)

On each frame, before clearing and redrawing the gear, fill canvas with a translucent black rect (opacity ~0.85) instead of `clearRect()`. This creates a brief motion trail that mimics phosphor persistence on old CRTs — the previous frame bleeds through faintly. Implement only if performance remains at 60fps.

### Background

Body background: palette's background color (`#0a0f0a` for Phosphor Green). Add a subtle noise texture via an SVG filter or a CSS background-image using a tiny base64-encoded noise PNG. Keep it very subtle — it should be noticed subconsciously, not consciously.

---

## 4. Pawl Visual Indicator

### What It Is

A small fixed indicator element positioned at the **top-center** of the gear (12 o'clock position, just outside the gear radius). It represents the ratchet pawl — the fixed tooth that the spinning gear clicks against.

### Visual Design

- Shape: a small downward-pointing triangle or rectangular tab, ~12×20px, same color as the gear.
- Positioned at: `center.x`, `center.y - gearRadius - 16px` (just above the topmost tooth path).
- Default state: dim (50% opacity of palette gear color).

### Click Animation

Every time `fireClick()` is called:

1. **Snap offset:** Move pawl element `+4px` in the direction of rotation (right for CW, left for CCW). Apply instantly (no ease-in).
2. **Spring return:** Transition back to original position over **80ms** with `cubic-bezier(0.2, 0, 0.8, 1)`.
3. **Color flash:** Set pawl color to bright white (`#ffffff`) for the first 30ms, then return to palette accent color over the remaining 50ms.

Implement using CSS custom properties + a brief class toggle (`pawl--clicked`) removed via `setTimeout(80)`. Do not use JavaScript animation libraries.

```css
.pawl { transition: transform 80ms cubic-bezier(0.2, 0, 0.8, 1), color 80ms ease; }
.pawl--clicked { transform: translateX(var(--snap-offset)); color: #ffffff; transition: none; }
```

---

## 5. Screen Layout

```
┌─────────────────────────────────────────┐
│ TACTICS                        [PALETTE] │  ← 40px header bar
│                                          │
│                  ▼                       │  ← Pawl indicator
│                                          │
│                                          │
│              [  GEAR  ]                  │  ← 80vmin canvas
│                                          │
│                                          │
│              REV: 003                    │  ← Stats label
└─────────────────────────────────────────┘
```

- **Full viewport, no scroll.** `html, body { overflow: hidden; height: 100%; margin: 0; }`
- **Gear centered** via flexbox on body: `display: flex; align-items: center; justify-content: center;`
- **Header bar** (40px, top): left-aligned "TACTICS" in pixel font, right-aligned palette toggle button. Subtle bottom border in palette gear color.
- **Stats label** below gear (not inside canvas): `REV: 003` — counts full 360° rotations. Use a monospace pixel font (see below). Update in real time.
- Optional: also display current RPM computed from recent angular velocity over a 500ms rolling window.

### Typography

Use `Press Start 2P` from Google Fonts — it is the canonical pixel/retro font. Fall back to `monospace`. Apply to all text elements. Font size: 10–12px for labels (it renders large for its size).

---

## 6. Technical Stack

| Concern | Solution |
|---|---|
| Structure | Single `index.html` — no build step, no bundler, no framework |
| Rendering | HTML5 Canvas (gear) + HTML/CSS (UI chrome, pawl, overlays) |
| Input | Pointer Events API (`pointerdown`, `pointermove`, `pointerup`, `pointercancel`) |
| Haptics | `navigator.vibrate()` (Android) + Web Audio API (iOS fallback) |
| Animation | `requestAnimationFrame` loop for gear rendering; CSS transitions for pawl |
| Fonts | Google Fonts CDN (`Press Start 2P`) |
| Dependencies | **Zero.** No npm, no node_modules, no external JS libraries |

The entire app ships as a single file that can be opened directly from the filesystem or served from any static host. Offline capable after first load (fonts will not load offline — acceptable).

File structure:
```
tactics/
└── index.html    ← everything: HTML, CSS, JS, inline or in <style>/<script> tags
```

---

## 7. Detailed Render Loop

```
requestAnimationFrame(render)

function render() {
  ctx.save()
  
  // Optional phosphor persistence: fill with translucent black
  ctx.fillStyle = 'rgba(0,0,0,0.85)'
  ctx.fillRect(0, 0, canvas.width, canvas.height)
  
  // Translate to gear center, rotate by gearRotation
  ctx.translate(center.x, center.y)
  ctx.rotate(gearRotation)
  
  // Draw gear body + 16 teeth as a closed polygon path
  drawGear(ctx, radius, 16)
  
  ctx.restore()
  
  // Update stats label (REV count, optional RPM)
  updateStats()
  
  requestAnimationFrame(render)
}
```

The render loop runs continuously even when the gear is not moving. This is fine — canvas is blank except for the gear, and idle redraws are cheap. Do not add logic to skip frames when idle.

---

## 8. Acceptance Criteria

Reviewers should verify all of the following before shipping:

- [ ] **AC-01 — Rotation tracks finger 1:1.** Dragging a finger in an arc rotates the gear by the same angular amount with no lag, no scaling, no smoothing.
- [ ] **AC-02 — Instant stop.** Releasing the finger freezes the gear immediately. No coast, no momentum, no deceleration. The gear is motionless within the same frame as `pointerup`.
- [ ] **AC-03 — 16 clicks per rotation.** A full 360° rotation fires exactly 16 haptic clicks, no more, no less. Verify in both CW and CCW directions.
- [ ] **AC-04 — Rapid spin fires all clicks.** A fast swipe that visually spins the gear 1+ full rotations fires the correct proportional number of clicks (e.g. 1.5 rotations = ~24 clicks). No clicks are silently dropped.
- [ ] **AC-05 — Android haptics fire.** On Android Chrome, each tooth crossing produces a distinct 8ms vibration felt in the hand. Verify with a physical Android device.
- [ ] **AC-06 — iOS audio click fires.** On iOS Safari, each tooth crossing produces an audible click sound. No vibration is expected. Verify the audio does not fail silently (check browser console for AudioContext errors).
- [ ] **AC-07 — Dead zone prevents flip.** Dragging the pointer across the gear center does not cause the gear to snap/flip. The dead zone suppresses tracking within 20px of center.
- [ ] **AC-08 — Pawl animates on each click.** The reference indicator snaps and flashes on every tooth crossing. The animation is visible and completes before the next click (or gracefully overlaps if clicking rapidly).
- [ ] **AC-09 — Single touch only.** Placing a second finger on screen does not alter rotation behavior. First finger maintains full control. Lifting first finger while second is down stops the gear.
- [ ] **AC-10 — Pixel-crisp rendering.** Gear edges are sharp and aliased on all screen resolutions including HiDPI/Retina. No blurry anti-aliased teeth. Verify by zooming in on a screenshot.

---

## Out of Scope (Explicitly Excluded)

- Multiplayer, sharing, social features
- Sound effects beyond the iOS click fallback
- Settings screen or preferences UI
- Gear customization (tooth count, size, color picker beyond the 3 palettes)
- Save state / progress tracking
- Analytics / telemetry
- PWA / service worker / installability
- Accessibility features (this is a haptic toy — screen reader support is not meaningful here)

---

## Open Questions (For Dev)

1. **Canvas resolution on HiDPI:** Multiply canvas pixel dimensions by `window.devicePixelRatio` and scale CSS size down to logical pixels. The pixelated gear look should be achieved by the low-resolution internal render described in §3, not by DPR mismatch.
2. **Palette persistence:** Should the selected palette survive a page refresh? Suggest: yes, via `localStorage`. One line of code, meaningful UX improvement.
3. **RPM calculation:** Optional but recommended. Track timestamps of the last N tooth clicks, compute angular velocity, display as RPM. Gives the user a fun performance metric.

---

*This spec is the source of truth. Deviations require PM sign-off.*
