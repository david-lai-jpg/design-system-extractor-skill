# Design System Extractor

## Project Structure

- `skill/SKILL.md` — The Claude Code skill definition
- `skill/references/` — Reference docs for the skill
- `skill-creation-prompt.md` — The original spec/architecture doc
- `test-output/` — Test extraction results (gitignored)

## Development

- Formatter: dprint (markdown, json)
- Pre-commit: husky + lint-staged
- Package manager: pnpm

## Key Architecture Decisions

- Firecrawl `branding` format is the primary extraction mechanism (~40% of work)
- Raw CSS parsing supplements branding data for exhaustive token coverage
- No local Playwright — Firecrawl browser sessions handle CSS-in-JS fallback
- Under-inference strictly preferred over wrong inference for semantic tokens
- Preview HTML eats its own dog food — uses extracted tokens for styling
