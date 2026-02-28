---
description: Generate a step-by-step animated migration walkthrough with before/after code transforms
---
Load the visual-explainer skill, then generate an animated migration walkthrough as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template at `./templates/scroll-showcase.html`, the scroll animation patterns at `./references/scroll-animations.md`, CSS patterns at `./references/css-patterns.md`, and libraries at `./references/libraries.md` before generating. Use a diff-inspired aesthetic with red/green before/after language — vary fonts and palette from previous diagrams.

**Input parsing** — determine the migration and scope:
- `$1`: Migration description (e.g., `"Express 4 to Express 5"`, `"React class components to hooks"`, `"CommonJS to ESM"`)
- `--scope <path>`: Optional directory scope to focus the migration analysis. If not provided, analyze the full project.
- If no arguments, ask the user what migration to document.

**Data gathering phase** — run these to understand the migration:

1. **Current state.** Scan the codebase (or `--scope` directory) for patterns matching the "before" side of the migration. Identify files, APIs, patterns, and configurations that need to change. Count occurrences of each pattern.

2. **Migration steps.** Break the migration into ordered steps. For each step:
   - What changes (API, import, configuration, pattern)
   - Before code (actual code from the project if possible, representative example if not)
   - After code (the migrated version)
   - Files affected (list specific files from the project)
   - Dependencies: does this step require a previous step to be completed?
   - Risk level: what could break?

3. **Dependency changes.** Check `package.json` / `Cargo.toml` / `pyproject.toml` for version changes needed. Identify new dependencies to add, old ones to remove, and version bumps.

4. **Configuration changes.** Check for config files that need updating: tsconfig, eslint, bundler config, CI/CD, environment variables.

5. **Breaking changes.** Identify API surface changes that could affect downstream consumers or tests. Read test files to understand what test changes are needed.

6. **Order and batching.** Determine the safest order to apply steps. Group steps that can be done in parallel vs. those that must be sequential. Identify a "point of no return" if applicable.

**Verification checkpoint** — before generating HTML:
- Every code example verified against the actual codebase
- Every file path confirmed to exist
- Step ordering validated (no circular dependencies)
- Risk assessments grounded in actual code patterns found

**Page structure** — step-by-step walkthrough using GSAP pin + scrub:

1. **Hero section** — migration title as a large SplitText heading (e.g., "Express 4 → Express 5"). KPI cards: steps count, files affected, estimated changes. Brief description of why this migration matters. Above the fold.

2. **Overview** — all steps in a connected pipeline visualization. SVG path connects steps as circles on a horizontal or vertical line. Steps color-coded by risk (green/amber/red). DrawSVG animates the path on scroll.

3. **Step-by-step walkthrough** — each step is a pinned section:
   - **Step header** pins at top with step number, title, and risk badge
   - **Before panel** shows current code — lines that will be removed highlighted in red
   - **Transformation animation** (scrub-driven): removed lines slide out with red fade, added lines slide in with green fade, moved lines physically travel to their new position
   - **After panel** shows migrated code — new lines highlighted in green
   - **Explanation cards** scroll in beside the code: what changed and why, gotchas, rollback instructions
   - Max pin duration: 2.5× viewport height per step

4. **Persistent progress bar** — tracks step position across all pinned sections. Shows which step the user is currently viewing out of the total. SVG path from the overview connects steps as the user transitions between them.

5. **Dependency changes** — a table showing packages to add/remove/update with version numbers. Scroll-revealed.

6. **Configuration changes** — before/after panels for each config file that needs updating. Use the pinned comparison pattern for complex configs.

7. **Test updates** — what tests need to change and how. Before/after test code.

8. **Rollback plan** — how to undo the migration if something goes wrong. Collapsed by default (`<details>`).

9. **Checklist** — a summary checklist of all steps with status indicators. The user's reference for tracking progress through the actual migration.

**GSAP patterns to use:**
- Step sections: `ScrollTrigger.create({ pin: true, scrub: true })` for each step
- Code line transforms: GSAP `to()` with scrub controlling translateX, opacity, and background-color
- Step connector: `DrawSVGPlugin` with scrub on the overview SVG path
- Progress bar: custom `ScrollTrigger` tracking position across all pinned sections
- `SplitText` on step headings
- `ScrollTrigger.batch('.gs-reveal')` for non-pinned content
- Lenis smooth scroll with `data-lenis-prevent` on code blocks
- `prefers-reduced-motion` fallback: show all before/after panels side by side, skip line animation, keep step structure

Include responsive section navigation. Write to `./.agent/diagrams/` and open in browser.

Ultrathink.

$@
