---
description: Generate a scroll-driven code path walkthrough — pin each system layer as a request travels through it
---
Load the visual-explainer skill, then generate a scroll-driven code path walkthrough as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template at `./templates/scroll-showcase.html`, the scroll animation patterns at `./references/scroll-animations.md`, CSS patterns at `./references/css-patterns.md`, and libraries at `./references/libraries.md` before generating. Use a technical blueprint or IDE-inspired aesthetic — vary fonts and palette from previous diagrams.

**Input parsing** — determine the request path and entry point from arguments:
- `$1`: The request path or flow to trace (e.g., `"POST /api/orders"`, `"user login flow"`, `"WebSocket connection lifecycle"`)
- `--entry <file>`: Optional entry point file. If not provided, infer from the request path by searching for route definitions, handlers, or controllers.
- If no arguments, ask the user what flow to trace.

**Data gathering phase** — run these first to map the complete flow:

1. **Entry point.** Find the route handler or entry function for the specified path. Read the file. Identify the function signature, middleware chain, and first operation.

2. **Trace the call chain.** Follow function calls from the entry point through each layer: routing → middleware → validation → service logic → data access → external calls → response. Read each file involved. For each layer, capture:
   - File path and function name
   - What this layer does (one sentence)
   - Data shape at entry and exit (what the payload looks like at this stage)
   - Key transformations or side effects
   - Error handling behavior

3. **Connection points.** For each transition between layers, identify: how data passes (function args, context object, event, queue message), what validation occurs at the boundary, and what could go wrong.

4. **Scope boundaries.** Identify where the flow crosses major boundaries: process boundary, network call, async/await transition, transaction scope, auth context change.

**Verification checkpoint** — before generating HTML, produce a structured trace:
- Ordered list of layers with file:line references
- Data shape at each transition point
- Every function name and module you will reference
- Verify each claim against the actual code

**Page structure** — scroll-driven walkthrough using GSAP pin + scrub:

1. **Hero section** — the request path as a large SplitText heading (e.g., `POST /api/orders`). Brief description of what this flow does end-to-end. Above the fold, CSS animation.

2. **Overview diagram** — Mermaid flowchart showing all layers at a glance. Wrap in `.mermaid-wrap` with zoom controls and `data-lenis-prevent`. Use DrawSVG to progressively reveal edges on scroll.

3. **Layer-by-layer walkthrough** — each layer is a pinned section. As the user scrolls:
   - **Left panel (pinned):** Code snippet from this layer with relevant lines highlighted. Use a styled code block with line numbers and filename label.
   - **Right panel (scrolls in):** Data payload shape at this stage (as a styled JSON/type annotation). Callout cards explaining what happens. Connection to next layer drawn as an SVG line.
   - Each layer's pinned section should be at most 2× viewport height.
   - Visible scroll indicator showing progress through the current layer.

4. **Error paths** — a section showing what happens when things go wrong at each layer. Color-coded red cards for error cases, with the normal flow shown in green for contrast.

5. **Summary** — condensed view: all layers in a compact table or pipeline visualization. Total request lifecycle timing if inferable.

**GSAP patterns to use:**
- `ScrollTrigger.create({ pin: true, scrub: true })` for each layer section
- `SplitText` on the hero heading
- `DrawSVGPlugin` on the overview diagram edges
- `ScrollTrigger.batch('.gs-reveal')` for non-pinned content
- Lenis smooth scroll with `data-lenis-prevent` on `.mermaid-wrap` and code blocks with overflow
- Scroll progress bar at viewport top
- `prefers-reduced-motion` fallback: skip Lenis, skip pin animations, show all content immediately

Include responsive section navigation. Overflow prevention on all grid/flex children. Write to `./.agent/diagrams/` and open in browser.

Ultrathink.

$@
