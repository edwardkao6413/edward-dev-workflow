---
name: ui-designer
description: >
  Use for ANY frontend UI task — building components, pages, dashboards,
  design systems, web apps, posters, or interactive artifacts. Applies to
  any framework or medium (HTML/CSS, React, Vue, Svelte, vanilla JS, or
  static pages). Trigger whenever the user mentions UI, interface, frontend,
  component, design system, dashboard, landing page, or wants something
  styled or visually crafted. Produces bold, distinctive, accessible
  interfaces on a solid token foundation.
---

# UI Designer

A skill that fuses **bold creative vision** with **systematic design rigour**. Every interface must be simultaneously *unforgettable* and *scalable* — striking aesthetics built on a solid token foundation, with accessibility baked in from the start.

---

## Phase 1 — Design Thinking (Always Do This First)

Before writing a single line of code, answer these four questions:

1. **Purpose** — What problem does this solve? Who uses it, in what context?
2. **Tone** — Commit to a specific aesthetic extreme. Examples:
   - Brutally minimal / editorial cold
   - Maximalist richness / layered visual noise
   - Retro-futuristic / terminal aesthetic
   - Organic / natural materials
   - Luxury / refined restraint
   - Playful / toy-like
   - Art deco / geometric precision
   - Industrial / utilitarian raw

   Pick one. Execute it fully. Never blend vaguely.
3. **Constraints** — Framework, accessibility level, performance, responsive targets, any brand constraints.
4. **Differentiation** — What is the ONE thing someone will remember about this interface?

**CRITICAL**: Intentionality beats intensity. A beautifully restrained minimal design and bold maximalism both work — the sin is blandness, not loudness.

---

**Important**: The CSS patterns and token structures described in this skill are guides and starting points, not standards to impose. If the user already has an existing frontend layout or design system, work within it — only reorganize or replace their structure if they explicitly ask you to.

---

## Phase 2 — Design System Foundation

Establish tokens **before** building components. Use CSS custom properties regardless of framework. All values must be derived from the user's brief, brand, or tone — never assumed.

### 2.1 Color Tokens

Define four categories of color tokens, all driven by the user's palette:

- **Brand** — a primary hue with a range of tints (light) to shades (dark), derived from the user's brand or chosen tone
- **Neutrals** — a greyscale family from near-white to near-black, chosen to complement the brand hue (cool, warm, or pure)
- **Semantic** — success, warning, error, and info colors that maintain WCAG AA contrast on their intended backgrounds
- **Surface** — background and overlay values derived from the neutral family; light and dark theme variants should both be defined

**Accessibility rule**: Normal text must achieve **4.5:1 contrast** against its background. Large text (18px+ bold or 24px+): **3:1**. Verify every color combination before shipping.

### 2.2 Typography Tokens

Choose fonts that serve the tone established in Phase 1. Match display and body fonts to create intentional contrast or harmony — do not default to whatever is installed. Define tokens for:

- **Font families** — separate display, body, and monospace families
- **Type scale** — a consistent size progression (choose a ratio that fits the tone: tight for dense UIs, generous for editorial)
- **Font weights** — only include weights the chosen fonts actually support
- **Line heights** — tight for headings, comfortable for body text, loose for reading-focused layouts

The font pairing should be deliberate. Ask: does this combination reinforce the tone, or contradict it?

### 2.3 Spacing Tokens

Define a spacing scale based on a consistent grid unit derived from the design's density requirements. A tighter unit suits dense data UIs; a larger unit suits editorial or marketing layouts. The scale should cover small (component internals), medium (between elements), and large (between sections) increments — all derived from the same base unit for visual rhythm.

### 2.4 Shadow & Elevation

Define a shadow scale that reflects the visual style. Flat designs may use minimal or no shadows; layered designs may use large diffuse ones. At minimum, define:

- **Low elevation** — subtle separation for cards or inputs
- **Mid elevation** — dropdowns, popovers
- **High elevation** — modals, drawers

Also define transition durations and easing curves that match the tone (snappy for functional UIs, slow and smooth for luxury or editorial).

---

## Phase 3 — Creative Execution

### 3.1 Visual Atmosphere

Never default to a flat solid-colour background. Create depth that matches the tone:

- **Gradient meshes** — multi-stop radial gradients for organic richness
- **Noise / grain overlays** — SVG filter or CSS pseudo-element texture
- **Geometric patterns** — SVG repeating backgrounds, diagonal lines, grids
- **Layered transparencies** — glassmorphism where the tone allows
- **Dramatic shadows** — large diffuse shadows for depth
- **Decorative borders** — thick accent borders, dashed outlines, coloured left-borders

### 3.2 Layout & Spatial Composition

Break from predictable grid layouts:
- **Asymmetry** over centred symmetry
- **Overlap** — let elements cross layer boundaries
- **Diagonal flow** — angled dividers, slanted cards
- **Grid-breaking elements** — hero images that bleed, text that escapes the container
- **Generous negative space** OR **controlled density** — pick one extreme, not the middle

### 3.3 Motion & Micro-interactions

Focus animation on **high-impact moments** — one well-orchestrated entrance beats scattered micro-interactions.

```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.reveal { animation: fadeUp 0.6s var(--transition-slow) both; }
.reveal:nth-child(2) { animation-delay: 0.1s; }
.reveal:nth-child(3) { animation-delay: 0.2s; }
```

Always respect `prefers-reduced-motion`:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Phase 4 — Component Standards

### State Matrix (required for every interactive component)

| State | Required behaviour |
|---|---|
| Default | Base design |
| Hover | Transform + colour shift; never just opacity |
| Active / pressed | Slight inset shadow or scale(0.98) |
| Focus-visible | 2px outline offset, never `outline: none` |
| Disabled | 0.5 opacity + `pointer-events: none` + `cursor: not-allowed` |
| Loading | Skeleton or spinner — never a blank space |
| Error | Red semantic colour + icon + descriptive message |
| Empty | Illustrated or iconographic empty state |

### Accessibility Checklist (non-negotiable)

- [ ] Colour contrast 4.5:1 (normal text) / 3:1 (large text) — WCAG AA
- [ ] All interactive elements reachable via keyboard (Tab / Shift+Tab)
- [ ] Focus indicators visible and styled (not browser default)
- [ ] Touch targets ≥ 44×44px on mobile
- [ ] `aria-label` or visible label on all inputs and icon buttons
- [ ] `role`, `aria-expanded`, `aria-selected` on custom interactive patterns
- [ ] Error messages associated with inputs via `aria-describedby`
- [ ] `prefers-reduced-motion` respected

### Responsive Strategy

Mobile-first. Breakpoints:

```css
/* Base: mobile 320–639px */
@media (min-width: 640px)  { /* sm  — tablet portrait */ }
@media (min-width: 768px)  { /* md  — tablet landscape */ }
@media (min-width: 1024px) { /* lg  — desktop */ }
@media (min-width: 1280px) { /* xl  — large desktop */ }
```

---

## Phase 5 — Framework Implementation Notes

Apply the token system above regardless of framework. Adapt patterns to your stack:

### HTML / CSS / Vanilla JS
- Define all tokens in `:root`, consume via `var()`
- CSS-only animations preferred; JS only for complex state
- Use semantic HTML elements (`<nav>`, `<main>`, `<article>`, `<section>`)

### React
- Functional components + hooks
- CSS custom properties for brand tokens; utility classes (e.g. Tailwind) for layout
- Available libraries: `lucide-react`, `recharts`, `framer-motion`, `shadcn/ui`

### Vue / Svelte / Angular
- Same token architecture; use framework-idiomatic component patterns
- Scoped styles are fine as long as tokens are global

### CSS Frameworks (Tailwind, Bootstrap, etc.)
- Extend the framework config to map your custom tokens rather than hardcoding values
- Prefer semantic token names (`--color-primary`) over one-off hex codes in components

### Common import patterns
```js
// React
import { useState, useEffect } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { LineChart, XAxis, YAxis, Tooltip } from "recharts";

// Vanilla
import { createApp } from "vue";
```

---

## Anti-patterns — Never Do These

| Anti-pattern | Why |
|---|---|
| Inter / Roboto / Arial as display font | Signals generic AI output |
| Purple gradient on white background | Overused AI aesthetic cliché |
| Timid, evenly distributed colour palette | Forgettable |
| Predictable card-grid layout | Expected, unmemorable |
| `outline: none` without replacement | Kills keyboard accessibility |
| Flat solid background with no atmosphere | Lacks depth and character |
| Scattered micro-animations everywhere | Distracting; dilutes impact |
| Same aesthetic across every project | Every design should be distinct |

---

## Quick Reference Checklist

Before delivering any output, verify:

- [ ] Bold aesthetic direction chosen and executed fully
- [ ] Design tokens defined (color, type, spacing, shadow)
- [ ] Non-generic font pairing selected
- [ ] All interactive states covered
- [ ] WCAG AA colour contrast confirmed
- [ ] `prefers-reduced-motion` handled
- [ ] Responsive behaviour defined
- [ ] One high-impact animation moment
- [ ] Background has atmosphere/depth, not a flat solid
- [ ] Code is production-grade and functional
