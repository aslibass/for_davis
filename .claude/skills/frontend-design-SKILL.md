---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.
license: Complete terms in LICENSE.txt
---

This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.

The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.

## Design Thinking

Before coding, understand the context and commit to a BOLD aesthetic direction:
- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that is true to the aesthetic direction.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work - the key is intentionality, not intensity.

Then implement working code (HTML/CSS/JS, React, Vue, etc.) that is:
- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail

## Expert Panel

When the work includes a style direction, visual system, or style guide, consult this four-voice panel before finalising the aesthetic choices. The panel should push back on the draft, and any unresolved objections should be fixed in the design rather than merely acknowledged.

### 1. James Fox - Colour

- **Who:** Art historian and broadcaster. *The World According to Color: A Cultural History* (2021); *A History of Art in Three Colours* (BBC).
- **Lens:** Colour carries cultural and emotional baggage. Choosing a hue is choosing what it means, not just how it looks.

Ask:

1. Does the target audience read this colour the way you intend, or are you assuming a universal meaning that does not hold across cultures, generations, or contexts?
2. What is this hue's social register: old wealth, blood, autumn, mourning, fashion, tradition? Does that match the brand's position?
3. Is the palette emotionally coherent, or are you combining colours whose meanings clash beneath the surface?
4. Are you using a colour because it is fashionable, or because it is true to the brand?
5. What is this palette not, and is that exclusion deliberate?

### 2. Erik Spiekermann - Typography

- **Who:** Typographer and designer. Founder of MetaDesign and FontShop; typeface designer of FF Meta, Fira, and ITC Officina.
- **Lens:** Type is a voice. The goal is to make the reader hear the writer, not notice the setting.

Ask:

1. Does this typeface do real work, or is it decoration?
2. Are you pairing fonts because they contrast or because they cooperate?
3. Is the type setting hospitable to reading, or has novelty been prioritised over comfort?
4. Would the brand pick this voice if it walked into the room as a person?
5. Is this font choice fashionable right now, or characterful for this brand specifically?

### 3. Michael Bierut - Brand identity

- **Who:** Pentagram partner since 1990. Designer for MIT, Saks Fifth Avenue, The New York Times, and Hillary Clinton 2016.
- **Lens:** Identity is the visual evidence of a strategy. If the visuals do not make the strategy more legible, they are decoration.

Ask:

1. Could a visitor identify this brand in five seconds without seeing the logo?
2. Does the visual language serve the strategic position, or are you optimising for craft for its own sake?
3. What is the one thing this brand is, and does the design make that one thing more obvious?
4. Is anything here a trend you will regret in 18 months?
5. If you stripped the colour and typography back to black and white at 50% size, does the structure still tell the story?

### 4. Frank Chimero - Web as medium

- **Who:** Designer and writer. *The Shape of Design* (2012); long-running essays on web design as a creative discipline.
- **Lens:** The web has its own grain: it scrolls, responds, links, and behaves. A good site uses that grain rather than fighting it.

Ask:

1. Does this feel like a site, or a brochure pretending to be one?
2. Is the composition doing something print could not, or is it static layout flattened onto a screen?
3. Does the page respond to scroll and viewport as a designed behaviour, not an afterthought?
4. Is there one moment a visitor will remember, and is it a web moment?
5. Does the page reward attention, or is everything visible in the first frame?

### How to use the panel

1. In the style-direction phase, test the draft palette, typography, motion, spatial composition, and component language against the four critiques in order.
2. Where a valid objection is raised and the draft does not answer it, adjust the design.
3. Where a critique is consciously rejected, record the rejection and the reason in the design notes.
4. When generating a `style-guide.md`, append a final section titled `Panel critique` that records, for each panellist, two or three sentences on what they would say about the final decisions.
5. Plan and Build stages should read that `Panel critique` section and must not contradict it without logging the deviation in `decisions.md`.

### Context weighting

- For landing pages with short visitor sessions, prioritise Bierut and Chimero.
- For content-heavy sites, prioritise Spiekermann and Chimero.
- For brand systems intended to scale across products, prioritise Bierut and Fox.
- For single-product or campaign sites, prioritise Fox and Spiekermann.

## Dark Mode

Dark mode is a first-class design decision, not an afterthought or a colour inversion. Decide in the design-thinking phase whether the project defaults to light, dark, or both — then design each theme intentionally.

### Design principles

- **Warm darks beat cold darks.** Pure black (`#000`) feels harsh and lifeless. Dark backgrounds should carry a hint of the brand's hue — warm brown, cool slate, deep navy, forest green. A warm-toned brand uses `#1A1714` not `#000000`.
- **Never invert — remap.** Dark mode is not `filter: invert()`. Each semantic role (background, surface, border, text, muted text) needs its own dark value chosen for that role, not derived mechanically from the light value.
- **Elevation via lightness, not shadow.** In dark mode, cards and surfaces feel raised by being *lighter* than the base, not by casting shadows. Base → surface → card should step up in lightness by 4–6% each.
- **Accent colours rarely need to change.** Burgundy, teal, amber — most accent colours work in both themes. Test contrast; adjust saturation slightly if needed, but avoid maintaining two entirely separate accent palettes.
- **Intentionally dark sections stay dark in both themes.** Hero CTAs, pull quotes, and feature blocks that are deliberately dark (for contrast or drama) should remain dark in dark mode — just slightly darker. Use scoped CSS variable overrides so text inside them stays readable regardless of the global theme.
- **Type contrast must meet WCAG AA at minimum.** Body text on dark backgrounds needs ≥ 4.5:1. Large display headings need ≥ 3:1. Muted text is the most common failure point — test it.

### Expert panel on dark mode

**James Fox (Colour):** Dark palettes carry cultural weight — cinema, luxury, authority, night. Ask whether darkness is a strategic choice for the brand or just a user preference feature. A brand that is equally confident in both modes is stronger than one that treats dark as the "developer option."

**Erik Spiekermann (Typography):** Dark mode demands tighter optical control. Reduce font weight slightly on dark backgrounds — a Regular that reads well on white may look too heavy on dark. Avoid pure white (`#FFF`) text; use a warm or slightly desaturated off-white that matches the background's hue. Line-height tolerances tighten on dark.

**Michael Bierut (Brand identity):** The brand must be unmistakably itself in both themes. If the identity only lands in one mode, the design is incomplete. Test logo, headline colour, and accent against both backgrounds before locking any palette decision.

**Frank Chimero (Web as medium):** Respect `prefers-color-scheme`. The web knows what the user wants — honouring it is good craft. The toggle is a feature, not the feature; the default should already be correct for most visitors.

### Technical implementation (Tailwind + CSS variables)

The only approach that keeps Tailwind opacity modifiers working while allowing runtime theme switching is **RGB channel variables**.

**1. Define variables as RGB triplets (not hex):**

```css
:root {
  --c-bg:      245 240 232;   /* warm ivory */
  --c-surface: 237 229 208;   /* parchment */
  --c-border:  214 201 168;   /* sand */
  --c-text:     44  36  32;   /* charcoal */
  --c-muted:   122 110 102;
  --c-input:   255 255 255;
}

html.dark {
  --c-bg:       26  23  20;   /* warm near-black */
  --c-surface:  35  31  28;   /* elevated surface */
  --c-border:   61  52  46;   /* visible but soft */
  --c-text:    240 235 228;   /* warm off-white */
  --c-muted:   155 145 137;
  --c-input:    42  36  32;
}
```

**2. Map to Tailwind colors using `rgb(var(...) / <alpha-value>)`:**

```js
// tailwind.config.js
colors: {
  bg:      'rgb(var(--c-bg) / <alpha-value>)',
  surface: 'rgb(var(--c-surface) / <alpha-value>)',
  border:  'rgb(var(--c-border) / <alpha-value>)',
  text:    'rgb(var(--c-text) / <alpha-value>)',
  muted:   'rgb(var(--c-muted) / <alpha-value>)',
  input:   'rgb(var(--c-input) / <alpha-value>)',
  // Fixed accent colours that don't change
  accent:  '#C4956A',
}
```

Do NOT use `'var(--c-bg)'` directly — Tailwind resolves it to a hardcoded value at compile time. The `rgb(var(...) / <alpha-value>)` pattern is the only way to get runtime theme switching with opacity modifier support (`bg-muted/50`, etc.).

**3. Intentionally dark sections (hero CTAs, dark cards):**

Scope variable overrides so text stays readable regardless of the global theme:

```css
.dark-section {
  background-color: #2C2420;
  /* Override semantic vars locally so text-text stays light here */
  --c-text:    245 240 232;
  --c-muted:   168 158 150;
  --c-surface:  58  49  44;
}
html.dark .dark-section {
  background-color: #0F0D0B;
}
```

**4. Theme toggle — `useTheme` hook:**

```typescript
// hooks/useTheme.ts
import { useState, useEffect } from 'react'

type Theme = 'light' | 'dark'

export function useTheme() {
  const [theme, setTheme] = useState<Theme>(() => {
    const saved = localStorage.getItem('theme') as Theme | null
    if (saved) return saved
    return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
  })

  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark')
    localStorage.setItem('theme', theme)
  }, [theme])

  return { theme, toggle: () => setTheme(t => t === 'light' ? 'dark' : 'light') }
}
```

**5. Prevent flash on load** — add this to `main.tsx` / `_document.tsx` before React renders:

```typescript
const saved = localStorage.getItem('theme')
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
if (saved === 'dark' || (!saved && prefersDark)) {
  document.documentElement.classList.add('dark')
}
```

**6. Add `darkMode: 'class'` to `tailwind.config.js`** — required for Tailwind's `dark:` variant to work if you also use it alongside the CSS variable approach.

### Common dark mode mistakes

| Mistake | Fix |
| ------- | --- |
| Pure `#000000` background | Use a warm near-black (`#1A1714`, `#0F0E0D`) |
| Pure `#FFFFFF` text | Use warm off-white (`#F0EBE4`, `#E8E3DC`) |
| Inverting accent colours | Test them as-is first; usually they work unchanged |
| `bg-white` on form inputs | Map to a `--c-input` variable that darkens in dark mode |
| Forgetting `color-scheme: dark` on `<html>` | Set it so browser-native UI (scrollbars, selects) follows the theme |
| Flash of wrong theme on load | Apply class from localStorage before React renders |
| Dark-section text going invisible | Use scoped CSS variable overrides, not global `dark:` variants |

## Frontend Aesthetics Guidelines

Focus on:
- **Typography**: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics; unexpected, characterful font choices. Pair a distinctive display font with a refined body font.
- **Color & Theme**: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.
- **Motion**: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.
- **Spatial Composition**: Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.
- **Backgrounds & Visual Details**: Create atmosphere and depth rather than defaulting to solid colors. Add contextual effects and textures that match the overall aesthetic. Apply creative forms like gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, and grain overlays.

NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character.

Interpret creatively and make unexpected choices that feel genuinely designed for the context. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. NEVER converge on common choices (Space Grotesk, for example) across generations.

**IMPORTANT**: Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details. Elegance comes from executing the vision well.

Remember: Claude is capable of extraordinary creative work. Don't hold back, show what can truly be created when thinking outside the box and committing fully to a distinctive vision.
