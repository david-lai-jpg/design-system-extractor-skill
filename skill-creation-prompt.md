# Design System Extractor — Claude Code Skill Build Prompt

**You are a senior frontend engineer and skill architect. Your task is to design and build a reusable Claude Code skill called `design-system-extractor` that extracts a website's visual design language using the Firecrawl MCP server and outputs a structured design system specification.**

## Architecture

**Orchestration model:** Claude orchestrates the pipeline. Firecrawl does all heavy lifting (scraping, screenshots, branding extraction, browser sessions for CSS-in-JS). No local Python scripts for CSS parsing — Firecrawl's `branding` format and `extract` tool handle structured extraction. Local code is only needed for file I/O and token post-processing.

**Why this works:** Firecrawl's `branding` format natively extracts colors, fonts, typography, spacing, and UI components. Its `browser_*` tools provide full CDP browser sessions that can run `getComputedStyle()` in the cloud — no local Playwright dependency. The `extract` tool accepts custom JSON schemas for LLM-powered structured extraction.

## Execution Environment

- Claude Code running locally
- Full filesystem access, full tool access
- Python dependencies (if any): use `uv pip install` in a virtual environment
- Skill install location: `~/.claude/skills/design-system-extractor/`

## MCP Server: Firecrawl

Firecrawl is available as an MCP server with the following tools:

| Tool                                      | Purpose in this skill                                                                          |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `firecrawl_map`                           | Discover all internal URLs on a domain                                                         |
| `firecrawl_scrape`                        | Single-page scrape with multiple formats (`branding`, `rawHtml`, `screenshot`, `html`, `json`) |
| `firecrawl_extract`                       | LLM-powered structured data extraction with custom JSON schema                                 |
| `firecrawl_crawl`                         | Multi-page crawl with same format options as scrape                                            |
| `firecrawl_browser_create/execute/delete` | Full CDP browser sessions — run JS/Python in a cloud browser                                   |
| `firecrawl_agent`                         | Autonomous research agent (async, for complex multi-site tasks)                                |

**Key format: `branding`** — When passed as a format to `firecrawl_scrape`, extracts brand identity including colors, fonts, typography, spacing, and UI components. This is the primary extraction mechanism for Phase 1.

**Key format: `screenshot`** — Takes page screenshots natively. No local browser needed.

**CRITICAL — MCP Tool Discovery Gate (Do NOT skip this):**
Before writing ANY extraction logic, you MUST:

1. Run MCP tool discovery to list every tool the Firecrawl MCP server exposes
2. Print every tool name and its full parameter schema
3. Save this to `~/.claude/skills/design-system-extractor/firecrawl-mcp-tools.json`
4. Do NOT write any extraction code until this file exists
5. All subsequent code must reference ONLY tool names and parameter shapes found in this file

## Reference Skills

1. **design-system-patterns** (installed skill) — Defines the **output schema**: the 3-layer token hierarchy (primitive -> semantic -> component), CSS custom property architecture, and naming conventions. Invoke this skill or read its SKILL.md to understand the target structure. Your extracted design system must conform to this schema.
2. **skill-creator** (installed skill) — Defines how to structure a Claude Code skill. Follow its conventions for SKILL.md format, description triggers, file layout.

## Phased Implementation

Each phase produces a **usable skill**. Phase 1 is the MVP — ship it, test it, then iterate.

---

### Phase 1 — MVP: Branding Extraction + Token Mapping

**Goal:** Extract design tokens from any website using Firecrawl's native capabilities. No CSS parsing. No framework detection.

#### Step 1: Acquire Raw Assets

1. Use `firecrawl_map` to discover internal URLs on the target domain
2. Select a representative sample: homepage + up to N subpages (controlled by `--max-pages`, default 5) covering visually distinct page types
3. For each selected page, call `firecrawl_scrape` with:
   ```
   formats: ["branding", "rawHtml", "screenshot"]
   ```
   This returns branding data (colors, fonts, spacing, components), raw HTML, and a screenshot — all in one call.

#### Step 2: Extract Raw Values from Branding Data

From each page's `branding` output, collect:

- **Colors**: all color values found. Normalize to hex.
- **Typography**: font-family stacks, font-size values, font-weight, line-height, letter-spacing
- **Spacing**: margin, padding, gap values. Normalize to rem where possible.
- **Border radius**: all values
- **Shadows**: box-shadow declarations
- **Breakpoints**: values from `@media` queries (if present in raw HTML/CSS)

Additionally, parse the `rawHtml` for:

- All `<link rel="stylesheet">` hrefs — fetch each via `firecrawl_scrape` with `formats: ["rawHtml"]` to get raw CSS text
- Inline `<style>` blocks
- CSS custom properties defined on `:root` or `html`

#### Step 3: Cluster and Deduplicate

- Group equivalent color values across formats (e.g., `#333`, `#333333`, `rgb(51,51,51)` -> single canonical hex)
- Sort colors by hue, then lightness to identify palette ramps
- Sort font sizes numerically to detect the type scale ratio (check against: major second 1.125, minor third 1.2, major third 1.25, perfect fourth 1.333, augmented fourth 1.414, perfect fifth 1.5, golden ratio 1.618)
- Sort spacing values to detect the base unit (4px? 8px?) and scale pattern
- Rank all values by frequency — higher frequency = more likely core token
- Cross-reference with branding output: values that appear in BOTH branding extraction AND raw CSS get a confidence boost

#### Step 4: Map to Token Hierarchy

Following the design-system-patterns schema:

**Primitive tokens** — The raw deduplicated values with systematic names:

- `--color-{hue}-{shade}` (e.g., `--color-blue-500`)
- `--space-{n}` (e.g., `--space-1: 0.25rem`)
- `--font-size-{name}` (e.g., `--font-size-sm`)
- `--radius-{name}`, `--shadow-{name}`, etc.

**Semantic tokens** — CONSERVATIVE inference using DOM-context mapping only:

- `color` property on `body` or `html` -> `--text-primary`
- `background-color` on `body` or `html` -> `--surface-default`
- `color` on `a` elements -> `--interactive-primary`
- `background-color` on `button` or submit-like elements -> `--interactive-primary-bg`
- `border-color` on most common bordered elements -> `--border-default`
- **Everything else stays as a primitive.** Do NOT infer semantic meaning from frequency alone. Under-inference is strictly better than wrong inference.

**Component tokens** — Only generate if a clear, repeated component pattern emerges across multiple pages (e.g., identical button styles across 3+ pages = `--button-bg`, `--button-text`, `--button-radius`). Do not force component tokens from insufficient data.

#### Step 5: Generate Output Artifacts

**Phase 1 outputs:**

1. `design-system.css` — CSS custom properties, all token layers, light mode only
2. `design-system.json` — Style Dictionary / Tokens Studio compatible JSON

**Always generate:**

- `design-system-report.md` — Human-readable report containing:
  - Color palette grouped by hue ramp with hex values
  - Type scale with computed sizes and detected ratio
  - Spacing scale with detected base unit
  - List of values that couldn't be classified
  - Pages crawled and sample size notes
  - Any extraction failures or fallback paths taken

- `design-system-preview.html` — A self-contained HTML file that renders the extracted design system as a living visual reference, including:
  - Color palette swatches grouped by hue family
  - Typography scale samples (each size rendered in extracted fonts, with both Latin and CJK text if applicable)
  - Spacing scale visualization (graduated bars)
  - Border radius samples
  - Shadow samples (progressive elevation)
  - Gradient samples
  - Component examples (buttons, badges, cards) using extracted tokens
  - The page uses the extracted tokens themselves for styling (eat your own dog food)
  - Light/dark toggle if dark mode tokens were extracted (Phase 2+)
  - Loads extracted Google Fonts via `<link>` for accurate rendering

---

### Phase 2 — CSS-in-JS Fallback + Dark Mode

**Adds:** Firecrawl browser sessions for CSS-in-JS sites. Dark mode extraction.

#### CSS-in-JS Fallback (replaces local Playwright)

**Trigger condition:** If Phase 1 scraping yields fewer than 20 meaningful CSS declarations (a sign of CSS-in-JS, styled-components, Emotion, CSS Modules, or heavy runtime injection).

Use Firecrawl's browser tools instead of local Playwright:

1. `firecrawl_browser_create` — Start a cloud browser session
2. `firecrawl_browser_execute` with JavaScript to:
   - Navigate to each target URL
   - Run `getComputedStyle()` against a sampling of DOM elements: `html`, `body`, `h1`-`h6`, `p`, `a`, `button`, `input`, `select`, `textarea`, `nav`, `header`, `footer`, `main`, `section`
   - Extract all CSS custom properties from the document via `getComputedStyle(document.documentElement)`
   - Return structured JSON of all extracted values
3. `firecrawl_browser_delete` — Clean up the session

**Filtering computed styles:** `getComputedStyle` returns ~300+ properties per element, most of which are browser defaults. Filter strategy:

- Only keep properties that differ from a baseline `<div>` with no styles
- Only keep properties in the target categories (color, font, spacing, border, shadow)
- Cross-reference against branding output — values appearing in both sources are signal

#### Dark Mode Extraction

Detect dark mode support by checking for:

- A `.dark` class on `html` or `body` with corresponding style overrides
- `@media (prefers-color-scheme: dark)` blocks
- `data-theme="dark"` or similar attribute-based switching

If found:

- Extract a parallel token set for dark mode
- Output both light and dark tokens in the CSS file with the dark set scoped under the appropriate selector
- In the JSON output, nest dark tokens under a `dark` key

---

### Phase 3 — Framework Detection + Tailwind Theme Extraction

**Adds:** CSS framework detection. Partial Tailwind theme extraction (NOT full config reconstruction).

#### Framework Detection

Detect the CSS framework by analyzing HTML class patterns:

- **Tailwind**: Look for utility class patterns (`bg-`, `text-`, `p-`, `m-`, `flex`, `grid`, `rounded-`, `shadow-`, etc.)
- **CSS custom properties native**: If `:root` has a structured custom property system
- **CSS-in-JS**: Detected via the Phase 2 trigger
- **Vanilla / preprocessor output**: Fall through to general extraction

#### Tailwind Theme Extraction (scoped)

When Tailwind is detected:

- Extract theme color values from utility classes in HTML (e.g., `bg-blue-500` -> blue-500 exists in their palette)
- Extract spacing values from utility classes (e.g., `p-4`, `gap-8`)
- Extract breakpoint values from responsive prefixes (e.g., `md:`, `lg:`)
- Map detected values to a `tailwind.config.ts` `theme.extend` object — NOT a full config reconstruction
- Also output a Tailwind v4 `@theme` CSS block with equivalent values

**What this does NOT do:**

- Reconstruct custom plugins or arbitrary utilities
- Reverse-engineer `@apply` directives
- Detect JIT-mode-only classes
- Produce a "drop-in replacement" config — it produces a starting point

Respect the `--framework-hint` argument when auto-detection is ambiguous.

---

### Phase 4 — Polish: Preview, Confidence Scoring, Multi-Format Output

**Adds:** Visual preview, confidence scoring, screenshot-based visual inference, full output format matrix.

#### Confidence Scoring

Every semantic token gets a confidence score (0-100) based on:

- How many pages the pattern was consistent across (cross-page consistency)
- How unambiguous the DOM context was (element specificity)
- Whether branding extraction corroborates the inference
- Tokens with confidence < 70: add `/* LOW CONFIDENCE (score: N): review manually */`
- Tokens with confidence >= 70: add `/* confidence: N */`

#### Visual Inference from Screenshots

From each screenshot (already captured in Phase 1), identify:

1. The 5 most dominant colors by area coverage
2. Whether there are distinct header/body/caption font sizes visible
3. Whether the layout uses a visible grid rhythm or consistent spacing pattern
4. Whether dark sections or alternate color schemes appear

These are categorical observations only (e.g., "primary CTA appears blue" NOT "#3b82f6"). Visual inference supplements parsed data; it does not replace it. Tag every visually inferred value with `source: visual-inference`.

#### Preview HTML

`design-system-preview.html` — A self-contained HTML file that renders the extracted design system:

- Color palette swatches (light and dark if applicable)
- Typography scale samples
- Spacing scale visualization
- Border radius and shadow samples
- Uses the extracted tokens for its own styling (eat your own dog food)
- Light/dark toggle if dark mode tokens were extracted

#### Full Output Format Matrix

Support an `--output-format` flag with these options (default: `all`):

| Flag value | Output files                                                              |
| ---------- | ------------------------------------------------------------------------- |
| `css`      | `design-system.css` — CSS custom properties, all 3 layers, light + dark   |
| `tailwind` | `tailwind.config.ts` + `app.css` with `@theme` block (Tailwind v4 format) |
| `json`     | `design-system.json` — Style Dictionary / Tokens Studio compatible JSON   |
| `all`      | All of the above                                                          |

---

## Skill SKILL.md Requirements

The skill's SKILL.md must:

- Trigger on: "extract design system", "reverse engineer design system", "get design tokens from", "crawl site for styles", "what design system does X use", "extract tokens from website"
- Accept arguments:
  - Target URL (required)
  - `--framework-hint` (optional: `tailwind`, `vanilla`, `css-modules`, `css-in-js`)
  - `--output-format` (optional: `css`, `tailwind`, `json`, `all` — default `all`)
  - `--max-pages` (optional: integer, default 5)
  - `--phase` (optional: `1`, `2`, `3`, `4` — controls how deep the extraction goes, default highest implemented phase)
- Document the Firecrawl MCP dependency and how to verify it's configured
- Document the browser session fallback and when it activates
- List all output files and what each contains

## What This Skill Is NOT

- NOT a website cloner or screenshot-only tool
- NOT a Figma integration — it works from live sites, not design files
- NOT a component library generator — it extracts the _design language_, not component implementations
- NOT a linter or validator — it extracts what IS, not what SHOULD BE
- NOT a frequency-based guesser — semantic inference uses DOM-context mapping

## Error Handling

- The skill must work incrementally — if Step 1 succeeds but Step 3 fails, output whatever was successfully extracted with a clear note about where it stopped and why
- If Firecrawl MCP is not configured or the API key is missing, detect this and output clear setup instructions rather than failing silently
- If Firecrawl credits run out mid-crawl, save whatever has been collected so far and note the incomplete crawl in the report
- If browser session creation fails, skip the CSS-in-JS fallback and note this limitation in the report
- If a page times out or returns an error, skip it and continue with remaining pages

## Dependencies

| Dependency                     | Required by | Purpose                                                      |
| ------------------------------ | ----------- | ------------------------------------------------------------ |
| Firecrawl MCP                  | All phases  | Scraping, branding extraction, screenshots, browser sessions |
| `design-system-patterns` skill | Phase 1+    | Output schema reference (3-layer token hierarchy)            |

No local Python packages required for Phase 1. Firecrawl handles scraping, screenshots, and browser automation in the cloud.

## Execution Order

1. Read the `design-system-patterns` skill (for output schema) and `skill-creator` skill (for skill structure conventions)
2. **MCP Tool Discovery Gate**: Inspect the Firecrawl MCP server. Print every tool name and full parameter schema. Save to `firecrawl-mcp-tools.json`. Do NOT proceed until this file exists.
3. Design the file structure and SKILL.md
4. Implement Phase 1 (Steps 1-5)
5. Test against TWO sites:
   - `https://ui.shadcn.com` — CSS custom properties, light/dark mode, component library docs. Validates token hierarchy extraction.
   - `https://tailwindcss.com` — Tailwind-based, utility classes, known design system. Validates branding extraction pipeline.
6. Review outputs against the design-system-patterns token hierarchy — verify conformance
7. Fix gaps, then present the Phase 1 skill for review
8. Iterate to Phase 2+ based on user feedback

## Test Strategy

**Phase 1 validation (2 sites):**

- `ui.shadcn.com` — Should produce CSS custom properties, structured color palette, type scale
- `tailwindcss.com` — Should produce branding data, color ramps, spacing scale

**Phase 2 validation (add 1 site):**

- `vercel.com` — CSS-in-JS site. Validates browser session fallback. If extraction quality is poor, document what failed and why rather than hiding it.

**Phase 3 validation:**

- Re-run `tailwindcss.com` with framework detection enabled. Should produce partial `tailwind.config.ts`.

**For each test, verify:**

- [ ] Output files exist and are valid (CSS parses, JSON parses)
- [ ] Color palette has at least 5 distinct hues
- [ ] Type scale detected with ratio identification
- [ ] Spacing scale detected with base unit
- [ ] Semantic tokens are conservative (no over-inference)
- [ ] Report documents any failures or limitations
