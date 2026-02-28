---
description: Generate a progressive depth exploration of a module — scroll to zoom from overview to implementation detail
---
Load the visual-explainer skill, then generate a progressive depth exploration as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template at `./templates/scroll-showcase.html`, the scroll animation patterns at `./references/scroll-animations.md`, CSS patterns at `./references/css-patterns.md`, and libraries at `./references/libraries.md` before generating. Use a technical blueprint or data-dense aesthetic — vary fonts and palette from previous diagrams.

**Input parsing** — determine the module and title:
- `$1`: Path to the module or directory to explore (e.g., `src/auth/`, `lib/parser.ts`, `packages/core/`)
- `$2` (optional): Display title (e.g., `"Authentication Module"`, `"Query Parser"`). If not provided, derive from the module path.
- If no arguments, ask the user what module to explore.

**Data gathering phase** — run these to understand the module at multiple depths:

1. **Surface scan.** List all files in the module. Read README or top-level comments. Identify the module's public API surface (exports, entry points). Determine the module's single-sentence purpose.

2. **Depth 0 — Bird's eye.** What does this module do? One paragraph. What are its inputs and outputs? What does the rest of the system depend on it for? Draw the module as a single box with labeled connections to the outside world.

3. **Depth 1 — Components.** What are the major internal components (files, classes, subsystems)? For each: name, one-sentence purpose, and which other components it talks to. Read each file's exports and top-level structure.

4. **Depth 2 — Internals.** For each major component, what are the key functions, types, and data structures? Read implementation details. Identify algorithms, state machines, caching strategies, or other implementation patterns. Note key invariants and contracts.

5. **Depth 3 — Implementation detail.** Critical code paths with line-level analysis. Edge cases, error handling, performance considerations. Configuration options and their effects. Known limitations or tech debt.

6. **Cross-cutting concerns.** Error handling strategy, logging, testing coverage, configuration, dependencies on external systems.

**Verification checkpoint** — before generating HTML, produce a structured outline:
- Every module, file, function, and type name you will reference with file:line
- Every relationship and dependency you will show
- Every behavior description verified against the actual code

**Page structure** — progressive depth using GSAP pin + scrub:

1. **Hero section** — module name as a large SplitText heading. One-sentence purpose. KPI cards: file count, export count, line count, test coverage if available. Above the fold.

2. **Depth 0 — Overview (pinned section).** The module as a single box in a Mermaid diagram showing external connections. As the user scrolls, the box visually "opens" — the single node expands (via scrub) to reveal the Depth 1 component view. Labels gain detail. New external connections draw in. The transition should feel like zooming into a map.

3. **Depth 1 — Components (pinned section).** Internal components shown as connected cards. As the user scrolls further, each component card expands to reveal its internal structure (Depth 2). Use `scrub` to control the expansion animation — scroll forward to zoom in, scroll backward to collapse back to components.

4. **Depth 2 — Internals.** Key functions and types for each component. Code snippets with annotations. Data flow diagrams showing how information moves through the internals. Cards with function signatures, parameter descriptions, return types.

5. **Depth 3 — Implementation detail.** Critical code paths with line-by-line annotation. Pinned code panels with scrolling callouts. Edge cases and error handling highlighted in amber cards. Performance notes in blue cards.

6. **Cross-cutting concerns.** Error strategy, test coverage, configuration surface. Compact card layout.

7. **Summary table.** All public exports in a data table with type, purpose, and which depth level covers them. Quick reference for navigating back.

**GSAP patterns to use:**
- Depth transitions: `ScrollTrigger` with `pin` and `scrub` to animate diagram transformations
- Node expansion: GSAP `to()` with scrub controlling scale, opacity, and position of child nodes
- `DrawSVGPlugin` for connection lines drawing in at each depth level
- `SplitText` on depth-level headings
- `ScrollTrigger.batch('.gs-reveal')` for non-pinned content cards
- Lenis smooth scroll with `data-lenis-prevent` on `.mermaid-wrap` and code blocks
- Scroll progress bar
- `prefers-reduced-motion` fallback: show all depth levels expanded, skip zoom transitions

Include responsive section navigation. Write to `./.agent/diagrams/` and open in browser.

Ultrathink.

$@
