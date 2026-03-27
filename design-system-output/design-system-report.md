# Design System Report — etoil-digital.com

**Extraction Date:** 2026-03-26
**Pages Analyzed:** 5
**Total Tokens Extracted:** 87 custom properties (47 primitives, 15 semantics, 25 component)
**Framework Detected:** Tailwind CSS v4.1.14 + shadcn/ui + custom `--etoil-*` tokens
**Failures:** None

## Pages Crawled

| # | URL                               | Status |
| - | --------------------------------- | ------ |
| 1 | https://etoil-digital.com/        | OK     |
| 2 | https://etoil-digital.com/product | OK     |
| 3 | https://etoil-digital.com/about   | OK     |
| 4 | https://etoil-digital.com/blog    | OK     |
| 5 | https://etoil-digital.com/contact | OK     |

## External Stylesheets

| URL                                                                                                                                         | Size          | Content                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | -------------------------------------------------------------- |
| `https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;600;700&family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap` | —             | Google Fonts (2 families, 9 weight variants)                   |
| `https://etoil-digital.com/assets/index-Doe63DEZ.css`                                                                                       | 118,938 bytes | Tailwind v4.1.14 utilities + theme tokens + etoil brand tokens |

---

## 1. Color Palette

### Blue (Brand Primary)

| Shade   | Hex           | Token                  | Frequency   | Source                               |
| ------- | ------------- | ---------------------- | ----------- | ------------------------------------ |
| 50      | `#e8f1f8`     | `--color-blue-50`      | CSS         | CSS `--etoil-primary-light`          |
| 300     | `#5a9fd6`     | `--color-blue-300`     | CSS         | CSS `--etoil-primary-gradient` start |
| 400     | `#4da6ff`     | `--color-blue-400`     | 3 pages     | Branding (button shadow rgba)        |
| **500** | **`#478ccb`** | **`--color-blue-500`** | **5 pages** | **Branding + CSS `--etoil-primary`** |
| 700     | `#2d5a8f`     | `--color-blue-700`     | CSS         | CSS `--etoil-primary-dark`           |

### Teal (Accent)

| Shade | Hex       | Token              | Frequency | Source                     |
| ----- | --------- | ------------------ | --------- | -------------------------- |
| 50    | `#e6fff9` | `--color-teal-50`  | CSS       | CSS `--etoil-accent-light` |
| 500   | `#00d4aa` | `--color-teal-500` | CSS       | CSS `--etoil-accent`       |

### Red (Destructive)

| Shade | Hex       | Token             | Frequency | Source                             |
| ----- | --------- | ----------------- | --------- | ---------------------------------- |
| 100   | `#fee2e2` | `--color-red-100` | CSS       | Tailwind `--color-red-100`         |
| 500   | `#ef4444` | `--color-red-500` | CSS       | Tailwind `--color-red-500`         |
| 600   | `#dc2626` | `--color-red-600` | CSS       | shadcn `--destructive` (oklch→hex) |

### Slate (Neutral)

| Shade | Hex       | Token               | Frequency      | Source                                             |
| ----- | --------- | ------------------- | -------------- | -------------------------------------------------- |
| 50    | `#f8fafc` | `--color-slate-50`  | Branding + CSS | `--color-slate-50` + branding bg                   |
| 100   | `#f1f5f9` | `--color-slate-100` | CSS            | Tailwind `--color-slate-100`                       |
| 200   | `#e2e8f0` | `--color-slate-200` | 1 page + CSS   | Branding (contact input border) + `--etoil-border` |
| 500   | `#4a6580` | `--color-slate-500` | 1 page + CSS   | Branding (homepage) + `--etoil-dark-muted`         |
| 600   | `#475569` | `--color-slate-600` | CSS            | Tailwind `--color-slate-600`                       |
| 700   | `#334155` | `--color-slate-700` | CSS            | Tailwind `--color-slate-700`                       |
| 900   | `#1a2b3c` | `--color-slate-900` | 5 pages + CSS  | Branding (all pages text) + `--etoil-dark`         |
| 950   | `#0f172a` | `--color-slate-950` | CSS            | Tailwind `--color-slate-900`                       |

### Achromatic

| Value     | Token           | Source                 |
| --------- | --------------- | ---------------------- |
| `#ffffff` | `--color-white` | 5 pages branding + CSS |
| `#000000` | `--color-black` | CSS `--color-black`    |

---

## 2. Typography

### Font Families

| Token            | Stack                                       | Role            | Source                   |
| ---------------- | ------------------------------------------- | --------------- | ------------------------ |
| `--font-display` | Plus Jakarta Sans, Noto Sans TC, sans-serif | Headings        | CSS + branding (5 pages) |
| `--font-body`    | Noto Sans TC, Plus Jakarta Sans, sans-serif | Body text       | CSS + branding (5 pages) |
| `--font-sans`    | ui-sans-serif, system-ui, sans-serif, ...   | System fallback | CSS                      |
| `--font-mono`    | ui-monospace, SFMono-Regular, Menlo, ...    | Code            | CSS                      |

**Google Fonts loaded:**

- **Noto Sans TC:** weights 300, 400, 500, 600, 700 (CJK Traditional Chinese)
- **Plus Jakarta Sans:** weights 400, 500, 600, 700 (Latin display)

### Type Scale

| Token              | Size (rem) | Size (px) | Usage                  | Source         |
| ------------------ | ---------- | --------- | ---------------------- | -------------- |
| `--font-size-xs`   | 0.75       | 12        | Labels, captions       | CSS            |
| `--font-size-sm`   | 0.875      | 14        | Small text             | CSS            |
| `--font-size-base` | 1          | 16        | Base/default           | CSS            |
| `--font-size-lg`   | 1.125      | 18        | Body text (3/5 pages)  | Branding + CSS |
| `--font-size-xl`   | 1.25       | 20        | Body text (about page) | Branding + CSS |
| `--font-size-2xl`  | 1.5        | 24        | Subheadings            | CSS            |
| `--font-size-3xl`  | 1.875      | 30        | Subheadings            | CSS            |
| `--font-size-4xl`  | 2.25       | 36        | Small headings         | CSS            |
| `--font-size-5xl`  | 2.5        | 40        | h2 (4/5 pages)         | Branding       |
| `--font-size-6xl`  | 3.5        | 56        | h1 (4/5 pages)         | Branding       |

**Scale Analysis:** Tailwind's default scale (xs→4xl) uses a ratio between 1.125 (major second) and 1.333 (perfect fourth) depending on the step. Not a pure mathematical ratio — **custom scale** following Tailwind v4 defaults.

### Font Weights

| Token                    | Value | Fonts Available |
| ------------------------ | ----- | --------------- |
| `--font-weight-light`    | 300   | Noto Sans TC    |
| `--font-weight-normal`   | 400   | Both            |
| `--font-weight-medium`   | 500   | Both            |
| `--font-weight-semibold` | 600   | Both            |
| `--font-weight-bold`     | 700   | Both            |

### Line Heights

| Token               | Value | Source                        |
| ------------------- | ----- | ----------------------------- |
| `--leading-tight`   | 1.2   | CSS `--text-3xl--line-height` |
| `--leading-snug`    | 1.375 | CSS `--leading-snug`          |
| `--leading-normal`  | 1.5   | CSS `--leading-normal`        |
| `--leading-relaxed` | 1.625 | CSS `--leading-relaxed`       |

### Letter Spacing

| Token               | Value    | Source |
| ------------------- | -------- | ------ |
| `--tracking-tight`  | -0.025em | CSS    |
| `--tracking-widest` | 0.1em    | CSS    |

---

## 3. Spacing Scale

**Base Unit:** 0.25rem (4px) — `--spacing: .25rem` in CSS

**Pattern:** Linear multiplier of base unit (Tailwind v4 default)

| Token         | Value (rem) | Value (px) |
| ------------- | ----------- | ---------- |
| `--space-0-5` | 0.125       | 2          |
| `--space-1`   | 0.25        | 4          |
| `--space-1-5` | 0.375       | 6          |
| `--space-2`   | 0.5         | 8          |
| `--space-2-5` | 0.625       | 10         |
| `--space-3`   | 0.75        | 12         |
| `--space-4`   | 1           | 16         |
| `--space-5`   | 1.25        | 20         |
| `--space-6`   | 1.5         | 24         |
| `--space-8`   | 2           | 32         |
| `--space-10`  | 2.5         | 40         |
| `--space-12`  | 3           | 48         |
| `--space-16`  | 4           | 64         |
| `--space-20`  | 5           | 80         |
| `--space-24`  | 6           | 96         |

---

## 4. Border Radius

| Token           | Value           | Usage           | Source              |
| --------------- | --------------- | --------------- | ------------------- |
| `--radius-xs`   | 0.125rem (2px)  | Fine detail     | CSS `--radius-xs`   |
| `--radius-sm`   | 0.25rem (4px)   | Base radius     | Branding (4 pages)  |
| `--radius-md`   | 0.625rem (10px) | Default shadcn  | CSS `--radius`      |
| `--radius-lg`   | 0.75rem (12px)  | Buttons, inputs | Branding (3+ pages) |
| `--radius-full` | 100px           | Pill buttons    | Branding (3 pages)  |

---

## 5. Shadows

| Token           | Value                                         | Usage                        | Source                  |
| --------------- | --------------------------------------------- | ---------------------------- | ----------------------- |
| `--shadow-sm`   | `0 1px 3px #0000000f, 0 1px 2px #0000000a`    | Subtle elevation             | CSS `--etoil-shadow-sm` |
| `--shadow-md`   | `0 4px 12px #00000014, 0 2px 4px #0000000a`   | Cards, panels                | CSS `--etoil-shadow-md` |
| `--shadow-lg`   | `0 20px 60px #4da6ff26, 0 8px 20px #0000000f` | Hero sections (blue-tinted!) | CSS `--etoil-shadow-lg` |
| `--shadow-glow` | `0 4px 15px rgba(77, 166, 255, 0.3)`          | CTA button glow              | Branding (3 pages)      |

**Notable:** `--shadow-lg` and `--shadow-glow` are blue-tinted (`#4da6ff`), creating a branded shadow effect unique to etoil.

---

## 6. Gradients

| Token                | Value                                               | Source                         |
| -------------------- | --------------------------------------------------- | ------------------------------ |
| `--gradient-primary` | `linear-gradient(135deg, #5a9fd6 0%, #478ccb 100%)` | CSS `--etoil-primary-gradient` |

---

## 7. Semantic Token Mapping

| Semantic Token                | Primitive Reference | DOM Context / Justification                                |
| ----------------------------- | ------------------- | ---------------------------------------------------------- |
| `--surface-default`           | `--color-white`     | `background-color` on `body` / `html` (all 5 pages)        |
| `--surface-muted`             | `--color-slate-50`  | Alternating section backgrounds                            |
| `--text-primary`              | `--color-slate-900` | `color` on body text (all 5 pages, matches `--etoil-dark`) |
| `--text-muted`                | `--color-slate-500` | Subtitle/description text (matches `--etoil-dark-muted`)   |
| `--text-on-primary`           | `--color-white`     | White text on blue CTA buttons                             |
| `--interactive-primary`       | `--color-blue-500`  | Links, button backgrounds (matches `--etoil-primary`)      |
| `--interactive-primary-hover` | `--color-blue-700`  | Hover state (matches `--etoil-primary-dark`)               |
| `--interactive-accent`        | `--color-teal-500`  | Accent color (matches `--etoil-accent`)                    |
| `--border-default`            | `--color-slate-200` | Input borders, dividers (matches `--etoil-border`)         |
| `--status-success`            | `--color-teal-500`  | Contact form success checkmark                             |
| `--status-error`              | `--color-red-500`   | Destructive/error states                                   |
| `--font-heading`              | `--font-display`    | Heading elements use Plus Jakarta Sans first               |
| `--font-text`                 | `--font-body`       | Body text uses Noto Sans TC first                          |
| `--text-base-size`            | `--font-size-lg`    | 18px body text (most common, 3/5 pages)                    |
| `--text-heading-1`            | `--font-size-6xl`   | 56px h1 (4/5 pages)                                        |
| `--text-heading-2`            | `--font-size-5xl`   | 40px h2 (4/5 pages)                                        |

---

## 8. Component Tokens

### Button — Primary CTA (Pill)

Observed on: homepage, blog, contact (3+ pages). The "預約 Demo" nav button.

| Property   | Token                     | Value                             |
| ---------- | ------------------------- | --------------------------------- |
| Background | `--button-primary-bg`     | `var(--gradient-primary)`         |
| Text       | `--button-primary-text`   | `var(--text-on-primary)` — white  |
| Radius     | `--button-primary-radius` | `var(--radius-full)` — 100px pill |
| Shadow     | `--button-primary-shadow` | `var(--shadow-glow)` — blue glow  |

### Button — Secondary (Outlined)

Observed on: product, about, contact pages.

| Property   | Token                       | Value                               |
| ---------- | --------------------------- | ----------------------------------- |
| Background | `--button-secondary-bg`     | `transparent`                       |
| Text       | `--button-secondary-text`   | `var(--interactive-primary)` — blue |
| Border     | `--button-secondary-border` | `var(--interactive-primary)` — blue |
| Radius     | `--button-secondary-radius` | `var(--radius-lg)` — 12px           |

### Input

Observed on: contact page. Insufficient pages (1) for high confidence — included for completeness.

| Property   | Token            | Value                             |
| ---------- | ---------------- | --------------------------------- |
| Background | `--input-bg`     | `var(--surface-default)` — white  |
| Text       | `--input-text`   | `var(--text-primary)`             |
| Border     | `--input-border` | `var(--border-default)` — #e2e8f0 |
| Radius     | `--input-radius` | `var(--radius-lg)` — 12px         |

### Navigation

Observed on: all 5 pages.

| Property          | Token              | Value                            |
| ----------------- | ------------------ | -------------------------------- |
| Background        | `--nav-bg`         | `var(--surface-default)` — white |
| Text              | `--nav-text`       | `var(--text-primary)`            |
| CTA Button BG     | `--nav-cta-bg`     | `var(--gradient-primary)`        |
| CTA Button Text   | `--nav-cta-text`   | `var(--text-on-primary)` — white |
| CTA Button Radius | `--nav-cta-radius` | `var(--radius-full)` — pill      |
| CTA Button Shadow | `--nav-cta-shadow` | `var(--shadow-glow)` — blue glow |

---

## 9. Unclassified Values

| Value                                          | Context          | Notes                                                       |
| ---------------------------------------------- | ---------------- | ----------------------------------------------------------- |
| 85 Sonner toast `--gray*` and status tokens    | Inline `<style>` | Library internals (Sonner toast), not part of design system |
| 72 `--tw-*` @property registrations            | External CSS     | Tailwind v4 internal machinery                              |
| `etoil-fadeInUp`, `etoil-float`, `etoil-pulse` | External CSS     | Animation keyframes (not tokenized)                         |

---

## 10. Dual Token System Architecture

This site runs **three concurrent token systems**:

1. **`--etoil-*` brand tokens** (12 properties) — The actual design system. Hex-based colors, custom shadows, and the brand gradient. These define the visual identity.

2. **shadcn/ui tokens** (27 properties) — Default oklch-based neutral grays. These are **NOT customized** to the brand — they use stock shadcn values (achromatic, chroma=0). The brand overrides happen via `--etoil-*` tokens and direct Tailwind class usage.

3. **Tailwind v4 theme tokens** (`--color-*`, `--text-*`, `--spacing`, etc.) — Framework structural tokens. Include a subset of the full Tailwind palette (only slate, red, blue color families are present in the CSS, not the full rainbow).

**Implication:** When extending this design system, use the `--etoil-*` naming convention for brand-specific tokens and Tailwind utilities for structural layout.

---

## 11. Limitations & Notes

- **Light mode only** — no `.dark` `:root` overrides found in CSS
- **CSS-in-JS:** Not detected. All styles come from static CSS file + inline Sonner styles
- **Single external CSS file** — Vite-bundled, hash `Doe63DEZ`, 118KB
- **CJK support** — Noto Sans TC provides full Traditional Chinese coverage
- **Button style confusion** — Firecrawl branding reported `background: #FFFFFF` for CTA buttons on all pages, but visual evidence shows blue gradient backgrounds with white text. The `textColor: #FFFFFF` confirms white-on-blue, but Firecrawl's `background` reading is unreliable for gradient backgrounds
- **Homepage anomalies** — Primary color reported as `#4A6580` (dark muted) rather than `#478CCB`; h2 size reported as `17.6px` instead of `40px`. These are Firecrawl misreads, corrected by cross-referencing with 4 other pages
- **Contact page success state** — Screenshot shows green checkmark animation, "感謝您的預約！" heading, and blue phone link. Success green likely maps to `--etoil-accent` (#00d4aa)
