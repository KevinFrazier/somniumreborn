# Hero Scroll & Dynamic Particle Effects Design

**Date:** 2026-03-31
**Project:** somnium-reborn.html
**Scope:** Enhance hero section with faster parallax and viewport-percentage-driven particle intensity

---

## Overview

Transform the hero section's visual feedback as users scroll. The feather image sinks faster down the page, while particle effects become "glowier and more intense" as the hero section remains visible, naturally fading as the user scrolls past.

---

## Hero Image Parallax

**Current State:**
- `featherImage` translates downward at `0.8x` scroll speed (moves 80% as far as user scrolls)

**New Behavior:**
- Increase multiplier to `1.2x` (moves 120% as far as user scrolls)
- Creates a "sinking" effect where the feather descends faster than the page scroll
- Multiplier applies only while hero section is in viewport; can stop or reset translation after hero exits

**Implementation Point:**
Update the scroll event handler that sets `featherImage.style.transform = translateY(${scrollTop * 1.2}px)`

---

## Particle Intensity System

**Core Metric: Visibility Percentage**

Calculate on every scroll event:
```
visibilityPercent = Math.max(0, Math.min(100,
  (heroElementVisibleHeight / heroElementTotalHeight) * 100
))
```

Where:
- `heroElementVisibleHeight` = pixels of hero currently in viewport
- `heroElementTotalHeight` = total hero section height
- Result ranges 0–100%, where:
  - 0% = hero completely off-screen
  - 100% = entire hero visible in viewport
  - Values in between scale smoothly

---

## Particle Spawn Rate

**Current State:**
- Spawn chance: 40% (only spawn if `Math.random() > 0.6`)
- Interval: 400ms
- Particles per spawn: 2–4

**New Behavior:**
Scale spawn frequency with `visibilityPercent`:
- At 0% visibility: 40% chance per 400ms (baseline)
- At 100% visibility: 80% chance per 400ms (double frequency)
- Linear interpolation between 0–100%

Formula: `spawnChance = 0.4 + (visibilityPercent / 100) * 0.4`

**Rationale:** More particles feel chaotic and energetic during hero view; baseline quietness when hero is off-screen.

---

## Particle Glow & Size

**Current State:**
- Size: 3–9px
- No dynamic glow effect

**New Behavior:**
- **Size scaling:** Size range expands with visibility
  - At 0% visibility: 3–9px (baseline)
  - At 100% visibility: 6–15px (larger range, pushed toward bigger)
  - Formula: `size = Math.random() * (3 + visibilityPercent / 100 * 9) + 3`

- **Glow intensity:** Apply dynamic CSS `filter` and `box-shadow` to particles
  - At 0% visibility: minimal glow (e.g., `filter: none; box-shadow: none`)
  - At 100% visibility: intense glow (e.g., `filter: blur(1px); box-shadow: 0 0 ${8 + intensity * 12}px currentColor`)
  - Particle color already set; glow uses `currentColor` to match

**Implementation Point:**
Set particle styles in the spawn loop:
```javascript
const glowIntensity = visibilityPercent / 100;
particle.style.filter = `blur(${glowIntensity * 1}px)`;
particle.style.boxShadow = `0 0 ${8 + glowIntensity * 12}px ${color}`;
particle.style.setProperty('--glow', glowIntensity);
```

---

## Behavior Outside Hero

- **While hero is visible (0 < visibilityPercent < 100):** Intensity scales smoothly
- **Hero completely on-screen (visibilityPercent = 100):** Max intensity maintained
- **Hero scrolls off (visibilityPercent = 0):** Particles return to baseline spawn rate and glow
- **Particle lifetime unaffected:** Existing particles still live 2.5s and animate out; only *new* spawns scale with intensity

---

## Data Flow

```
Scroll Event
  ↓
Calculate visibilityPercent (hero bounds vs. viewport)
  ↓
Update featherImage transform (1.2x multiplier)
  ↓
Update spawnChance and particle glow parameters
  ↓
Next 400ms interval spawns particles with scaled size/glow
```

---

## Edge Cases & Constraints

- **Hero position:** Assumes hero section has a fixed or predictable DOM position; uses `getBoundingClientRect()` or `offsetTop` to calculate bounds
- **Mobile responsiveness:** Visibility calculation is viewport-agnostic; works on all screen sizes
- **Performance:** Scroll event fires frequently; visibility calculation is O(1), no expensive DOM traversals
- **Particle cleanup:** Particles still auto-remove after 2.5s; no memory leaks
- **Re-entrancy:** If user scrolls rapidly up/down, intensity scales smoothly without jumps (linear interpolation between frames)

---

## Success Criteria

- ✅ Feather visually sinks faster (1.2x parallax)
- ✅ Particles spawn more frequently as hero fills viewport
- ✅ Particles grow larger and glow brighter near 100% visibility
- ✅ Effect naturally fades when hero scrolls off
- ✅ No visual jank or performance degradation on scroll
- ✅ Works on mobile and desktop viewports

---

## Files to Modify

- `somnium-reborn.html`
  - Scroll event handler: update featherImage transform multiplier
  - Particle spawn loop: calculate visibilityPercent, scale spawn chance, size, and glow
