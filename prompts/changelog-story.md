---
description: Generate a scrollytelling project timeline — scroll-driven changelog with milestone cards and contributor activity
---
Load the visual-explainer skill, then generate a scrollytelling project timeline as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template at `./templates/scroll-showcase.html`, the scroll animation patterns at `./references/scroll-animations.md`, CSS patterns at `./references/css-patterns.md`, and libraries at `./references/libraries.md` before generating. Use an editorial or paper/ink aesthetic — vary fonts and palette from previous diagrams.

**Time window** — determine the recency window from `$1`:
- Shorthand like `2w`, `30d`, `3m`, `6m`, `1y`: parse to git's `--since` format
- If `$1` doesn't match a time pattern, treat it as free-form context and use the default window
- No argument: default to `3m` (3 months)

**Data gathering phase** — run these first to understand the project arc:

1. **Commit history.** `git log --oneline --since=<window>` for full list. `git log --stat --since=<window>` for file-level scope. Count total commits.

2. **Contributor activity.** `git shortlog -sn --since=<window>` for per-author commit counts. `git log --format="%an|%ad" --date=short --since=<window>` for contributor-by-date heatmap data.

3. **Phase detection.** Analyze commit messages and timing to identify natural phases: feature sprints, bug-fix periods, refactoring phases, documentation pushes, release prep. Group commits into phases with descriptive names.

4. **Milestones.** Identify significant commits: initial commits, version bumps, large refactors, new feature introductions, breaking changes. Extract from commit messages, tags (`git tag --sort=-creatordate`), and CHANGELOG.md entries.

5. **File hotspots.** `git log --name-only --since=<window>` to identify most-changed files. Group by directory/module to show where effort concentrated.

6. **Project identity.** Read `README.md`, `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` for name and version context.

**Verification checkpoint** — before generating HTML, produce a structured fact sheet:
- Ordered timeline of phases with date ranges and commit counts
- Every milestone with commit hash, date, and description
- Contributor activity counts verified against git output
- File hotspot rankings with change frequencies

**Page structure** — scrollytelling timeline using GSAP scroll animations:

1. **Hero section** — project name and time window as a large SplitText heading (e.g., "3 Months of visual-explainer"). KPI cards: total commits, contributors, files changed, lines added/removed. Above the fold, CSS animation.

2. **Central timeline** — an SVG vertical line that draws downward as the user scrolls (DrawSVG + scrub). Milestone markers appear as circles on the line. The line represents the full time window from earliest to most recent.

3. **Phase sections** — each phase is a group of milestone cards that batch-animate in via `ScrollTrigger.batch()`. Phase dividers pin briefly with a SplitText character animation on the phase name. For each phase:
   - Phase name and date range
   - 2-3 sentence narrative of what happened and why
   - Milestone cards with commit hash, date, description, and files touched
   - Cards alternate left/right of the timeline for visual variety

4. **"Current date" header** — a sticky element at the top that updates via scrub to show the current position in the timeline as the user scrolls.

5. **Contributor heatmap** — a grid visualization where cells fill per time period as the user scrolls past them. Rows are contributors, columns are weeks/days. Color intensity maps to commit frequency.

6. **File activity map** — treemap or grouped bar visualization showing which modules received the most attention. Animate bars growing on scroll entry.

7. **Arc summary** — a closing section that captures the narrative arc: where the project was at the start of the window, the momentum shifts and pivots, and where it stands now. Forward-looking inference based on recent trajectory.

**GSAP patterns to use:**
- Central SVG timeline: `DrawSVGPlugin` with `scrub: true` tied to overall scroll position
- Milestone cards: `ScrollTrigger.batch()` with staggered entrance
- Phase dividers: `SplitText` character animation triggered on scroll
- Sticky date header: `ScrollTrigger` with `onUpdate` callback to update text
- Heatmap cells: `ScrollTrigger.batch()` with fill animation
- Lenis smooth scroll, scroll progress bar
- `prefers-reduced-motion` fallback: show all content, skip animations, keep timeline visible

Include responsive section navigation. Write to `./.agent/diagrams/` and open in browser.

Ultrathink.

$@
