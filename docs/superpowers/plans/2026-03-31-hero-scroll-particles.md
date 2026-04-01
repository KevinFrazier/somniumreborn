# Hero Scroll & Dynamic Particles Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Enhance hero section with 1.2x parallax feather and viewport-percentage-driven particle intensity (spawn rate, size, glow).

**Architecture:** Single file modification (`somnium-reborn.html`). Add a helper function to calculate hero visibility percentage on every scroll event, then use that metric to scale three particle parameters: spawn chance, size range, and glow/filter effects.

**Tech Stack:** Vanilla JavaScript, existing scroll event listener, existing particle spawn interval.

---

## File Structure

**Modify:** `somnium-reborn.html`
- Scroll event handler section (~line 1925–1945): Add visibility calculation and update featherImage parallax
- Particle spawn interval (~line 1973–2013): Use visibility metric to scale spawn chance, size, and glow

---

## Task 1: Add Hero Visibility Percentage Calculation Helper

**Files:**
- Modify: `somnium-reborn.html` (scroll event section, ~line 1925–1945)

- [ ] **Step 1: Identify the scroll event listener location**

Find the scroll event handler in somnium-reborn.html around line ~1880–1950. Look for `window.addEventListener('scroll', ...)`

- [ ] **Step 2: Add helper function to calculate visibility percentage**

Inside the scroll event listener, before any other calculations, add this helper. Find where `const scrollTop = window.scrollY;` is declared, and add after it:

```javascript
// Calculate hero visibility percentage
const heroSection = document.querySelector('.hero');
let visibilityPercent = 0;

if (heroSection) {
    const heroBounds = heroSection.getBoundingClientRect();
    const viewportHeight = window.innerHeight;

    // Calculate visible height of hero in viewport
    const heroTop = heroBounds.top;
    const heroBottom = heroBounds.bottom;
    const heroHeight = heroBounds.height;

    const visibleTop = Math.max(0, heroTop);
    const visibleBottom = Math.min(viewportHeight, heroBottom);
    const visibleHeight = Math.max(0, visibleBottom - visibleTop);

    visibilityPercent = Math.max(0, Math.min(100, (visibleHeight / heroHeight) * 100));
}
```

- [ ] **Step 3: Verify the calculation is in the right place**

The `visibilityPercent` variable should now be available to all code after it in the scroll handler. Verify it's declared before the featherImage parallax update and particle spawn code.

- [ ] **Step 4: Commit**

```bash
git add "/Users/kevinfrazier/Downloads/files 2/somnium-reborn.html"
git commit -m "feat: add hero visibility percentage calculation on scroll"
```

---

## Task 2: Update Feather Parallax Multiplier to 1.2x

**Files:**
- Modify: `somnium-reborn.html` (scroll event section, ~line 1925–1930)

- [ ] **Step 1: Locate the current feather parallax code**

Find the line that looks like:
```javascript
featherImage.style.transform = `translateY(${scrollTop * 0.8}px)`;
```

This should be around line 1929 in the scroll event handler.

- [ ] **Step 2: Update multiplier from 0.8 to 1.2**

Replace the line with:
```javascript
featherImage.style.transform = `translateY(${scrollTop * 1.2}px)`;
```

- [ ] **Step 3: Test visually**

Open somnium-reborn.html in a browser. Scroll down the page. The feather image should sink faster than before (appears to move down quicker relative to page scroll).

- [ ] **Step 4: Commit**

```bash
git add "/Users/kevinfrazier/Downloads/files 2/somnium-reborn.html"
git commit -m "feat: increase feather parallax multiplier to 1.2x"
```

---

## Task 3: Update Particle Spawn Chance Based on Visibility

**Files:**
- Modify: `somnium-reborn.html` (particle spawn interval, ~line 1979–1982)

- [ ] **Step 1: Locate the particle spawn interval**

Find the code around line 1979 that looks like:
```javascript
setInterval(() => {
    // Random chance to create particles
    if (Math.random() > 0.6) {
```

This is where particles currently spawn with 40% chance (`Math.random() > 0.6` means 40%).

- [ ] **Step 2: Replace spawn chance with visibility-scaled calculation**

Replace the spawn chance check. Change:
```javascript
if (Math.random() > 0.6) {
```

To:
```javascript
const spawnChance = 0.4 + (visibilityPercent / 100) * 0.4;
if (Math.random() < spawnChance) {
```

This scales spawn chance from 40% (at 0% hero visibility) to 80% (at 100% hero visibility).

- [ ] **Step 3: Verify visibilityPercent is in scope**

The `visibilityPercent` variable from Task 1 should be available here since it's calculated in the same scroll event listener. Verify the particle interval is inside the scroll event handler or at least has access to the calculation.

- [ ] **Step 4: Test visually**

Open somnium-reborn.html. Scroll slowly into the hero section. Particles should spawn more frequently as more of the hero becomes visible. Scroll past the hero; particle spawn rate should drop back to baseline.

- [ ] **Step 5: Commit**

```bash
git add "/Users/kevinfrazier/Downloads/files 2/somnium-reborn.html"
git commit -m "feat: scale particle spawn chance with hero visibility"
```

---

## Task 4: Update Particle Size Scaling Based on Visibility

**Files:**
- Modify: `somnium-reborn.html` (particle spawn loop, ~line 1991)

- [ ] **Step 1: Locate the particle size calculation**

Find the line in the particle spawn loop around line 1991:
```javascript
const size = Math.random() * 6 + 3; // 3-9px
```

- [ ] **Step 2: Replace with visibility-scaled size**

Update to:
```javascript
const minSize = 3;
const maxSize = 3 + visibilityPercent / 100 * 9;
const size = Math.random() * (maxSize - minSize) + minSize;
```

This scales particles from 3–9px at 0% visibility to 6–15px at 100% visibility.

- [ ] **Step 3: Test visually**

Open somnium-reborn.html. Scroll into hero. Particles should grow larger as hero visibility increases. When hero scrolls off, new particles spawn at the baseline smaller size again.

- [ ] **Step 4: Commit**

```bash
git add "/Users/kevinfrazier/Downloads/files 2/somnium-reborn.html"
git commit -m "feat: scale particle size with hero visibility"
```

---

## Task 5: Update Particle Glow Based on Visibility

**Files:**
- Modify: `somnium-reborn.html` (particle spawn loop, after line 2002)

- [ ] **Step 1: Locate where particle styles are set**

In the particle spawn loop, find where the particle color is set (around line 1992–2002):
```javascript
particle.style.backgroundColor = color;
particle.style.color = color;
particle.style.setProperty('--tx', tx + 'px');
particle.style.animationDelay = (Math.random() * 0.3) + 's';
```

- [ ] **Step 2: Add glow and filter styles after color assignment**

After the lines above, add:
```javascript
const glowIntensity = visibilityPercent / 100;
particle.style.filter = `blur(${glowIntensity * 1}px)`;
particle.style.boxShadow = `0 0 ${8 + glowIntensity * 12}px ${color}`;
particle.style.setProperty('--glow', glowIntensity);
```

This applies blur and box-shadow glow that scale from minimal (0% visibility) to intense (100% visibility).

- [ ] **Step 3: Test visually**

Open somnium-reborn.html. Scroll into hero. Particles should:
- Start with no visible glow when hero is off-screen
- Gradually brighten and blur as hero fills viewport
- Reach peak glow intensity when hero is fully visible
- Return to minimal glow when hero scrolls off

- [ ] **Step 4: Commit**

```bash
git add "/Users/kevinfrazier/Downloads/files 2/somnium-reborn.html"
git commit -m "feat: add dynamic glow and blur to particles based on hero visibility"
```

---

## Task 6: Full Integration Test

**Files:**
- Test: Visual inspection of `somnium-reborn.html`

- [ ] **Step 1: Open somnium-reborn.html in browser**

Open the file in a web browser (Chrome, Firefox, Safari — any modern browser).

- [ ] **Step 2: Test feather parallax**

Scroll down the page. Verify:
- Feather image sinks faster than the page scroll (1.2x effect)
- Movement is smooth without jank
- Works the same on mobile/tablet viewports

- [ ] **Step 3: Test particle spawn rate**

With hero fully visible, scroll slowly. Count approximate particle spawns per second or observe spawn frequency:
- High spawn rate when hero fills viewport (near 100% visibility)
- Lower spawn rate when hero is partially visible
- Minimal spawn rate when hero scrolls completely off-screen

- [ ] **Step 4: Test particle size**

Particles spawning while hero is visible should appear noticeably larger than particles spawning after you scroll past.

- [ ] **Step 5: Test particle glow**

Particles spawning while hero is fully visible should have bright glow and slight blur:
- At 0% visibility: particles have no glow, no blur
- At 50% visibility: particles have moderate glow (~12px shadow)
- At 100% visibility: particles have intense glow (~20px shadow) and visible blur

- [ ] **Step 6: Test rapid scroll**

Scroll up and down quickly. Intensity should scale smoothly without visual stuttering or jumps.

- [ ] **Step 7: Test mobile responsiveness**

Using browser dev tools, test on mobile viewport sizes (e.g., iPhone 12):
- Feather parallax works
- Particle effects are smooth
- No layout shifts

- [ ] **Step 8: Final verification commit**

If all tests pass:
```bash
git log --oneline -5
# Should show commits from Tasks 1-5
```

---

## Success Criteria Checklist

- ✅ Feather translates at 1.2x scroll speed (Task 2)
- ✅ Particle spawn chance scales with hero visibility 40%–80% (Task 3)
- ✅ Particle size scales from 3–9px to 6–15px (Task 4)
- ✅ Particle glow and blur scale with visibility (Task 5)
- ✅ All effects are smooth on scroll without jank (Task 6)
- ✅ Mobile responsive (Task 6)
- ✅ Each task committed independently (Tasks 1-5)

---

## Notes

- **No CSS changes:** All changes are JavaScript; existing CSS particle animations remain untouched.
- **No breaking changes:** Existing particle cleanup (2.5s timeout) and animation intervals are preserved.
- **Scoped to hero:** Visibility calculation only affects particles while hero section is in viewport; naturally resets when scrolled off.
- **Performance:** Visibility calculation is O(1); scroll event already fires frequently, no new performance bottlenecks introduced.
