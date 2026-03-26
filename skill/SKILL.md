---
name: design-system-extractor
description: >
  Extracts a complete design system from any live website. Crawls pages using
  Firecrawl MCP, pulls colors, typography, spacing, shadows, radii, and
  gradients, then maps everything to a 3-layer token hierarchy (primitive,
  semantic, component). Outputs CSS custom properties, Style Dictionary JSON,
  a human-readable report, and a self-contained preview HTML that uses its own
  extracted tokens for styling.

  Triggers on: "extract design system", "reverse engineer design tokens",
  "get color palette from site", "crawl site for styles", "what design system
  does X use", "extract tokens from website", "what fonts does X use",
  "pull styles from URL", "scrape design tokens", "analyze site colors",
  "get typography from website", "reverse engineer CSS from site",
  "what colors does X use", "extract brand colors"
---

# Design System Extractor

Extract a structured design system from any live website using Firecrawl MCP.

## Prerequisites

**Firecrawl MCP must be configured and accessible.**

Before running the pipeline, verify Firecrawl is available:

1. Attempt to call `firecrawl_scrape` with a known URL (e.g., `https://example.com`)
   with `formats: ["branding"]`.
2. If the call fails with a connection or auth error, stop and print:
   ```
   ERROR: Firecrawl MCP is not available.
   To configure it, add the Firecrawl MCP server to your Claude Code config:
   https://docs.firecrawl.dev/integrations/claude-desktop
   Ensure FIRECRAWL_API_KEY is set in your environment.
   ```
3. Do NOT proceed with any extraction steps until Firecrawl responds successfully.

## Arguments

| Argument          | Required | Default                  | Description                                                  |
| ----------------- | -------- | ------------------------ | ------------------------------------------------------------ |
| `URL`             | Yes      | -                        | Target website URL (e.g., `https://example.com`)             |
| `--max-pages`     | No       | `5`                      | Maximum pages to scrape (1-20). Homepage is always included. |
| `--output-dir`    | No       | `./design-system-output` | Directory for output files                                   |
| `--output-format` | No       | `all`                    | Output formats: `css`, `json`, `all`                         |

## Output Files

| File                         | Description                                                                             |
| ---------------------------- | --------------------------------------------------------------------------------------- |
| `design-system.css`          | CSS custom properties organized in 3 `:root` blocks (primitives, semantics, components) |
| `design-system.json`         | Style Dictionary / Tokens Studio compatible JSON with `$value` and `$type` fields       |
| `design-system-report.md`    | Full analysis: color palettes, type scale, spacing scale, extraction notes, failures    |
| `design-system-preview.html` | Self-contained HTML preview that uses the extracted tokens for its own styling          |

## Extraction Pipeline

Follow these steps in order. If a step fails, output whatever was successfully
extracted up to that point with a clear note about where extraction stopped and why.

---

### Step 1: Discover and Scrape

**1a. Discover URLs**

Call `firecrawl_map` with the target URL to get a list of internal URLs.

From the results, select a representative sample:

- Always include the homepage
- Pick up to `--max-pages - 1` additional pages that look visually distinct
  (prefer pages with different URL path prefixes: `/about`, `/pricing`,
  `/blog/...`, `/docs/...`, `/contact`, etc.)
- Skip anchors, query params, pagination, and asset URLs

**1b. Scrape each page**

For each selected URL, call `firecrawl_scrape` with:

```
formats: ["branding", "rawHtml", "screenshot"]
```

This returns three things per page:

- **branding**: Structured extraction of colors, fonts, typography, spacing, components
- **rawHtml**: Full page HTML source
- **screenshot**: Visual capture of the page

Store all results keyed by URL.

**1c. Fetch external stylesheets**

From each page's `rawHtml`, find all `<link rel="stylesheet" href="...">` tags.
Deduplicate stylesheet URLs across all pages. For each unique stylesheet URL,
call `firecrawl_scrape` with `formats: ["rawHtml"]` to retrieve the raw CSS text.

Also extract any inline `<style>` blocks from the HTML.

Collect all CSS text (external + inline) into a single corpus for Step 2.

---

### Step 2: Extract Raw Values

Extract values from TWO sources, then cross-reference.

**Source A: Branding data**

From each page's `branding` output, collect:

- All color values (hex, rgb, hsl, named colors)
- Font families (full stacks)
- Font sizes, font weights, line heights, letter spacing
- Spacing values (margin, padding, gap)
- Border radius values
- Box shadow declarations
- Gradient definitions
- Any component patterns identified

**Source B: Raw CSS parsing**

From the collected CSS corpus (external stylesheets + inline styles), extract:

- CSS custom properties defined on `:root`, `html`, or `body`
  (both property names and values)
- All color values from `color`, `background-color`, `border-color`,
  `outline-color`, `fill`, `stroke` properties
- All font declarations: `font-family`, `font-size`, `font-weight`,
  `line-height`, `letter-spacing`
- All spacing values from `margin`, `padding`, `gap`, `row-gap`, `column-gap`
- `border-radius` values
- `box-shadow` values
- `@media` query breakpoint values
- `@font-face` declarations (font names + sources)
- Gradient values from `background`, `background-image`

**Cross-reference:** Values that appear in BOTH Source A and Source B get a
confidence boost. Track the source(s) for every extracted value.

---

### Step 3: Normalize and Deduplicate

**Colors:**

- Normalize ALL color values to 6-digit lowercase hex (`#rrggbb`)
- Collapse duplicates: `#333`, `#333333`, `rgb(51,51,51)` are the same value
- Sort by HSL hue (0-360), then by lightness to reveal palette ramps
- Group colors into hue families (reds, oranges, yellows, greens, blues,
  purples, neutrals/grays)
- Identify achromatic colors separately (pure blacks, whites, grays where
  saturation < 5%)

**Typography:**

- Collect all unique font-size values, convert to rem (assume 16px base if not
  specified)
- Sort numerically and attempt to detect the type scale ratio by checking
  adjacent size ratios against known scales:
  - Minor second: 1.067
  - Major second: 1.125
  - Minor third: 1.200
  - Major third: 1.250
  - Perfect fourth: 1.333
  - Augmented fourth: 1.414
  - Perfect fifth: 1.500
  - Golden ratio: 1.618
- If no clean ratio is detected, note "custom scale" in the report
- Collect all unique font-family stacks, preserving full fallback chains

**Spacing:**

- Normalize to rem (using 16px base)
- Sort numerically and detect the base unit (commonly 4px/0.25rem or 8px/0.5rem)
- Identify the scale pattern (linear like 4/8/12/16 or geometric like 4/8/16/32)

**Frequency ranking:**

- Count occurrences of each unique value across all pages and stylesheets
- Higher frequency = more likely a core design token
- Values appearing on 1 page only are flagged as potentially incidental

---

### Step 4: Map to 3-Layer Token Hierarchy

#### Layer 1: Primitive Tokens

Assign systematic names to ALL deduplicated values:

| Category       | Naming pattern                   | Example                              |
| -------------- | -------------------------------- | ------------------------------------ |
| Colors         | `--color-{hue}-{shade}`          | `--color-blue-500: #3b82f6`          |
| Neutral colors | `--color-gray-{shade}`           | `--color-gray-900: #111827`          |
| Black/White    | `--color-black`, `--color-white` | `--color-white: #ffffff`             |
| Font families  | `--font-{name}`                  | `--font-sans: "Inter", sans-serif`   |
| Font sizes     | `--font-size-{name}`             | `--font-size-lg: 1.125rem`           |
| Font weights   | `--font-weight-{name}`           | `--font-weight-bold: 700`            |
| Line heights   | `--leading-{name}`               | `--leading-normal: 1.5`              |
| Letter spacing | `--tracking-{name}`              | `--tracking-tight: -0.025em`         |
| Spacing        | `--space-{n}`                    | `--space-4: 1rem`                    |
| Border radius  | `--radius-{name}`                | `--radius-lg: 0.5rem`                |
| Shadows        | `--shadow-{name}`                | `--shadow-md: 0 4px 6px ...`         |
| Breakpoints    | `--breakpoint-{name}`            | `--breakpoint-md: 768px`             |
| Gradients      | `--gradient-{n}`                 | `--gradient-1: linear-gradient(...)` |

For shade numbering, use 50-950 scale (50, 100, 200, ..., 800, 900, 950)
mapping lightest to darkest within each hue family.

For font size names, map to: `xs`, `sm`, `base`, `lg`, `xl`, `2xl`, `3xl`,
`4xl`, `5xl` (extend if needed). Anchor `base` to the most frequent body
text size.

For spacing, use an index scale: `0.5`, `1`, `1.5`, `2`, `3`, `4`, `5`, `6`,
`8`, `10`, `12`, `16`, `20`, `24` (map to nearest).

#### Layer 2: Semantic Tokens

**CONSERVATIVE approach only.** Map semantic meaning ONLY when DOM context
makes the purpose unambiguous:

| Condition                                            | Semantic token             | References             |
| ---------------------------------------------------- | -------------------------- | ---------------------- |
| `color` on `body` or `html`                          | `--text-primary`           | `var(--color-...)`     |
| `background-color` on `body`/`html`                  | `--surface-default`        | `var(--color-...)`     |
| `color` on `a` elements                              | `--interactive-primary`    | `var(--color-...)`     |
| `background-color` on `button`                       | `--interactive-primary-bg` | `var(--color-...)`     |
| `border-color` on most common bordered elements      | `--border-default`         | `var(--color-...)`     |
| Primary heading font-family (if different from body) | `--font-heading`           | `var(--font-...)`      |
| Body text font-family                                | `--font-body`              | `var(--font-...)`      |
| Body text font-size                                  | `--text-base-size`         | `var(--font-size-...)` |

**Under-inference is strictly better than wrong inference.** If you are not
confident about a semantic mapping, leave it as a primitive. Do NOT guess
semantic meaning from frequency alone.

#### Layer 3: Component Tokens

Generate component tokens ONLY when a clear, repeated component pattern is
observed across 3+ pages. Examples:

| Pattern                             | Component tokens                                                      |
| ----------------------------------- | --------------------------------------------------------------------- |
| Identical button styles on 3+ pages | `--button-bg`, `--button-text`, `--button-radius`, `--button-padding` |
| Consistent card pattern on 3+ pages | `--card-bg`, `--card-radius`, `--card-shadow`, `--card-padding`       |
| Repeated badge/tag styles           | `--badge-bg`, `--badge-text`, `--badge-radius`                        |

If insufficient data exists for component tokens, skip this layer entirely
and note it in the report. Do NOT force component tokens from insufficient data.

---

### Step 5: Generate Output Files

Create the `--output-dir` directory if it does not exist.

#### design-system.css

Structure with 3 clearly separated `:root` blocks:

```css
/* ==========================================================================
   Design System — Extracted from {URL}
   Generated: {date}
   Pages analyzed: {count}
   ========================================================================== */

/* --------------------------------------------------------------------------
   Layer 1: Primitive Tokens
   Raw design values. These are the source of truth.
   -------------------------------------------------------------------------- */
:root {
  /* Colors — {hue family} */
  --color-blue-50: #eff6ff;
  --color-blue-100: #dbeafe;
  /* ... */

  /* Typography */
  --font-sans: "Inter", system-ui, sans-serif;
  --font-size-sm: 0.875rem;
  /* ... */

  /* Spacing */
  --space-1: 0.25rem;
  /* ... */

  /* Radius, Shadows, etc. */
}

/* --------------------------------------------------------------------------
   Layer 2: Semantic Tokens
   Purpose-driven aliases. Reference primitives only.
   -------------------------------------------------------------------------- */
:root {
  --text-primary: var(--color-gray-900);
  --surface-default: var(--color-white);
  --interactive-primary: var(--color-blue-600);
  /* ... only tokens with confident DOM-context mapping */
}

/* --------------------------------------------------------------------------
   Layer 3: Component Tokens
   Repeated component patterns. Reference semantic or primitive tokens.
   -------------------------------------------------------------------------- */
:root {
  /* Only populated if patterns found on 3+ pages */
}
```

#### design-system.json

Style Dictionary / Tokens Studio format:

```json
{
  "$schema": "https://design-tokens.github.io/community-group/format/",
  "color": {
    "blue": {
      "500": {
        "$value": "#3b82f6",
        "$type": "color",
        "$description": "Found on N pages, sources: branding, css"
      }
    }
  },
  "font": {
    "sans": {
      "$value": "\"Inter\", system-ui, sans-serif",
      "$type": "fontFamily"
    },
    "size": {
      "base": {
        "$value": "1rem",
        "$type": "dimension"
      }
    }
  },
  "space": {
    "4": {
      "$value": "1rem",
      "$type": "dimension"
    }
  },
  "semantic": {
    "text": {
      "primary": {
        "$value": "{color.gray.900}",
        "$type": "color"
      }
    }
  },
  "component": {}
}
```

Every token must include `$value` and `$type`. Include `$description` with
source attribution (branding, css, or both) and page count.

#### design-system-report.md

A human-readable analysis report containing:

1. **Extraction Summary** — URL, date, pages crawled, total tokens extracted,
   any failures or skipped pages
2. **Color Palette** — Table grouped by hue family, with hex values, shade
   names, frequency counts, and source (branding/css/both)
3. **Typography** — Font families with full stacks, type scale table with
   computed sizes and detected ratio (or "custom"), font weights found
4. **Spacing Scale** — Table with values, detected base unit, scale pattern
5. **Border Radius** — All values found
6. **Shadows** — All shadow declarations
7. **Gradients** — All gradient declarations
8. **Semantic Token Mapping** — Table showing each semantic token, what
   primitive it references, and the DOM context that justified it
9. **Component Tokens** — Patterns found (or "insufficient data")
10. **Unclassified Values** — Values extracted but not mapped to any token
11. **Limitations & Notes** — Framework detection results, extraction gaps,
    CSS-in-JS indicators, anything the user should know

#### design-system-preview.html

A self-contained HTML file that eats its own dog food — it uses the extracted
tokens for ALL its styling. Requirements:

- Embed the full `design-system.css` content in an inline `<style>` block
- If Google Fonts were detected, include `<link>` tags to load them
- If custom `@font-face` sources were found, include those declarations

**Sections to render:**

1. **Header** — Site name/URL, extraction date, styled with extracted tokens
2. **Color Palette** — Swatches grouped by hue family. Each swatch shows the
   color, hex value, and token name. Use CSS grid.
3. **Typography Scale** — Each font size rendered as a sample line in the
   extracted font, showing the token name and computed size. Include both
   Latin text ("The quick brown fox") and CJK text if applicable.
4. **Font Families** — Sample text rendered in each extracted font family
5. **Spacing Scale** — Horizontal bars of graduated width, each labeled with
   the token name and value
6. **Border Radius** — Sample boxes showing each radius value
7. **Shadows** — Sample cards showing each shadow level (progressive elevation)
8. **Gradients** — Sample blocks showing each gradient
9. **Component Examples** — If component tokens exist, render sample buttons,
   badges, or cards using those tokens
10. **Semantic Token Reference** — Visual mapping showing semantic tokens
    pointing to their primitive values

**Styling rules for the preview:**

- Use `var(--token-name)` references everywhere, never raw values
- Body background: `var(--surface-default)` if available, else `#ffffff`
- Body text: `var(--text-primary)` if available, else `#111111`
- Headings: use extracted heading font if detected
- The preview itself is proof that the tokens work

---

### Step 6: Present Results

After generating all files:

1. Print a summary table:
   ```
   File                          Size      Tokens
   design-system.css             4.2 KB    87 custom properties
   design-system.json            6.1 KB    87 tokens
   design-system-report.md       3.8 KB    Full analysis
   design-system-preview.html    12.3 KB   Living preview
   ```

2. Highlight key findings:
   - Number of color hues detected and total color values
   - Type scale ratio (if detected) and font families found
   - Spacing base unit and scale
   - Semantic tokens generated vs skipped
   - Any notable patterns or anomalies

3. Note any limitations:
   - Pages that failed to load
   - CSS-in-JS detected but not fully extracted (Phase 2 feature)
   - Framework detected (useful context for the user)
   - Values that could not be classified

---

## Framework Detection Notes

While analyzing HTML classes and CSS, note indicators of CSS frameworks:

| Indicator                                                                        | Framework                              |
| -------------------------------------------------------------------------------- | -------------------------------------- |
| Classes like `bg-`, `text-`, `p-`, `m-`, `flex`, `grid`, `rounded-`, `shadow-`   | Tailwind CSS                           |
| Structured `:root` custom property system with systematic naming                 | Native CSS custom properties           |
| Very few static CSS declarations + many inline styles or `css-` prefixed classes | CSS-in-JS (styled-components, Emotion) |
| Classes like `MuiButton`, `MuiTypography`                                        | Material UI                            |
| Classes like `chakra-`, `css-` with Emotion hashes                               | Chakra UI                              |
| Classes like `ant-btn`, `ant-card`                                               | Ant Design                             |
| Classes like `btn`, `card`, `container`, `row`, `col`                            | Bootstrap                              |

Report detected frameworks in the report. This is informational only for
Phase 1 — it does not change the extraction behavior.

---

## Error Handling

**Firecrawl unavailable:** Stop before Step 1. Print configuration instructions
(see Prerequisites).

**Page scrape failure:** Skip the failed page, continue with remaining pages.
Note the failure in the report. If ALL pages fail, stop and report the error.

**Stylesheet fetch failure:** Skip unfetchable stylesheets. Note in report.
Continue with whatever CSS was successfully retrieved.

**Insufficient data:** If fewer than 5 unique colors are found across all
sources, warn that the extraction may be incomplete (possible CSS-in-JS site
or heavy dynamic styling). Suggest the user try with more pages.

**Credit exhaustion:** If Firecrawl returns a rate limit or credit error
mid-crawl, immediately save all data collected so far. Generate output files
from partial data. Note the incomplete crawl prominently in the report.

**Output directory issues:** If the output directory cannot be created, fall
back to the current working directory. Warn the user.

**Incremental output:** The pipeline is designed to be incremental. If Step 3
fails after Step 2 succeeded, output the raw extracted values in a simplified
format rather than producing nothing. Always output whatever was successfully
processed.
