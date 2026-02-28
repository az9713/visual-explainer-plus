---
description: Generate a scroll-driven dependency graph exploration — progressive depth reveal of transitive chains
---
Load the visual-explainer skill, then generate a scroll-driven dependency graph exploration as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template at `./templates/scroll-showcase.html`, the scroll animation patterns at `./references/scroll-animations.md`, CSS patterns at `./references/css-patterns.md`, and libraries at `./references/libraries.md` before generating. Use a data-dense or blueprint aesthetic — vary fonts and palette from previous diagrams.

**Input parsing** — determine the focus and scope:
- `--focus <package>`: Optional package name to highlight in the tree (e.g., `lodash`, `express`, `react`)
- `--depth <n>`: Optional max depth to explore (default: 3)
- `--dev`: Include devDependencies (excluded by default)
- If no arguments, analyze all production dependencies.

**Data gathering phase** — run these to map the dependency tree:

1. **Direct dependencies.** Read `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod`. List production and dev dependencies with version constraints.

2. **Dependency tree.** Use the package manager's tree command to get the full transitive tree:
   - npm: `npm ls --all --json` (or `npm ls --depth=<n>`)
   - yarn: `yarn info --dependents`
   - pnpm: `pnpm ls --depth=<n>`
   - cargo: `cargo tree`
   - pip: `pip show <pkg>` or `pipdeptree`
   - go: `go mod graph`
   Parse the output into a structured tree with depth levels.

3. **Size and impact.** For JS projects: check `node_modules` sizes if available, or note package sizes from registry metadata. Identify the heaviest transitive chains. For Rust/Go: note compile-time impact of large dependency trees.

4. **Duplicate detection.** Identify packages that appear at multiple versions in the tree (version conflicts). Flag these as potential optimization targets.

5. **Security.** Run `npm audit` / `cargo audit` / `pip audit` if available. Note any known vulnerabilities with severity levels.

6. **Focus analysis.** If `--focus` is specified: trace all paths from the root to the focused package. Identify which direct dependencies pull it in. Check if it could be replaced or removed.

**Verification checkpoint** — before generating HTML:
- Dependency tree structure verified against package manager output
- Every package name and version confirmed
- Depth levels accurate
- Duplicate and vulnerability data confirmed

**Page structure** — progressive depth exploration using GSAP scroll animations:

1. **Hero section** — project name and dependency count as a SplitText heading. KPI cards: direct deps, total transitive deps, max depth, duplicates found, vulnerabilities. Above the fold.

2. **Direct dependencies (Depth 0)** — the starting view. Direct dependencies shown as a ring or row of cards around the project root. Each card shows: package name, version, purpose (one line), and count of transitive deps it brings.

3. **Progressive depth reveal** — as the user scrolls:
   - **Depth 1:** First-level transitive dependencies fly outward from their parent nodes. Connection lines draw in via DrawSVG. `ScrollTrigger.batch()` staggers nodes per depth level.
   - **Depth 2:** Second-level transitives appear, expanding the graph further. Pinning keeps the full graph centered while new layers appear within it.
   - **Depth 3+:** Deeper levels continue to expand. At each level, the graph recenters and rescales to fit.
   - Scrolling backward collapses deeper levels — nodes fade out, connections retract.
   - Use `scrub` for continuous scroll-position-driven expansion, not binary show/hide.

4. **Focus spotlight** (if `--focus` specified) — a pinned section that highlights all paths to the focused package. Dim all unrelated nodes. Trace lines glow to show the dependency chains. Show which direct dependencies are responsible for pulling in the focused package.

5. **Duplicates and conflicts** — cards highlighting packages with multiple versions. Before/after showing what deduplication could look like. Color-coded by severity (red for major version conflicts, amber for minor).

6. **Security findings** — vulnerability cards with severity badges (critical/high/medium/low). Each card shows: package, version, vulnerability ID, and which dependency chain introduces it.

7. **Heaviest chains** — a ranked list or bar chart of dependency chains by total transitive count or size. Identifies optimization opportunities.

8. **Summary table** — all direct dependencies in a data table with columns: name, version, direct transitive count, total transitive count, has duplicates, has vulnerabilities.

**GSAP patterns to use:**
- Depth level reveals: `ScrollTrigger` with `scrub` controlling node opacity, scale, and position
- Connection lines: `DrawSVGPlugin` with staggered draw-in per depth level
- Graph recentering: GSAP `to()` on the graph container's transform
- `ScrollTrigger.batch()` for nodes at each depth level
- `pin` on the graph section while new depth levels appear within it
- `SplitText` on the hero heading
- Lenis smooth scroll, scroll progress bar
- `prefers-reduced-motion` fallback: show all depth levels at once, skip fly-out animations, keep graph static

Include responsive section navigation. Write to `./.agent/diagrams/` and open in browser.

Ultrathink.

$@
