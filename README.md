# Design System Extractor

A Claude Code skill that extracts structured design tokens from any live website using the Firecrawl MCP server.

## What it does

Given a URL, the skill:

1. Crawls the site using Firecrawl's `branding` format + raw HTML scraping
2. Extracts colors, typography, spacing, shadows, border radius, breakpoints
3. Clusters and deduplicates values, detects type scale ratios and spacing base units
4. Maps everything to a 3-layer token hierarchy (primitive → semantic → component)
5. Outputs four files:
   - `design-system.css` — CSS custom properties in 3 layers
   - `design-system.json` — Style Dictionary / Tokens Studio compatible JSON
   - `design-system-report.md` — Human-readable analysis with palette tables and scale detection
   - `design-system-preview.html` — Visual reference page styled with the extracted tokens

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- [Firecrawl MCP](https://firecrawl.dev) configured with API key

## Installation

The skill is symlinked to `~/.claude/skills/design-system-extractor/`. If the symlink is missing:

```bash
ln -sf /path/to/this/repo/skill ~/.claude/skills/design-system-extractor
```

## Usage

In Claude Code, say:

- "Extract the design system from https://example.com"
- "What fonts and colors does example.com use?"
- "Get design tokens from https://example.com"
- "Crawl example.com for its design system"

## Development

```bash
pnpm install        # Install dev dependencies
pnpm format         # Format with dprint
pnpm format:check   # Check formatting
```
