# Tactics — Staff Code Review

**Reviewer:** Staff Engineer  
**Date:** 2026-04-23  
**File reviewed:** `index.html`  
**Spec version:** 1.0

---

## Executive Summary

The implementation is largely correct and well-structured for a single-file app. The core interaction model, pointer capture, instant-stop behavior, and CSS CRT effects are all solid. However, there is one **critical** correctness bug in tooth detection that will cause the wrong click count on CCW rotation (violates AC-03), one **major** spec deviation in dead zone handling, and one **major** iOS audio reliability issue. Everything else is minor.

---

## Issues Found

---

### [CRITICAL] CCW tooth detection fires an extra click at zero

**Location:** `pointermove` handler, lines 390–394 and `pointerdown`, line 355

**Description:**  
The tooth index uses `Math.floor(gearRotation / TOOTH_ANGLE)`. This function is not symmetric around zero. `Math.floor(+ε / TOOTH_ANGLE) = 0` (no change), but `Math.floor(-ε / TOOTH_ANGLE) = -1` (immediate tooth crossing). In concrete terms:

- User starts with `gearRotation = 0`, `prevTooth = 0`.
- First CCW move of **any magnitude** sets `gearRotation = -0.0001`, `currTooth = floor(-0.000255) = -1`.
- `crossed = |(-1) - 0| = 1` → `fireClick()` is called.

This means a full CCW rotation fires **17 clicks** instead of 16, violating AC-03. Going CW from rest correctly requires traveling a full `TOOTH_ANGLE` before the first click. Going CCW from rest fires one click immediately.

This also means reversal behavior is wrong: spinning a little CW (no click), then reversing CCW (immediate click) instead of traveling back the same angular distance.

**Suggested fix:**  
Replace `Math.floor` with a symmetric computation. The cleanest approach is to offset by a half tooth before flooring, placing click boundaries symmetrically between teeth rather than aligned to zero:

```js
function toothIndex(rot) {
  return Math.floor((rot + TOOTH_ANGLE / 2) / TOOTH_ANGLE);
}

// In pointerdown:
prevTooth = toothIndex(gearRotation);

// In pointermove:
const currTooth = toothIndex(gearRotation);
```

This places click events at ±11.25° (half-tooth) boundaries, symmetric in both directions. A full rotation in either direction yields exactly 16 clicks.

---

### [MAJOR] Dead zone on `pointerdown` completely rejects the touch

**Location:** `pointerdown` handler, lines 348–349

```js
const { angle, cssDist } = pointerState(e);
if (cssDist < DEAD_ZONE) return;  // ← returns before setPointerCapture
```

**Description:**  
The spec says the dead zone should "pause angle tracking for that move event" and "Resume tracking normally once the pointer exits the dead zone." This language clearly describes `pointermove` behavior. For `pointerdown`, the spec says "Finger down → begin tracking."

The current implementation refuses to start tracking at all if the initial touch is within 20px of center. `setPointerCapture` is never called. If a user places their thumb near-center and slides outward (a natural grip on a small phone), the gear never responds — they must lift and re-touch.

Note that `initAudio()` is still called before the dead zone check (line 345), which is actually correct — audio context unlock should happen on any touch.

**Suggested fix:**  
Always start tracking and call `setPointerCapture` on `pointerdown`. Set `wasInDeadZone = true` if the initial touch is within the dead zone, but still set `tracking = true` and capture the pointer:

```js
canvas.addEventListener('pointerdown', e => {
  if (tracking) return;
  if (isIOS) initAudio();

  tracking   = true;
  capturedId = e.pointerId;
  canvas.setPointerCapture(e.pointerId);

  const { angle, cssDist } = pointerState(e);
  if (cssDist < DEAD_ZONE) {
    wasInDeadZone = true;
    // prevAngle not set yet — will be captured on dead zone exit
  } else {
    prevAngle     = angle;
    wasInDeadZone = false;
    prevTooth     = toothIndex(gearRotation);
  }
});
```

The existing `wasInDeadZone` logic in `pointermove` already handles the exit correctly (syncs `prevAngle` on exit and returns early). Extend it to also set `prevTooth` on exit:

```js
if (wasInDeadZone) {
  prevAngle     = angle;
  wasInDeadZone = false;
  prevTooth     = toothIndex(gearRotation);  // ← add this
  return;
}
```

---

### [MAJOR] `AudioContext` not resumed after suspension — silent iOS failure

**Location:** `initAudio()`, lines 294–299

**Description:**  
`initAudio()` creates the `AudioContext` on first touch, which starts it in "running" state per autoplay policy. However, iOS Safari suspends the `AudioContext` automatically when the tab loses focus (home button, notification, lock screen, tab switch). When the user returns and starts spinning, `audioCtx.state` is `"suspended"`. Calling `src.start()` on a suspended context fails silently on iOS — no error is thrown but no sound plays.

The code never calls `audioCtx.resume()` after the initial creation.

**Suggested fix — two changes:**

1. In `initAudio()`, add a resume after creation:
```js
function initAudio() {
  if (audioCtx) return;
  try {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
  } catch (e) { }
}
```

2. In `pointerdown`, always resume even if already initialized (this is a user gesture, so it's valid):
```js
canvas.addEventListener('pointerdown', e => {
  if (isIOS) {
    initAudio();
    if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
  }
  ...
```

`resume()` is safe to call on a running context (no-op).

---

### [MINOR] `isIOS` user-agent detection is fragile

**Location:** Line 291

```js
const isIOS = /iPhone|iPad|iPod/.test(navigator.userAgent);
```

**Description:**  
iPads running iPadOS 13+ in "Request Desktop Website" mode report a macOS user agent string (`Macintosh; Intel Mac OS X`). This is now the default for iPad Safari. These devices have no `navigator.vibrate` and benefit from the audio fallback, but `isIOS` will be `false` and they'll get no feedback at all.

A more robust platform detection: check for the absence of `navigator.vibrate` as a proxy for "needs audio fallback," which is the actual condition we care about:

```js
const needsAudioFallback = !navigator.vibrate;
```

This naturally handles iPads, future iOS devices, and any desktop browser without vibrate. Replace all `isIOS` references with `needsAudioFallback`.

---

### [MINOR] `body { filter: ... }` breaks `position: fixed` semantics

**Location:** CSS, line 31; pawl `#pawl { position: fixed }`, line 118

**Description:**  
`body` has `filter: brightness(1.05) contrast(1.1)`. Per CSS spec, any element with a `filter` value other than `none` creates a new containing block for `position: fixed` descendants. This means `#pawl`, `.scanlines`, and `.vignette` are all positioned relative to `body`, not the viewport.

In practice this is currently harmless because `body` has `height: 100%; overflow: hidden` and fills the viewport exactly. But it's semantically incorrect and fragile — any margin or padding added to body would misplace the pawl.

**Suggested fix:**  
Move `filter` off `body` and onto a wrapper div, or use a pseudo-element overlay instead:

```css
body { /* no filter here */ }
body::after {
  content: '';
  position: fixed;
  inset: 0;
  pointer-events: none;
  filter: brightness(1.05) contrast(1.1);
  /* but this doesn't affect content — bad approach */
}
```

Alternatively, apply the brightness/contrast via a fixed overlay div with `mix-blend-mode`, or simply remove it (the effect is subtle and the CSS glow effects already provide the CRT feel). The cleanest fix is to move the filter to a wrapper that doesn't contain the fixed elements:

```html
<div id="app-filter">
  <!-- header, gear-area, stats here -->
</div>
<!-- pawl, scanlines, vignette outside app-filter -->
```

---

### [MINOR] `localStorage` access not guarded against `SecurityError`

**Location:** Line 188–191

```js
let paletteIdx = Math.min(
  parseInt(localStorage.getItem('tactics-pal') || '0', 10),
  PALETTES.length - 1
);
```

**Description:**  
`localStorage.getItem()` throws `SecurityError` in private/incognito browsing mode in some browsers (Firefox, older Safari). This would crash the script on load before anything renders.

**Suggested fix:**
```js
function loadPaletteIdx() {
  try { return parseInt(localStorage.getItem('tactics-pal') || '0', 10); }
  catch (e) { return 0; }
}
let paletteIdx = Math.min(loadPaletteIdx(), PALETTES.length - 1);
```

Similarly wrap the `localStorage.setItem` call in the palette button handler.

---

### [MINOR] Resize handler is not debounced

**Location:** Line 481

```js
window.addEventListener('resize', setupCanvas);
```

**Description:**  
`setupCanvas` recomputes canvas dimensions, re-creates the internal coordinate system, and calls `positionPawl`. On desktop, dragging a window edge fires resize events at 60fps. Each call resets the canvas mid-draw. While not catastrophic (the rAF render loop will catch up), it causes unnecessary thrash.

**Suggested fix:** Debounce to ~100ms:
```js
let resizeTimer;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(setupCanvas, 100);
});
```

---

### [MINOR] Rapid pawl clicks can conflict with mid-transition transform

**Location:** `firePawl()`, lines 268–277

```js
function firePawl(dir) {
  if (pawlTimer) clearTimeout(pawlTimer);
  const dx = dir > 0 ? 4 : -4;
  pawlEl.classList.add('clicked');
  pawlEl.style.transform = `translateX(${dx}px) translateY(4px)`;
  pawlTimer = setTimeout(() => {
    pawlEl.classList.remove('clicked');
    pawlEl.style.transform = '';
  }, 80);
}
```

**Description:**  
When a rapid spin fires multiple clicks within 80ms (easily achievable — 3+ teeth in one swipe), each call resets `pawlEl.style.transform` to a new absolute value while potentially mid-transition from the previous call. The CSS transition is disabled during `.clicked` (via `transition: none`) but restored during the removal. The race between `setTimeout` callbacks and new `firePawl` calls can leave the pawl stuck in an intermediate position.

Specifically: if `clicked` is removed by the first timeout while a second click has already re-added it, the transform restoration fires with `transition: 80ms` briefly active before the next `transition: none` is set — causing a visible snap-back flash.

**Suggested fix:**  
Track the snap direction independently and use a CSS animation with `animation-name` toggle instead of manual transform + timer. Or simpler: just let rapid clicks fully stack — if a new click fires while the timer is pending, reset the transform to the new direction and reset the timer. Add a `Y` offset animation via CSS `@keyframes` rather than inline style:

```css
@keyframes pawl-snap {
  0%   { transform: translateX(var(--snap-dx)) translateY(4px); }
  100% { transform: translateX(0) translateY(0); }
}
.pawl--active {
  animation: pawl-snap 80ms cubic-bezier(0.2, 0, 0.8, 1) forwards;
}
```

Then each `firePawl` removes the class, forces a reflow, and re-adds it with the new `--snap-dx` value — this correctly restarts the animation from the beginning.

---

### [MINOR] No `devicePixelRatio` handling — canvas not pixel-crisp on HiDPI

**Location:** `setupCanvas()`, lines 226–248

**Description:**  
The spec (Open Questions §1) notes that DPR should be applied to the canvas pixel dimensions for crispness, while the pixelated look is achieved by the intentional quarter-resolution internal render (not DPR mismatch). The current code ignores DPR entirely. On a 3× display, the 4× CSS scaling of the internal canvas gets blended with the device's subpixel rendering rather than snapping to device pixels.

`image-rendering: pixelated` helps, but on Retina displays the blocky pixel art will render crisper if the canvas is DPR-aware:

```js
const dpr = window.devicePixelRatio || 1;
// Quarter-res for pixel art is still applied, but on top of DPR:
internalSize = Math.round(cssSize / 4);  // pixel-art resolution
canvas.width  = internalSize;
canvas.height = internalSize;
canvas.style.width  = cssSize + 'px';
canvas.style.height = cssSize + 'px';
// cssScale remains 4 regardless of DPR — pixelation is intentional
```

Actually the current approach achieves the pixelated look correctly. The spec's DPR note is about the hub/crosshair rendering quality rather than the tooth pixelation. This is low priority given the explicit pixelated aesthetic.

---

### [MINOR] `AudioContext` allocation per click creates GC pressure at high RPM

**Location:** `playClick()`, lines 302–319

**Description:**  
Every tooth crossing allocates a new `AudioBuffer` (20ms × sampleRate samples) and `AudioBufferSourceNode`. At high RPM (say 120 RPM = 32 teeth/second), this allocates ~32 buffers/sec. These are fire-and-forget and will be GC'd, but on low-memory devices the GC pauses can cause frame drops during rapid spins.

**Suggested fix:** Pre-compute and cache the click buffer once (it's random noise, so consistent content is fine — it will sound natural):

```js
let clickBuffer = null;

function initAudio() {
  if (audioCtx) return;
  try {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
    // Pre-bake click buffer
    const sr  = audioCtx.sampleRate;
    const len = Math.floor(sr * 0.02);
    clickBuffer = audioCtx.createBuffer(1, len, sr);
    const d = clickBuffer.getChannelData(0);
    for (let i = 0; i < len; i++) {
      d[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / len, 8);
    }
  } catch (e) { }
}

function playClick() {
  if (!audioCtx || !clickBuffer) return;
  try {
    const src = audioCtx.createBufferSource();
    src.buffer = clickBuffer;
    const gain = audioCtx.createGain();
    gain.gain.value = 0.4;
    src.connect(gain);
    gain.connect(audioCtx.destination);
    src.start();
  } catch (e) { }
}
```

`AudioBufferSourceNode` is designed to be lightweight — allocating one per click is fine. The `AudioBuffer` is the heavier allocation. Reusing the buffer eliminates per-click allocation.

---

## Items Verified Correct

These were on the review checklist and are **correctly implemented**:

- **Angle delta wrapping:** The ±π normalization is exactly per spec (`delta > Math.PI` → subtract 2π, `delta < -Math.PI` → add 2π). No snap at boundary.
- **`gearRotation` vs `totalRotation`:** `gearRotation` is the single cumulative rotation used for both rendering and tooth detection (correct). `totalRotation` accumulates `|delta|` for the REV counter (correct). No naming confusion, no separate track needed.
- **Fast swipe fires all clicks:** `crossed = Math.abs(currTooth - prevTooth)` loop correctly fires one click per tooth for multi-tooth swipe events. No drops.
- **`setPointerCapture`:** Called immediately in `pointerdown` (line 352). Drag works when finger moves outside canvas or even to screen edge.
- **Dead zone in `pointermove`:** Implemented correctly. On dead-zone entry, `wasInDeadZone = true`, no `prevAngle` update. On exit, `prevAngle` syncs to current angle and returns early (no rotation applied for that frame). Prevents flip artifact.
- **Instant stop:** `endTracking` only sets `tracking = false`. No velocity variable, no rAF cleanup needed — the rAF loop just keeps redrawing the same `gearRotation`. Gear is frozen within the next frame.
- **`navigator.vibrate` guard:** `if (navigator.vibrate) navigator.vibrate(8)` — correct feature detection.
- **`AudioContext` created on user gesture:** `initAudio()` is called in `pointerdown` handler. Not on page load. Satisfies browser autoplay policy.
- **`pointercancel` handled:** `canvas.addEventListener('pointercancel', endTracking)` — correct.
- **HTML validity:** Structure is valid. Doctype, charset, viewport meta all present. No unclosed tags. `touch-action: none` on canvas prevents scroll interference.
- **Gear path geometry:** The 16-tooth polygon path is geometrically correct. The leading/trailing corner points, flat tooth tops, and gap arcs are all properly constructed. The last tooth's gap arc correctly terminates at the first tooth's leading root (verified: both at 2π - TOOTH_HW). `closePath` closes cleanly.
- **Palette persistence:** `localStorage` load/save wired up (with caveat about SecurityError noted above).
- **CRT effects:** Scanlines, vignette, and phosphor glow via CSS `drop-shadow` are all implemented per spec. `pointer-events: none` on overlays is correct.
- **`ctx.imageSmoothingEnabled = false`:** Set in `setupCanvas` for pixel-crisp rendering.
- **Single-finger logic:** First pointer wins. Subsequent `pointerdown` events while `tracking = true` are ignored. Only the captured `pointerId` is processed in `pointermove` and `endTracking`.

---

## Priority Fix Order

1. **[CRITICAL]** CCW tooth asymmetry — fix `Math.floor` → symmetric `toothIndex()`. Breaks AC-03.
2. **[MAJOR]** Dead zone pointerdown rejection — move dead zone check out of the tracking-start gate.
3. **[MAJOR]** `audioCtx.resume()` on pointerdown — fixes silent iOS audio after tab switch.
4. **[MINOR]** localStorage SecurityError guard — prevents crash on private browsing.
5. **[MINOR]** `isIOS` → `needsAudioFallback` — fixes iPad desktop mode.
6. **[MINOR]** body filter stacking context — low risk given body fills viewport, but clean it up.
7. **[MINOR]** Pre-bake click buffer — performance improvement for high-RPM use.
8. **[MINOR]** Resize debounce — desktop quality-of-life.
9. **[MINOR]** Pawl animation restart race — visible only at very high RPM.

---

*REVIEW_DONE*
