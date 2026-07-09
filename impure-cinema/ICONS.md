# Impure Cinema — ICONS.md
### Complete icon reference, verified against current shipped code

Three separate icon systems exist in this app. They use different `viewBox`/`stroke-width` conventions on purpose (different sizes, different contexts) — keeping them straight matters, since an earlier icon round shipped in the wrong convention by mistake.

---

## 1. Wordmark glyph (brand mark, not a sprite)

Used once, in the header lockup next to "Impure Cinema." Not part of either `<use>` sprite below — it's a standalone inline SVG, since it only ever appears in one place.

```html
<svg viewBox="0 0 24 24" aria-hidden="true">
  <path d="M9 3h6M10 3v5.2L5.5 20h13L14 8.2V3" fill="none" stroke="var(--text-faint)" stroke-width="1.6" stroke-linejoin="round" stroke-linecap="round"/>
  <path d="M7.8 14h8.4L18.5 20H5.5z" fill="currentColor"/>
</svg>
```
- **Size:** 17×17px
- **Color:** `var(--accent)` (cyan, `#2FE6D8`) — the one place accent color, not tier color, tints a flask
- **Outline color:** hardcoded `var(--text-faint)`, independent of the fill — the flask outline stays neutral gray even though the liquid fill takes the accent color

---

## 2. Flask-tier sprite (Impurity Score)

**Convention:** `viewBox="0 0 64 64"`, `stroke-width="2.4"`, `stroke-linecap="round"`, `stroke-linejoin="round"`, defined once as a hidden `<svg style="display:none">` block, referenced via `<use href="#id">`.

**Purpose:** represents the Impurity Score's tier. Unlike a typical icon, the *shape itself* changes per tier, not just the color — this is deliberate: a cracked flask silhouette communicates "bad" even before you read the number.

**Rendered via:** `tierFlaskSvg(tierName, sizePx = 16)` → maps `SCORE_TIERS` names to symbol IDs:
```js
const TIER_FLASK_ID = { Panned: 'flask-panned', Mixed: 'flask-mixed', Liked: 'flask-liked', Acclaimed: 'flask-acclaimed' };
// falls back to 'flask-unmeasured' if tierName is null/unmatched
```

**Usage size:** 16×16px in the current UI (the Impurity Score hero).

### flask-panned (score < 50)
Cracked flask, one small shard flying off, one drip below.
```svg
<symbol id="flask-panned" viewBox="0 0 64 64">
  <path d="M26 11 V26 L13 50 a4 4 0 0 0 4 6 H47 a4 4 0 0 0 4 -6 L38 26 V11"/>
  <path d="M23 11 H32 l2 3 l3 -3 H41"/>
  <path d="M33 14 L30 22 L35 30 L31 41"/>
  <path fill="currentColor" stroke="none" d="M40 9 l4 -2 l1 4 Z"/>
  <path fill="currentColor" stroke="none" d="M50 51 c3 4 3 7 0 7 c-3 0 -3 -3 0 -7 Z"/>
</symbol>
```

### flask-mixed (50–69)
Calm flask, flat liquid line, no bubbles, small neck tick marks.
```svg
<symbol id="flask-mixed" viewBox="0 0 64 64">
  <path d="M26 11 V26 L13 50 a4 4 0 0 0 4 6 H47 a4 4 0 0 0 4 -6 L38 26 V11"/>
  <path d="M23 11 H41"/>
  <path stroke-width="1.5" d="M40 34 h4 M42 40 h4 M44 46 h4"/>
  <path fill="currentColor" stroke="none" d="M20 44 H44 L46 49 a3 3 0 0 1 -3 4 H21 a3 3 0 0 1 -3 -4 L18 49 Z"/>
</symbol>
```

### flask-liked (70–84)
Same base, liquid line raised, four small bubbles above the surface.
```svg
<symbol id="flask-liked" viewBox="0 0 64 64">
  <path d="M26 11 V26 L13 50 a4 4 0 0 0 4 6 H47 a4 4 0 0 0 4 -6 L38 26 V11"/>
  <path d="M23 11 H41"/>
  <path stroke-width="1.5" d="M40 34 h4 M42 40 h4 M44 46 h4"/>
  <path fill="currentColor" stroke="none" d="M23.5 36 H40.5 L46 49 a3 3 0 0 1 -3 4 H21 a3 3 0 0 1 -3 -4 L18 49 Z"/>
  <circle fill="currentColor" stroke="none" cx="28" cy="31" r="1.6"/>
  <circle fill="currentColor" stroke="none" cx="34" cy="27" r="2.1"/>
  <circle fill="currentColor" stroke="none" cx="38" cy="32" r="1.3"/>
  <circle fill="currentColor" stroke="none" cx="32" cy="20" r="1.5"/>
</symbol>
```

### flask-acclaimed (85+)
Overflowing liquid above the neck, a foam-cap shape, two side drips, three bubbles floating above the rim.
```svg
<symbol id="flask-acclaimed" viewBox="0 0 64 64">
  <path d="M26 11 V26 L13 50 a4 4 0 0 0 4 6 H47 a4 4 0 0 0 4 -6 L38 26 V11"/>
  <path d="M23 11 H41"/>
  <path stroke-width="1.5" d="M40 34 h4 M42 40 h4 M44 46 h4"/>
  <path fill="currentColor" stroke="none" d="M27.5 20 H36.5 V26.5 L46 49 a3 3 0 0 1 -3 4 H21 a3 3 0 0 1 -3 -4 L18 49 L27.5 26.5 Z"/>
  <path fill="currentColor" stroke="none" d="M24 12 q8 -7 16 0 q2 4 -1 6 q-7 -4 -14 0 q-3 -2 -1 -6 Z"/>
  <path fill="currentColor" stroke="none" d="M23 17 c-3 4 -3 7 0 7 c3 0 3 -3 0 -7 Z"/>
  <path fill="currentColor" stroke="none" d="M41 17 c3 4 3 7 0 7 c-3 0 -3 -3 0 -7 Z"/>
  <circle fill="currentColor" stroke="none" cx="31" cy="4" r="2"/>
  <circle fill="currentColor" stroke="none" cx="37" cy="6" r="1.4"/>
  <circle fill="currentColor" stroke="none" cx="26" cy="7" r="1.2"/>
</symbol>
```

### flask-unmeasured (no data available)
Dashed outline, no liquid, a question mark centered where the liquid would be.
```svg
<symbol id="flask-unmeasured" viewBox="0 0 64 64">
  <path stroke-dasharray="0.5 4" opacity="0.65" d="M26 11 V26 L13 50 a4 4 0 0 0 4 6 H47 a4 4 0 0 0 4 -6 L38 26 V11"/>
  <path stroke-dasharray="0.5 4" opacity="0.65" d="M23 11 H41"/>
  <text x="32" y="45" text-anchor="middle" font-family="monospace" font-size="22" font-weight="700" fill="currentColor" stroke="none" opacity="0.65">?</text>
</symbol>
```

**Color:** tinted per-tier via `currentColor`, matching `SCORE_TIERS` (`--tier-panned` → `--tier-acclaimed`, the neon-magenta brightness ramp). `flask-unmeasured` uses `var(--text-faint)` (neutral gray) since it has no tier.

---

## 3. Verdict sprite (Miscibility Index badges)

**Convention:** `viewBox="0 0 24 24"`, `stroke-width="2"`, `stroke-linecap="round"`, `stroke-linejoin="round"` — matching the app's general small-icon convention, distinct from the flask sprite's heavier 64-viewBox proportions.

**Purpose:** four badge icons for the Miscibility Index's classification and two independent context notes. Deliberately built around chemistry concepts (miscible/immiscible — whether two liquids blend or separate) rather than generic gesture icons (an earlier design used handshake/arm-wrestle icons and was replaced for feeling generic and unrelated to the app's own vocabulary).

**Rendered via:** `verdictIconSvg(id, sizePx = 15)` — always called with an explicit symbol ID (no tier-mapping function, unlike the flask sprite, since each badge has a fixed one-to-one icon).

**Usage size:** 15×15px, inside a `.flag-badge` text pill.

### verdict-miscible
Two circles overlapping, the lens-shaped intersection filled solid. Replaces a generic handshake. Deliberately sentiment-neutral — reads the same whether the agreement is "widely loved" or "widely panned," since a merged shape carries no emotional valence the way a handshake gesture does. The fill path's coordinates are the exact mathematical intersection points of the two r=6 circles (not hand-tuned), so the fill aligns perfectly with the outlines at any render size.
```svg
<symbol id="verdict-miscible" viewBox="0 0 24 24">
  <circle cx="8" cy="12" r="6"/>
  <circle cx="16" cy="12" r="6"/>
  <path fill="currentColor" stroke="none" d="M12 7.53 A6 6 0 0 1 12 16.47 A6 6 0 0 1 12 7.53 Z"/>
</symbol>
```
**Status:** approved, tested at 16/32/20/15px and in both bright and dusky tier-color tints.

### verdict-immiscible
A rounded container split into two flatly-filled bands by a straight divider line — like oil settled on water. Replaces a generic arm-wrestle icon. Same icon serves both Critic-Darling and Crowd-Pleaser classifications; direction is conveyed by the badge's text label ("Critics' film" / "Audience's film"), not by the icon.
```svg
<symbol id="verdict-immiscible" viewBox="0 0 24 24">
  <rect x="6" y="4" width="12" height="16" rx="3"/>
  <path d="M6 12.5 H18"/>
  <rect x="7.2" y="5.3" width="9.6" height="6.2" rx="1.5" fill="currentColor" fill-opacity="0.32" stroke="none"/>
  <rect x="7.2" y="13.2" width="9.6" height="6.5" rx="1.5" fill="currentColor" stroke="none"/>
</symbol>
```
**Status:** ⚠️ built directly from a design spec, not resubmitted through the design-review loop the other icons went through. Worth a direct on-device look before considering it final — see `ISSUES.md`.

### verdict-contested
A semicircular gauge arc with a single bent/kinked needle (not pointing cleanly to the rim) and a pivot dot. Reuses the app's own ring/gauge visual language (used throughout for scores) rather than borrowing an unrelated warning symbol. An earlier version used a dashed "ghost needle" pattern that was too fine to survive scaling to 15px (the dash length became sub-pixel) — this version has no dashed elements, only solid strokes, specifically to avoid that failure mode.
```svg
<symbol id="verdict-contested" viewBox="0 0 24 24">
  <path d="M4 15 A8 8 0 0 1 20 15"/>
  <path d="M12 15 L9 9 L13 6"/>
  <circle fill="currentColor" stroke="none" cx="12" cy="15" r="1.6"/>
</symbol>
```
**Status:** ⚠️ same caveat as Immiscible — built from spec, not visually re-approved. See `ISSUES.md`.

### verdict-divided
Two jagged half-circle pieces with interlocking torn edges, separated by a visible gap in the middle. Represents the audience splitting into two camps (as opposed to Immiscible's "two different liquids," this is one population dividing against itself). The visible negative-space gap between the two halves is deliberate — an earlier version (a single circle with one crack drawn across it) still read primarily as "one circle with a scratch," whereas two genuinely separated pieces is a harder-to-misread "split apart" story.
```svg
<symbol id="verdict-divided" viewBox="0 0 24 24">
  <path d="M10.5 4 A8 8 0 0 0 10.5 20 L8.5 16 L12 12 L8 9 Z"/>
  <path d="M13.5 4 A8 8 0 0 1 13.5 20 L11.5 16 L15 12 L11 9 Z"/>
</symbol>
```
**Status:** v2 of this icon (this version) is believed to supersede an earlier v1 (a simpler single-crack-through-one-circle design that was technically approved but shipped in the wrong viewBox convention). Where the two interlocking zigzag edges pass close together in the middle, confirm on-device that it reads as "two torn pieces" rather than a small tangle at actual size — flagged as worth a direct look, not a known problem.

**Color:** all four verdict icons render in `var(--text-dim)` by default (neutral, matching the badge pill's muted text color) — they are not tier-colored the way the flask sprite is, since a badge can attach to any classification and isn't itself a graded score.

---

## 4. Design rules for this icon system, if adding more

1. **Match the right convention.** Flask-tier additions: 64 viewBox, 2.4 stroke. Verdict/badge additions: 24 viewBox, 2 stroke. Mixing them up produces icons that render at the wrong relative weight next to their siblings.
2. **Budget for 15px.** Anything added to the verdict sprite will render at 15×15px in production. Test at that exact size before considering a design final — fine details (thin dashes, small multi-circle clusters) reliably fail at this scale; 2–4 bold elements is the practical ceiling.
3. **`currentColor` everywhere.** Every icon in both sprites uses `currentColor` for its primary strokes/fills so a parent's `color` CSS property tints the whole icon — never hardcode a color inside a symbol (the wordmark glyph's neutral outline is the one intentional exception, and it's hardcoded specifically so the outline *doesn't* follow the accent tint).
4. **Sprite once, `<use>` everywhere.** Don't inline a full `<svg>...</svg>` per instance the way the very first version of this app's icons did — define the shape once in the hidden sprite block, reference it via `<use href="#id">`. Cheaper, and guarantees every instance of an icon is pixel-identical.
