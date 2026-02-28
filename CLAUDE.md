# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**visual-explainer** is an agent skill (not a traditional app/library) that turns complex terminal output into styled, self-contained HTML pages. It has no build step, no runtime, and no tests — it's a set of design reference documents, HTML templates, and prompt templates that guide an AI agent to generate beautiful diagrams, diff reviews, slide decks, and data tables.

## Repository Structure

```
SKILL.md              ← Core skill definition: workflow, diagram types, aesthetics, anti-patterns
references/
  css-patterns.md     ← Reusable CSS: themes, depth tiers, cards, grids, animations, overflow protection
  libraries.md        ← CDN libraries: Mermaid theming, Chart.js, anime.js, font pairings
  responsive-nav.md   ← Sticky sidebar TOC for multi-section pages
  slide-patterns.md   ← Slide engine CSS, 10 slide types, transitions, presets
templates/
  architecture.html   ← CSS Grid cards reference (terracotta/sage palette)
  mermaid-flowchart.html ← Mermaid + ELK reference (teal/cyan palette)
  data-table.html     ← HTML tables with KPIs reference (rose/cranberry palette)
  slide-deck.html     ← Viewport-fit slides reference (midnight editorial palette)
prompts/
  *.md                ← Slash command templates (diff-review, plan-review, generate-slides, etc.)
package.json          ← pi-package metadata for `pi install`
```

## How This Skill Works

The agent follows a 4-step workflow defined in `SKILL.md`:
1. **Think** — Pick aesthetic direction, audience, diagram type
2. **Structure** — Read the matching reference template + `references/` docs before generating
3. **Style** — Apply typography, color, animation, and depth rules
4. **Deliver** — Write to `~/.agent/diagrams/` and open in browser

Templates use deliberately different palettes so the agent learns variety. The skill routes to the right rendering approach per diagram type (Mermaid for connections, CSS Grid for text-heavy cards, HTML tables for data, Chart.js for dashboards).

## Key Conventions

- **Output is always a single self-contained `.html` file** — no external assets except CDN links (fonts, Mermaid, Chart.js, anime.js)
- **Both light and dark themes required** via CSS custom properties + `prefers-color-scheme`
- **Card class is `.ve-card`**, never `.node` (Mermaid uses `.node` internally and it causes CSS collisions)
- **Mermaid must use `theme: 'base'`** with custom `themeVariables` — built-in themes ignore variable overrides
- **Mermaid zoom controls** (+/−/reset, Ctrl+scroll, drag-to-pan) are required on every `.mermaid-wrap`
- **Never set `color:` in Mermaid `classDef`** — it hardcodes a value that breaks in the opposite color scheme; use CSS overrides with `var(--text)` instead
- **Use semi-transparent fills (8-digit hex)** in Mermaid `classDef` for backgrounds that work in both themes

## Anti-Slop Rules (Strictly Enforced)

Forbidden patterns that signal "AI-generated template":
- **Fonts**: Inter, Roboto, Arial, Helvetica, or bare system-ui as primary body font
- **Colors**: Indigo/violet (`#8b5cf6`, `#7c3aed`, `#a78bfa`), fuchsia (`#d946ef`), cyan-magenta-pink neon combo
- **Effects**: Gradient text on headings (`background-clip: text`), animated glowing box-shadows, emoji section headers
- **Aesthetics**: "Neon dashboard" and "Gradient mesh" are explicitly banned
- **Layout**: Three-dot window chrome on code blocks, perfectly uniform card grids with no hierarchy

Preferred palettes: terracotta/sage, teal/slate, rose/cranberry, amber/emerald, deep blue/gold. Preferred font pairings are listed in `references/libraries.md` (top 5: DM Sans+Fira Code, Instrument Serif+JetBrains Mono, IBM Plex Sans+IBM Plex Mono, Bricolage Grotesque+Fragment Mono, Plus Jakarta Sans+Azeret Mono).

## Editing Guidelines

- When modifying templates, maintain the self-contained HTML pattern (all CSS inline in `<style>`, no external stylesheets)
- When adding diagram types, update the routing table in `SKILL.md` and add a reference template if needed
- When adding prompt templates, place them in `prompts/` as markdown with YAML frontmatter
- Reference docs in `references/` are read by the agent before each generation — keep them as concrete patterns with code examples, not abstract guidelines
- The agent re-reads reference files fresh each time, so changes take effect immediately
