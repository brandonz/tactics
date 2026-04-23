# TACTICS — Design Review
**Reviewer:** SrirachaBot  
**Date:** 2026-04-23  
**Against:** DESIGN.md v1.0 (source of truth)

---

## 1. Color Palettes

### FAIL — Architecture mismatch

Spec calls for CSS custom properties on `:root[data-palette="green|amber|cyber"]` with 13 tokens per palette. Implementation uses JS objects with 6 fields set via inline style `--color-*` variables.

**Missing tokens (all three palettes):** `--gear-stroke`, `--gear-hub`, `--glow-rgba`, `--pawl-idle`, `--pawl-active`, `--text`, `--header-border`, `--scanline`, `--vignette`. These collapse into single variables or are hardcoded.

**CYBER glow color WRONG:** implementation sets `glow: '#bf00ff'` (purple/accent) where spec requires `--glow-solid: #00d4ff` (cyan). The CYBER canvas glows purple instead of cyan.

**`dark` values don't match spec stroke or hub:**
| Palette | impl `dark` | spec `--gear-stroke` | spec `--gear-hub` |
|---|---|---|---|
| GREEN | `#003b0e` | `#1a7a08` | `#2acc10` |
| AMBER | `#3b2000` | `#7a4400` | `#e69900` |
| CYBER | `#001030` | `#004466` | `#008fb3` |

The `dark` value is used for both hub fill and stroke — but it's too dark and the hub colour is completely wrong (e.g., green hub should be `#2acc10`, a brighter green, not near-black `#003b0e`).

**glow-rgba opacity:** implementation uses `0.5`, spec uses `0.4` for green/amber and `0.35` for cyber.

**Core bg/gear/accent hex values:** all correct for all three palettes. ✓

**Scanline and vignette:** hardcoded `rgba(0,0,0,0.15)` and `rgba(0,0,0,0.7)` — not palette-driven. Amber and Cyber variants have different values that are never applied.

---

## 2. Gear Geometry

### FAIL — Inner radius and tooth width both wrong

**Tooth count:** 16 ✓  
**Canvas low-res setup (÷4 scale, imageSmoothingEnabled=false):** ✓  
**Outer radius (R):** `72 * s` = 0.45 × internalSize ✓  
**Hub radius (Rhub):** `20 * s` = 0.125 × internalSize ✓  
**Center hole radius (Rhole):** `7 * s` = 0.044 × internalSize ✓  

**Inner radius (Rroot) WRONG:**  
- Implementation: `innerR = 58 * s` → 58/160 = **0.3625** of internalSize  
- Spec requires: `Rroot = internalSize * 0.40` → **0.40** (= 64px at 160px)  
- Tooth depth at 160px internal: implementation = 14px, spec = 8px  
- Teeth are **75% deeper** than specced — more like cog-wheel spikes than gear teeth

**Tooth angular width WRONG:**  
- `TOOTH_HW = 0.12 rad` → each tooth spans `0.12 × 2 = 0.24 rad`  
- Spec: 50% duty cycle → tooth arc = π/16 = **0.1963 rad**  
- Implementation duty cycle: 0.24 / 0.3927 = **61%** — teeth are noticeably wider than gaps  
- Spec requires equal tooth/gap width (50/50)

**Hub design FAIL — missing elements:**  
- Hub fill uses `p.dark` (wrong color per §1 above) instead of `--gear-hub`  
- Hub is not stroked separately (spec §2: "Stroke the hub circle separately")  
- **Crosshair is completely missing** — spec §2 Hub Design requires horizontal + vertical lines drawn over hub in background color  

**Pixel crispness:** `image-rendering: pixelated`, vertex rounding with `Math.round()`, and no devicePixelRatio multiplication are all correct. ✓

**Coordinate rounding:** applied to all tooth corners ✓

---

## 3. CRT Effects

### FAIL — Several effects missing or incorrect

**Scanlines:** present with correct repeating-linear-gradient pattern ✓ — but hardcoded `rgba(0,0,0,0.15)` (not palette-variable), and **missing `mix-blend-mode: multiply`** which the spec explicitly requires for correct blending with bright glow.

**Vignette:** present ✓ — but gradient uses only 2 stops (`transparent 60%`, `rgba 100%`) vs spec's 4-stop gradient (`transparent 0%`, `transparent 45%`, `rgba(0,0,0,0.35) 70%`, `var(--vignette) 100%`). The spec's 45% transparent hold zone creates a larger clean center; implementation starts darkening earlier.

**Phosphor glow (canvas drop-shadow):** 2 drop-shadow layers applied, spec requires 3:
- Implementation: `drop-shadow(0 0 8px ...) drop-shadow(0 0 20px ...)`  
- Spec: `drop-shadow(0 0 4px ...) drop-shadow(0 0 12px ...) drop-shadow(0 0 28px ...)`  
- Missing the tight 4px edge glow layer. Cyber special-case glow (§4.3) not implemented.

**Screen wrapper glow (§4.4):** `brightness(1.05) contrast(1.1)` on body ✓ — but missing the `box-shadow` ambient glow component.

**Background noise texture (§4.6):** not implemented ✗

**Screen flicker (§4.7):** not implemented (marked optional — acceptable)

**Barrel distortion (§4.5):** not implemented (marked optional — acceptable)

**Phosphor persistence (§7.5):** uses `fillRect` solid clear instead of `rgba(0,0,0,0.85)` fade. Marked optional — acceptable, but the effect is nice.

---

## 4. Pawl Indicator

### FAIL — Color, timing, and Cyber active state wrong

**Shape:** 12×10px downward-pointing triangle via clip-path polygon. Correct dimensions ✓

**Position:** computed from center and gear radius, fixed positioning ✓ — gap to gear top is 14px; spec says 20px. Minor ✗

**Idle color WRONG:**  
- Implementation: `var(--color-accent)` at `opacity: 0.5`  
- Spec: `var(--pawl-idle)` = gear fill color at 40% opacity (e.g., `rgba(57,255,20,0.4)` for green)  
- Idle should look like a dim version of the gear, not the accent color

**Active color — Cyber palette WRONG:**  
- Implementation: always flashes `#ffffff`  
- Spec: Cyber `--pawl-active` is `#ff00ff` (magenta), not white

**Snap timing WRONG:**  
- Implementation: class stays 80ms then is removed  
- Spec: class removed at **30ms** (white flash hold), then 80ms CSS transition returns it  
- Total timeline: 110ms (30ms flash + 80ms return). Implementation collapses this — the transition fires when class is removed, so no return glide

**Snap direction:** `±4px` translateX based on direction ✓  
**Return easing:** `cubic-bezier(0.2, 0, 0.8, 1)` ✓

---

## 5. Typography

### FAIL — Font sizes, letter-spacing, and button behavior differ

**Font:** `Press Start 2P` from Google Fonts, applied globally ✓

**Title ("TACTICS"):**
- Font size: 12px vs spec's 11px ✗  
- Missing `letter-spacing: 0.25em` ✗  
- `text-shadow` uses same color for both glow values; spec uses `--glow-solid` + `--glow-rgba` (different opacity) ✗  
- `text-transform: uppercase` missing (not critical since text is already uppercase)

**Palette button:**
- Font size: 7px vs spec's 8px ✗  
- Button text is "PALETTE" — spec wireframe shows "[GRN]" cycling label showing current palette ✗  
- Missing `letter-spacing: 0.1em` ✗  
- Has `transition: background 0.1s` — spec explicitly says **no transitions on hover** for CRT aesthetic ✗  
- Uses `:active` state not `:hover` (minor — touch-friendly tradeoff)  
- Uses `--color-gear` for border instead of `--accent` ✗

**Stats:**
- Font size: 10px vs spec's 9px ✗  
- `letter-spacing: 1px` vs spec's `0.15em` ✗  
- Missing `opacity: 0.7` (secondary dimming) ✗  
- Shows "DIR: CW/CCW" — spec shows "RPM:" (revolution speed) not "DIR:" ✗  
- Spec's `--scanline` for text-shadow not palette-driven

**Header:**
- Height 40px ✓, flex layout ✓  
- `border-bottom` uses `--color-gear` — spec says `--header-border` (dark variant) ✗  
- Missing `box-shadow: 0 1px 8px var(--glow-rgba)` ✗

---

## 6. Visual Issues Summary

| Issue | Severity |
|---|---|
| Gear teeth 75% deeper than spec (innerR 58 vs 64 at 160px) | High |
| Gear teeth 22% wider than spec (61% vs 50% duty cycle) | Medium |
| Hub fill colour wildly wrong (near-black instead of distinct mid-tone) | High |
| Crosshair missing from hub entirely | Medium |
| Hub not separately stroked | Low |
| CYBER canvas glows purple instead of cyan | High |
| Scanlines missing `mix-blend-mode: multiply` — may over-darken bright pixels | Medium |
| Vignette gradient starts too early (no 45% hold zone) | Low |
| Missing third drop-shadow layer on canvas glow | Low |
| Background noise texture missing | Low |
| No `box-shadow` ambient screen glow | Low |

---

## 7. Mobile Rendering

### PASS with notes

- `user-scalable=no` viewport ✓  
- `touch-action: none` on canvas ✓  
- Pointer capture prevents touch drift ✓  
- Dead zone (20px CSS) prevents center-grab issues ✓  
- `flex-shrink: 0` prevents canvas from being squashed ✓  
- Layout uses flexbox column with `min-height: 0` to handle gear area ✓

**Concern:** At 375px vmin (iPhone), `internalSize = 75px` — the 16 teeth at this resolution are only ~5–6px per tooth at internal res. Scaled 4× to 300px CSS, teeth will be very blocky. This is intentional pixel art, but the tooth width error (61% vs 50%) will be more visible at low resolution. Also the stats and header consume 88px, leaving only ~287px for gear area — the 300px canvas will overflow vertically on small phones unless the gear-area flex works correctly. Worth testing.

---

## Summary Scorecard

| Check | Result |
|---|---|
| Palette hex values correct | PARTIAL (bg/gear/accent ✓, glow/stroke/hub ✗) |
| All 13 palette tokens implemented | FAIL |
| Gear: 16 teeth | PASS |
| Gear: inner radius correct | FAIL |
| Gear: tooth duty cycle 50% | FAIL |
| Gear: pixel crispness (low-res canvas, no smoothing) | PASS |
| Hub: correct fill color | FAIL |
| Hub: crosshair drawn | FAIL |
| Hub: stroked separately | FAIL |
| CRT: scanlines present | PASS |
| CRT: scanlines palette-driven + mix-blend-mode | FAIL |
| CRT: vignette present | PASS |
| CRT: vignette 4-stop gradient | FAIL |
| CRT: 3-layer phosphor glow | FAIL |
| CRT: background noise | FAIL |
| Pawl: shape and position | PASS |
| Pawl: idle color correct | FAIL |
| Pawl: Cyber active color (#ff00ff) | FAIL |
| Pawl: snap timing (30ms flash + 80ms return) | FAIL |
| Typography: Press Start 2P | PASS |
| Typography: font sizes | FAIL |
| Typography: letter-spacing | FAIL |
| Typography: button no-transition | FAIL |
| Typography: palette button shows current palette | FAIL |
| Mobile: touch input | PASS |
| Mobile: layout integrity | PASS (minor concern at 375px) |
