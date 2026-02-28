# Scroll-Driven Animation Patterns (GSAP + Lenis)

Patterns for scroll-triggered reveals, smooth scroll, text splitting, SVG draw-in, pinned sections, and scroll progress. Use on multi-section pages (reviews, recaps, walkthroughs, narratives) where below-the-fold content should animate on scroll rather than on load.

**When to use:** Multi-section pages (4+ sections), walkthrough/narrative pages, any page where scroll pacing enhances comprehension. Single-section diagrams should stick with CSS load animations from `css-patterns.md`.

**When not to use:** Single-section pages, slide decks (they have their own engine in `slide-patterns.md`), pages where content must be immediately visible without scrolling.

## CDN Imports

Add these in `<head>` or before `</body>`. All GSAP plugins are free since Webflow's 2024 acquisition — no license restrictions.

```html
<!-- Lenis smooth scroll (2KB) -->
<script src="https://cdn.jsdelivr.net/npm/lenis@1.3.17/dist/lenis.min.js"></script>

<!-- GSAP core + plugins -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/ScrollTrigger.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/SplitText.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/DrawSVGPlugin.min.js"></script>
```

Only import the plugins you actually use. Core + ScrollTrigger is the minimum for scroll reveals. Add SplitText only for hero text effects, DrawSVG only for SVG path animations.

## Lenis + ScrollTrigger Sync

Standard integration pattern. Place once per page, after all CDN imports:

```javascript
// Initialize Lenis with manual RAF — GSAP controls the loop
const lenis = new Lenis({
  autoRaf: false,
  lerp: 0.08,       // scroll smoothness (0.05 = silky, 0.12 = snappy)
  duration: 1.2,     // scroll duration
  orientation: 'vertical',
  gestureOrientation: 'vertical',
  smoothWheel: true,
  touchMultiplier: 2,
});

// Sync Lenis scroll position with ScrollTrigger
lenis.on('scroll', ScrollTrigger.update);

// Tie Lenis to GSAP's ticker for frame-perfect sync
gsap.ticker.add((time) => {
  lenis.raf(time * 1000);
});

// Disable GSAP lag smoothing — Lenis handles its own timing
gsap.ticker.lagSmoothing(0);
```

**Critical:** Always set `autoRaf: false` when using with GSAP. Double-RAF causes jittery scroll.

## Mermaid + Lenis Coexistence

Mermaid zoom containers use scroll internally (Ctrl+scroll to zoom, overflow:auto for panning). Lenis must not intercept scroll events inside these containers.

```html
<!-- Add data-lenis-prevent to every .mermaid-wrap -->
<div class="mermaid-wrap" data-lenis-prevent>
  <div class="zoom-controls">...</div>
  <pre class="mermaid">...</pre>
</div>
```

`data-lenis-prevent` tells Lenis to ignore scroll events originating from this element, letting the browser handle them natively. Without this, Ctrl+scroll zoom and overflow scrolling inside Mermaid diagrams break.

Also add `data-lenis-prevent` to any horizontally-scrollable container (wide tables, code blocks with overflow):

```html
<div class="table-scroll" data-lenis-prevent>
  <table class="data-table">...</table>
</div>
```

## Pattern 1: Scroll-Triggered Section Reveals

Replace CSS load-time `fadeUp` animations for below-the-fold content. Elements animate in as they enter the viewport.

### CSS — Initial Hidden State

```css
/* Elements start invisible — GSAP reveals them on scroll */
.gs-reveal {
  opacity: 0;
  transform: translateY(24px);
  will-change: opacity, transform;
}

/* Variant: scale-reveal for KPI cards and badges */
.gs-reveal-scale {
  opacity: 0;
  transform: scale(0.92);
  will-change: opacity, transform;
}

/* Ensure visibility if JS fails or reduced-motion is active */
@media (prefers-reduced-motion: reduce) {
  .gs-reveal,
  .gs-reveal-scale {
    opacity: 1 !important;
    transform: none !important;
  }
}
```

### JavaScript

```javascript
// Batch reveal — efficient for many cards/sections
ScrollTrigger.batch('.gs-reveal', {
  onEnter: (batch) => {
    gsap.to(batch, {
      opacity: 1,
      y: 0,
      duration: 0.6,
      ease: 'power2.out',
      stagger: 0.08,
    });
  },
  start: 'top 85%',   // trigger when top of element hits 85% of viewport
  once: true,          // animate once, don't reverse on scroll up
});

// Scale-reveal for KPIs
ScrollTrigger.batch('.gs-reveal-scale', {
  onEnter: (batch) => {
    gsap.to(batch, {
      opacity: 1,
      scale: 1,
      duration: 0.5,
      ease: 'back.out(1.4)',
      stagger: 0.06,
    });
  },
  start: 'top 85%',
  once: true,
});
```

### HTML Usage

```html
<!-- Section cards -->
<div class="ve-card gs-reveal">Section content...</div>
<div class="ve-card gs-reveal">More content...</div>

<!-- KPI row -->
<div class="kpi-row">
  <div class="kpi-card gs-reveal-scale">...</div>
  <div class="kpi-card gs-reveal-scale">...</div>
  <div class="kpi-card gs-reveal-scale">...</div>
</div>
```

**Note:** Don't use both `gs-reveal` and the CSS `fadeUp` animation on the same element. For above-the-fold content that should animate on load, keep using CSS `fadeUp` from `css-patterns.md`. Use `gs-reveal` for everything below the fold.

## Pattern 2: Smooth Scroll Navigation (Lenis)

Replaces native `scrollIntoView({ behavior: 'smooth' })` for consistent cross-browser smooth scroll with momentum easing.

### JavaScript — TOC Click Handler

Replace the scroll spy click handler from `responsive-nav.md`:

```javascript
// TOC smooth scroll via Lenis (replaces scrollIntoView)
document.querySelectorAll('.toc a').forEach(link => {
  link.addEventListener('click', (e) => {
    e.preventDefault();
    const id = link.getAttribute('href').slice(1);
    const target = document.getElementById(id);
    if (target) {
      lenis.scrollTo(target, {
        offset: -20,         // breathing room above section
        duration: 1.2,       // scroll duration in seconds
        easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // expo out
      });
      history.replaceState(null, '', '#' + id);
    }
  });
});
```

The scroll spy observer from `responsive-nav.md` works unchanged — it only observes intersection, which Lenis doesn't affect.

## Pattern 3: Hero Text Reveals (SplitText)

Split hero headings into characters or words, then stagger-animate them for an editorial entrance.

### JavaScript

```javascript
// Split the hero heading into words and characters
const heroHeading = document.querySelector('.hero-title');
if (heroHeading) {
  const split = new SplitText(heroHeading, {
    type: 'words,chars',
    wordsClass: 'gs-word',
    charsClass: 'gs-char',
  });

  // Stagger characters in from below
  gsap.from(split.chars, {
    y: 40,
    opacity: 0,
    duration: 0.6,
    ease: 'power3.out',
    stagger: 0.02,       // 20ms between each character
    delay: 0.3,          // wait for page to settle
  });
}
```

### CSS

```css
/* SplitText wraps create inline-block elements — ensure they wrap properly */
.gs-word {
  display: inline-block;
  overflow: hidden;  /* clips the translateY animation */
}

.gs-char {
  display: inline-block;
  will-change: transform, opacity;
}

/* Reduced motion: skip the split animation entirely */
@media (prefers-reduced-motion: reduce) {
  .gs-word, .gs-char {
    opacity: 1 !important;
    transform: none !important;
  }
}
```

### HTML

```html
<h1 class="hero-title">Authentication System Architecture</h1>
```

**Variants:**
- Word-level stagger: Use `split.words` instead of `split.chars`, with `stagger: 0.05` for a calmer feel
- Scroll-triggered: Wrap in a ScrollTrigger for headings that appear on scroll:

```javascript
ScrollTrigger.create({
  trigger: '.section-heading',
  start: 'top 80%',
  once: true,
  onEnter: () => {
    const split = new SplitText('.section-heading', { type: 'words' });
    gsap.from(split.words, {
      y: 30, opacity: 0, duration: 0.5,
      ease: 'power2.out', stagger: 0.04,
    });
  },
});
```

## Pattern 4: Mermaid Diagram Progressive Reveal (DrawSVG)

Animate SVG edge paths drawing in on scroll. Nodes fade in first, then edges draw between them.

### JavaScript

```javascript
// Wait for Mermaid to render, then animate
function animateMermaidDiagram(wrapSelector) {
  const wrap = document.querySelector(wrapSelector);
  if (!wrap) return;

  // Get all edge paths and node shapes
  const edges = wrap.querySelectorAll('.edge-pattern-solid, .flowchart-link');
  const nodes = wrap.querySelectorAll('.node rect, .node circle, .node polygon');
  const nodeLabels = wrap.querySelectorAll('.nodeLabel');

  // Set initial states
  gsap.set(nodes, { opacity: 0, scale: 0.8, transformOrigin: 'center center' });
  gsap.set(nodeLabels, { opacity: 0 });
  gsap.set(edges, { drawSVG: '0%' });

  // Create scroll-triggered timeline
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: wrap,
      start: 'top 75%',
      once: true,
    },
  });

  // Phase 1: Nodes fade in
  tl.to(nodes, {
    opacity: 1,
    scale: 1,
    duration: 0.4,
    ease: 'power2.out',
    stagger: 0.06,
  });

  // Phase 2: Labels appear
  tl.to(nodeLabels, {
    opacity: 1,
    duration: 0.3,
    stagger: 0.04,
  }, '-=0.2');

  // Phase 3: Edges draw in
  tl.to(edges, {
    drawSVG: '100%',
    duration: 0.8,
    ease: 'power1.inOut',
    stagger: 0.1,
  }, '-=0.1');
}

// Call after mermaid.run() completes
// CRITICAL: Mermaid changes page height — refresh ScrollTrigger so batch
// reveals fire at the correct scroll positions (prevents gs-reveal misses
// on sections below diagrams)
mermaid.run().then(() => {
  ScrollTrigger.refresh();
  animateMermaidDiagram('.mermaid-wrap');
});
```

### Reduced Motion Fallback

```javascript
const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

if (prefersReduced) {
  // Skip animation — everything visible immediately
  // Don't set initial hidden states, don't create timelines
} else {
  animateMermaidDiagram('.mermaid-wrap');
}
```

## Pattern 5: Pinned Comparison Panels

Lock a "before" panel in place while the "after" panel scrolls in beside it. Used for diff reviews, migration guides, and before/after comparisons.

### HTML Structure

```html
<section class="pinned-comparison">
  <div class="pinned-comparison__before">
    <div class="ve-card">
      <div class="ve-card__label" style="color: var(--red);">Before</div>
      <!-- Before content -->
    </div>
  </div>
  <div class="pinned-comparison__after">
    <div class="ve-card">
      <div class="ve-card__label" style="color: var(--green);">After</div>
      <!-- After content -->
    </div>
  </div>
</section>
```

### CSS

```css
.pinned-comparison {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  min-height: 100vh; /* enough height for the pin to work */
}

.pinned-comparison__before,
.pinned-comparison__after {
  min-width: 0;
}

/* Mobile: stack vertically, no pinning */
@media (max-width: 768px) {
  .pinned-comparison {
    grid-template-columns: 1fr;
    min-height: auto;
  }
}
```

### JavaScript

```javascript
// Pin the "before" panel while scrolling through the comparison
if (window.innerWidth > 768) {
  ScrollTrigger.create({
    trigger: '.pinned-comparison',
    start: 'top 20%',
    end: 'bottom 80%',
    pin: '.pinned-comparison__before',
    pinSpacing: false,
  });

  // Animate the "after" panel sliding in
  gsap.from('.pinned-comparison__after', {
    x: 60,
    opacity: 0,
    duration: 0.8,
    ease: 'power2.out',
    scrollTrigger: {
      trigger: '.pinned-comparison__after',
      start: 'top 80%',
      once: true,
    },
  });
}
```

**Anti-scroll-jacking rule:** Never pin for more than 3× viewport height. Always show a visible scroll indicator on pinned sections so the user knows scrolling is progressing. If a comparison is short, skip pinning entirely — it only helps when content needs extended comparison time.

### Scroll Indicator for Pinned Sections

```html
<div class="pin-indicator">
  <div class="pin-indicator__track">
    <div class="pin-indicator__fill"></div>
  </div>
  <span class="pin-indicator__label">Scroll to compare</span>
</div>
```

```css
.pin-indicator {
  position: fixed;
  right: 24px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  z-index: 50;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.pin-indicator.visible { opacity: 1; }

.pin-indicator__track {
  width: 3px;
  height: 60px;
  background: var(--border);
  border-radius: 2px;
  overflow: hidden;
}

.pin-indicator__fill {
  width: 100%;
  height: 0%;
  background: var(--accent);
  border-radius: 2px;
  transition: height 0.1s linear;
}

.pin-indicator__label {
  font-family: var(--font-mono);
  font-size: 9px;
  color: var(--text-dim);
  text-transform: uppercase;
  letter-spacing: 1px;
  writing-mode: vertical-rl;
}
```

```javascript
// Drive indicator fill with ScrollTrigger scrub
const indicator = document.querySelector('.pin-indicator');
const fill = document.querySelector('.pin-indicator__fill');

if (indicator && fill) {
  ScrollTrigger.create({
    trigger: '.pinned-comparison',
    start: 'top 20%',
    end: 'bottom 80%',
    onUpdate: (self) => {
      fill.style.height = (self.progress * 100) + '%';
    },
    onToggle: (self) => {
      indicator.classList.toggle('visible', self.isActive);
    },
  });
}
```

## Pattern 6: Scroll Progress Indicator

Thin progress bar at the top of the viewport showing reading progress.

### CSS

```css
.scroll-progress {
  position: fixed;
  top: 0;
  left: 0;
  width: 0%;
  height: 3px;
  background: var(--accent);
  z-index: 9999;
  pointer-events: none;
  transition: none; /* GSAP handles the animation */
}

/* Optional: gradient variant */
.scroll-progress--gradient {
  background: linear-gradient(90deg, var(--accent), var(--node-b, var(--accent)));
}
```

### HTML

```html
<div class="scroll-progress"></div>
```

### JavaScript

```javascript
// ScrollTrigger-driven progress bar
gsap.to('.scroll-progress', {
  width: '100%',
  ease: 'none',
  scrollTrigger: {
    trigger: document.body,
    start: 'top top',
    end: 'bottom bottom',
    scrub: 0.3,   // slight smoothing
  },
});
```

## Complete Boilerplate

Full setup for a multi-section scroll-driven page. Copy and adapt.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
  <link href="https://fonts.googleapis.com/css2?family=...&display=swap" rel="stylesheet">
  <!-- Lenis -->
  <script src="https://cdn.jsdelivr.net/npm/lenis@1.3.17/dist/lenis.min.js"></script>
  <style>
    /* Theme variables, layout, components — see css-patterns.md */

    /* Scroll animation initial states */
    .gs-reveal { opacity: 0; transform: translateY(24px); will-change: opacity, transform; }
    .gs-reveal-scale { opacity: 0; transform: scale(0.92); will-change: opacity, transform; }
    .scroll-progress { position: fixed; top: 0; left: 0; width: 0%; height: 3px; background: var(--accent); z-index: 9999; pointer-events: none; }

    @media (prefers-reduced-motion: reduce) {
      .gs-reveal, .gs-reveal-scale { opacity: 1 !important; transform: none !important; }
      .scroll-progress { display: none; }
    }
  </style>
</head>
<body>

<div class="scroll-progress"></div>

<!-- Page content with .gs-reveal on below-the-fold elements -->

<!-- GSAP core + plugins (before </body>) -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/ScrollTrigger.min.js"></script>
<!-- Add only if needed: -->
<!-- <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/SplitText.min.js"></script> -->
<!-- <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/DrawSVGPlugin.min.js"></script> -->

<script>
(function() {
  const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  // --- Lenis smooth scroll ---
  let lenis;
  if (!prefersReduced) {
    lenis = new Lenis({ autoRaf: false, lerp: 0.08 });
    lenis.on('scroll', ScrollTrigger.update);
    gsap.ticker.add((time) => { lenis.raf(time * 1000); });
    gsap.ticker.lagSmoothing(0);
  }

  // --- Scroll progress bar ---
  if (!prefersReduced) {
    gsap.to('.scroll-progress', {
      width: '100%',
      ease: 'none',
      scrollTrigger: { trigger: document.body, start: 'top top', end: 'bottom bottom', scrub: 0.3 },
    });
  }

  // --- Section reveals ---
  // IMPORTANT: If page has Mermaid diagrams, defer batch setup until after
  // mermaid.run() completes + ScrollTrigger.refresh(). Mermaid changes page
  // height, making batch trigger positions stale. Without this, gs-reveal
  // elements below diagrams may not appear on first scroll.
  function initBatchReveals() {
    ScrollTrigger.batch('.gs-reveal', {
      onEnter: (batch) => {
        gsap.to(batch, { opacity: 1, y: 0, duration: 0.6, ease: 'power2.out', stagger: 0.08 });
      },
      start: 'top 85%',
      once: true,
    });

    ScrollTrigger.batch('.gs-reveal-scale', {
      onEnter: (batch) => {
        gsap.to(batch, { opacity: 1, scale: 1, duration: 0.5, ease: 'back.out(1.4)', stagger: 0.06 });
      },
      start: 'top 85%',
      once: true,
    });
  }
  if (!prefersReduced) {
    // If Mermaid is present, wait for it to finish rendering
    if (document.querySelector('.mermaid')) {
      window.addEventListener('mermaid-ready', () => {
        ScrollTrigger.refresh();
        initBatchReveals();
      }, { once: true });
      // Fallback in case event is missed
      setTimeout(() => {
        if (!document.querySelector('.gs-reveal[style*="opacity: 1"]')) {
          ScrollTrigger.refresh();
          initBatchReveals();
        }
      }, 3000);
    } else {
      initBatchReveals();
    }
  }

  // --- TOC smooth scroll (if TOC exists) ---
  if (lenis) {
    document.querySelectorAll('.toc a').forEach(link => {
      link.addEventListener('click', (e) => {
        e.preventDefault();
        const id = link.getAttribute('href').slice(1);
        const target = document.getElementById(id);
        if (target) {
          lenis.scrollTo(target, { offset: -20, duration: 1.2 });
          history.replaceState(null, '', '#' + id);
        }
      });
    });
  }
})();
</script>
</body>
</html>
```

## Reduced Motion Strategy

Every scroll-driven page **must** check `prefers-reduced-motion` and degrade gracefully:

1. **Skip Lenis** — use native scroll (no `new Lenis()`)
2. **Skip ScrollTrigger animations** — use `toggleActions` with instant durations, or don't create them at all
3. **Set durations to 0** if using `gsap.to()` directly
4. **Still pin sections** but without animations — pinning is a layout feature, not a motion feature
5. **Hide the scroll progress bar** — it's a decorative indicator
6. **Show all content immediately** — CSS fallback ensures `gs-reveal` elements are visible

The complete boilerplate above demonstrates the pattern: wrap all GSAP setup in `if (!prefersReduced)` and ensure CSS includes the `prefers-reduced-motion: reduce` overrides.

## Anti-Scroll-Jacking Rules

These rules are strictly enforced in `SKILL.md`:

1. **Never remove the user's ability to scroll past a section.** Pinned sections must eventually release. No infinite scroll traps.
2. **Max pin duration: 3x viewport height.** If you need more, break the content into separate pinned sections with unpinned content between them.
3. **Visible scroll indicator on every pinned section.** The user must know that scrolling is doing something even when the viewport isn't moving. Use the pin indicator pattern above.
4. **No full-page scroll hijacking.** Lenis smooths scroll — it doesn't replace it. The user's scroll gesture must always produce visible movement.
5. **`scrub` values should be low (0.1–0.5).** High scrub values make animations feel laggy and disconnected from scroll input.
6. **Touch devices: test momentum.** Lenis handles touch, but pinned sections can feel sticky on mobile. Consider disabling pins on viewports below 768px.

## When to Use What

| Page type | Animation approach |
|---|---|
| Single-section diagram | CSS load animations (`fadeUp`, `fadeScale` from `css-patterns.md`) |
| Multi-section page (4+ sections) | GSAP ScrollTrigger reveals (`gs-reveal`) + Lenis smooth scroll |
| Walkthrough / narrative page | Full GSAP stack: Lenis + ScrollTrigger + pin + scrub + SplitText |
| Slide deck | Slide engine from `slide-patterns.md` (not GSAP) |
| Data table / comparison | CSS load animations + optional scroll reveals for below-fold sections |
