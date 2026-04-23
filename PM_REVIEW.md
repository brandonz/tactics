# PM Review — Tactics v1.0
**Reviewer:** PM/UX  
**Date:** 2026-04-23  
**Verdict:** CONDITIONAL SHIP (minor fixes before release)

---

## Acceptance Criteria

### AC-01 — Rotation tracks finger 1:1
**PASS**  
Correct `Math.atan2` angle computation, proper ±π delta normalization, delta applied directly with no scaling factor or smoothing. No interpolation, no lag introduced.

### AC-02 — Instant stop
**PASS**  
`pointerup` and `pointercancel` both call `endTracking`, which clears `tracking` and `capturedId`. No velocity tracking, no RAF animation of rotation. Gear stops updating in same frame as pointer release.

### AC-03 — 16 clicks per rotation
**PASS**  
`TOOTH_ANGLE = (2π)/16`. Tooth index via `Math.floor(gearRotation / TOOTH_ANGLE)`. On a full 360° rotation, exactly 16 floor-boundary crossings occur. Works for both CW (positive `gearRotation`) and CCW (negative `gearRotation` correctly floors to negative integers with symmetric boundaries).

### AC-04 — Rapid spin fires all clicks
**PASS**  
`crossed = Math.abs(currTooth - prevTooth)` and `for (let i = 0; i < crossed; i++) fireClick()`. No throttle, no debounce. Multi-tooth crossings in a single move event are fully counted.

### AC-05 — Android haptics fire
**PASS** *(device-verify required)*  
`navigator.vibrate(8)` called unconditionally when `navigator.vibrate` is present. Guard is correct — Android Chrome has the API, iOS does not. 8ms duration matches spec exactly.

### AC-06 — iOS audio click fires
**PASS with note**  
`isIOS` UA detection (`/iPhone|iPad|iPod/`) is standard. `initAudio()` is called on first `pointerdown` only when `isIOS` — this satisfies browser autoplay policy. Audio click waveform matches spec exactly (noise * power-8 decay, gain 0.4).  
**Note:** Non-iOS devices without `navigator.vibrate` (macOS Chrome, Firefox) get neither vibration nor audio feedback. Spec doesn't address this platform, but it's a real gap for desktop testing.

### AC-07 — Dead zone prevents flip
**PASS**  
Dead zone (20 CSS px) pauses tracking on `pointermove`. On exit from dead zone, `prevAngle` is re-synced before any delta is computed, preventing jump. The angle-resync-then-return pattern is correct.  
**Minor deviation:** `pointerdown` also rejects initial contact within the dead zone (returns early). Spec only describes dead zone behavior during `pointermove`. Starting in the dead zone with current code means tracking never starts at all — functionally safe and arguably better, but slightly more restrictive than spec.

### AC-08 — Pawl animates on each click
**PASS with notes**  
`firePawl(dir)` applies `translateX(±4px)` snap instantly via `transition: none` class, then removes `.clicked` after 80ms to trigger spring return via `cubic-bezier(0.2, 0, 0.8, 1)`. Color flashes white, returns to accent on class removal.  
**Deviation 1:** Implementation adds `translateY(4px)` to the snap transform. Spec only specifies horizontal snap offset.  
**Deviation 2:** Pawl element is 12×10px. Spec says ~12×20px.  
**Deviation 3:** Rapid-fire clicking uses `clearTimeout` to cancel the return timer and re-apply the clicked state — this means the spring-return is cut short on rapid spins. Functionally acceptable but not what spec describes ("gracefully overlaps").

### AC-09 — Single touch only
**PASS**  
`pointerdown` returns early if `tracking` is already true. `pointermove` guards on `e.pointerId !== capturedId`. `setPointerCapture` ensures first finger's events are routed correctly even when lifted outside canvas. Lifting first finger while second is down correctly stops tracking (endTracking checks capturedId, which is the first finger's ID).

### AC-10 — Pixel-crisp rendering
**PASS with note**  
Internal canvas at `cssSize/4`, displayed at `cssSize` via CSS. `image-rendering: pixelated` and `crisp-edges` applied. `imageSmoothingEnabled = false` set on context.  
**Note:** `window.devicePixelRatio` is not applied to canvas pixel dimensions. Spec Open Question #1 says to multiply by DPR for correct Retina handling, but notes the pixelated look should come from the low-res internal render. On a 2× Retina display, the effective upscale becomes 8× (4× render scale × 2× DPR) instead of 4×. The teeth will look chunkier/blockier on Retina. Whether this is acceptable or more-is-better is a judgment call — the teeth are still crisp (pixelated, not blurry), just coarser.

---

## UX Issues

### Critical (blocks ship)
None found.

### Major (should fix before ship)

**1. Palette button gives no feedback on current selection**  
The button always reads "PALETTE". After clicking, you don't know if you're on GREEN, AMBER, or CYBER. The palettes are visually obvious but the label should show current palette name (e.g., "AMBER") or at minimum an indicator. `PALETTES` objects have a `.name` field — just update `palette-btn.textContent` on each switch and on load.

**2. No background noise texture**  
Spec §3 explicitly calls for "a subtle noise texture via an SVG filter or a CSS background-image using a tiny base64-encoded noise PNG." Not implemented. This is a visual spec item, not optional — the spec says "Keep it very subtle — it should be noticed subconsciously, not consciously." The flat black background feels slightly sterile compared to what the CRT aesthetic promises.

### Minor (polish pass)

**3. Pawl dimensions wrong**  
Spec: `~12×20px`. Implementation: `12×10px` (height: 10px). The pawl is half as tall as specced, making it less prominent visually. Suggest: bump to `height: 20px`.

**4. Pawl snap adds unspecced vertical offset**  
`pawlEl.style.transform = 'translateX(${dx}px) translateY(4px)'` — the vertical 4px drop isn't in spec. It's a nice touch (simulates tooth engagement) but unreviewed. Harmless if intentional; flag for dev to confirm it was deliberate.

**5. Hub color doesn't match spec**  
Spec says hub should be "same color" as gear with a crosshair indicator. Implementation renders hub as `p.dark` (dark fill) with a `p.bg` center hole. No crosshair. The dark hub with hole looks decent but diverges from spec intent. The crosshair is listed as optional, but the color is specified. Minor visual deviation.

**6. Phosphor persistence not implemented**  
Spec §3 marks this "Optional, Low Priority." Not a blocker. Worth filing for v1.1 — it would significantly enhance the CRT feel.

**7. No keyboard shortcut for palette cycling**  
Spec says "a small button or keyboard shortcut." Only the button is implemented. Suggest: `'p'` key cycles palette. One `keydown` listener.

**8. Desktop users without vibration get no feedback**  
macOS/Windows desktop browsers generally don't support `navigator.vibrate`. They also aren't iOS, so they don't get audio. Result: spinning the gear on desktop produces only visual feedback. For a local tool used mostly on Brandon's Mac Mini, this is probably acceptable, but worth noting if it's demoed on desktop.

**9. No RPM display**  
Spec §5 says optional RPM display. Not implemented. `DIR:` is shown instead (not in spec). The DIR display is harmless and informative, but RPM would be more fun. File for v1.1.

**10. Palette not labeled in localStorage key per spec suggestion**  
Spec Open Question #2 says yes to palette persistence via localStorage. Implementation does this correctly (`tactics-pal` key). ✓ No issue.

---

## Spec Conformance Summary

| Section | Status | Notes |
|---|---|---|
| §1 Input Handling | ✅ Fully conformant | Pointer Events, capture, single-finger, instant stop |
| §1 Rotation Math | ✅ Fully conformant | Correct formula + wraparound normalization |
| §1 Dead Zone | ✅ Conformant | Minor pointerdown behavior difference |
| §2 Haptics — Android | ✅ Conformant | 8ms, guarded correctly |
| §2 Haptics — iOS Audio | ✅ Conformant | initAudio on pointerdown, correct waveform |
| §2 Click Timing | ✅ Fully conformant | No throttle, no debounce |
| §3 Visual / Palettes | ✅ All 3 implemented | localStorage persistence included |
| §3 Canvas Rendering | ✅ Conformant | Quarter-res, pixelated, drop-shadow glow |
| §3 Background Texture | ❌ Missing | Not implemented |
| §3 Phosphor Persistence | ⚠️ Not implemented | Optional/low-priority per spec |
| §4 Pawl Indicator | ⚠️ Mostly conformant | Wrong size, extra vertical offset |
| §4 Pawl Animation | ✅ Conformant | Snap + spring + flash |
| §5 Layout | ✅ Fully conformant | Header, gear centered, stats below |
| §5 Typography | ✅ Press Start 2P loaded | |
| §5 REV counter | ✅ Implemented | Counts absolute rotations |
| §6 Tech Stack | ✅ Zero dependencies | Single index.html |

---

## Verdict

**CONDITIONAL SHIP.** Core interaction model is solid — the 1:1 tracking, instant stop, 16-click tooth detection, and haptics implementation are all correct and match spec. The feel will be right.

Fix before ship:
1. Palette button should label current palette
2. Background noise texture (it's in spec, not optional)
3. Pawl height: 10px → 20px

The rest can be v1.1.
