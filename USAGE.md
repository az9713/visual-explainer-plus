# Visual Explainer — Command Usage Guide

A complete guide to using the 12 slash commands, scroll-driven animation features, and 1 automatic behavior effectively. Covers when to use each, how to invoke them, why they exist, and recommended workflows.

---

## The Decision Framework

Ask yourself one question: **What am I trying to understand?**

| I need to understand... | Use this | Why |
|---|---|---|
| A concept, system, or technical topic | `/generate-web-diagram` | General-purpose — turns any topic into a visual page |
| What changed in my code | `/diff-review` | Purpose-built for code diffs with before/after analysis |
| Whether a plan will actually work | `/plan-review` | Cross-references a plan against real code |
| A project I haven't touched in days | `/project-recap` | Rebuilds your mental model from git history |
| Whether a generated document is accurate | `/fact-check` | Verifies every factual claim against actual code |
| Presenting any of the above to others | `/generate-slides` or `--slides` | Converts content into a slide deck format |
| How a request flows through our code | `/trace-flow` | Scroll-driven walkthrough pinning each layer |
| The arc of a project over weeks/months | `/changelog-story` | Scrollytelling timeline with phases and milestones |
| A module at multiple zoom levels | `/deep-dive` | Progressive depth — scroll to zoom overview → detail |
| How to migrate from X to Y | `/migration-guide` | Step-by-step animated before/after code transforms |
| What our dependency tree looks like | `/dependency-explorer` | Scroll-driven graph with progressive depth reveal |
| Onboarding a new team member | `/onboarding-walkthrough` | Guided codebase intro at the reader's own scroll pace |

### Static vs. Scroll-Driven Commands

The 12 commands split into two families:

| Family | Commands | Animation model |
|---|---|---|
| **Static pages** | `/generate-web-diagram`, `/diff-review`, `/plan-review`, `/project-recap`, `/fact-check`, `/generate-slides` | CSS load animations. Everything visible at once. Scroll to read. |
| **Scroll-driven pages** | `/trace-flow`, `/changelog-story`, `/deep-dive`, `/migration-guide`, `/dependency-explorer`, `/onboarding-walkthrough` | GSAP + Lenis. Content reveals as you scroll. Pinned sections, scrub-driven animation, progressive disclosure. |

Scroll-driven pages use **GSAP ScrollTrigger** for viewport-based reveals, **Lenis** for smooth scroll with momentum, **SplitText** for editorial heading effects, and **DrawSVG** for SVG path animations. The reader controls pacing with their scroll speed. All scroll-driven pages degrade gracefully to static content when `prefers-reduced-motion` is enabled.

---

## Understanding GSAP and Lenis

The scroll-driven commands rely on two libraries — **GSAP** and **Lenis** — that work together to create pages where the reader's scroll wheel controls the animation. This section explains what they are, why they're here, and exactly what role each plays across the 12 commands.

### What is GSAP?

**GSAP** (GreenSock Animation Platform) is a JavaScript animation library used on over 12 million websites. It's the industry standard for high-performance web animation — used by Google, Apple, NASA, and most major creative agencies. Unlike CSS animations (which fire once on page load), GSAP can tie animations to scroll position, split text into individually animatable characters, draw SVG paths progressively, and pin sections in place while content scrolls around them.

**All GSAP plugins are free.** Webflow acquired GreenSock in 2024 and removed all paywalls. SplitText and DrawSVGPlugin (previously $99+/year) are now available on public CDN at no cost.

The visual-explainer skill uses four GSAP components:

| Component | CDN size | What it does | Where it's used |
|---|---|---|---|
| **GSAP Core** | 27 KB | Animation engine — tweens, timelines, easing | Every scroll-driven page |
| **ScrollTrigger** | 14 KB | Triggers animations based on scroll position — viewport entry, scrub (scroll-position-driven progress), and pin (section stays fixed while user scrolls) | Every scroll-driven page + multi-section static pages |
| **SplitText** | 8 KB | Splits headings into individual words/characters so each can be animated independently — creates editorial-quality text reveals | Hero headings on all scroll-driven pages |
| **DrawSVGPlugin** | 4 KB | Animates SVG `stroke-dashoffset` — makes lines and paths appear to "draw" themselves | Mermaid diagram edge reveals, timeline lines, step connectors |

### What is Lenis?

**Lenis** is a smooth scroll library (2 KB) that replaces the browser's native scroll behavior with buttery momentum-based scrolling. When you release the scroll wheel, the page continues gliding and decelerates naturally instead of stopping abruptly. It's the same scroll feel you experience on Apple.com, Linear, and other polished marketing sites.

**Why Lenis and not just native scroll?** Three reasons:

1. **Consistent cross-browser behavior.** Native `scrollIntoView({ behavior: 'smooth' })` uses different easing curves on Chrome, Firefox, and Safari. Lenis makes TOC navigation feel identical everywhere.

2. **GSAP sync.** Lenis feeds its scroll position into GSAP's animation ticker via a standard integration pattern. This means ScrollTrigger animations are perfectly synchronized with the scroll — no jank, no desync between scroll position and animation state.

3. **Momentum.** When the user flicks the scroll wheel, the page has physical weight. It doesn't stop dead. This makes long scroll-driven pages (like `/changelog-story` timelines or `/onboarding-walkthrough` concept sequences) feel responsive rather than mechanical.

**Lenis is disabled** when the user has `prefers-reduced-motion: reduce` enabled in their OS. The page falls back to native browser scroll.

### How They Work Together

The integration pattern is the same in every scroll-driven page:

```javascript
// Lenis handles the scroll physics (momentum, easing)
const lenis = new Lenis({ autoRaf: false, lerp: 0.08 });

// Lenis feeds its scroll position to GSAP's ScrollTrigger
lenis.on('scroll', ScrollTrigger.update);

// GSAP's ticker drives Lenis's animation frame
gsap.ticker.add((time) => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
```

**Lenis** controls *how* the page scrolls (smooth momentum). **GSAP** controls *what happens* as the page scrolls (animations, pins, reveals). Together they create the scroll-driven experience.

### Which Features Each Command Uses

Not every command uses every feature. Here's the exact breakdown:

| Command | ScrollTrigger `batch` | ScrollTrigger `pin` | ScrollTrigger `scrub` | SplitText | DrawSVG | Lenis |
|---|---|---|---|---|---|---|
| `/generate-web-diagram` | On multi-section pages | — | — | — | — | On multi-section pages |
| `/diff-review` | On multi-section pages | Pinned comparison panels | — | — | — | On multi-section pages |
| `/plan-review` | On multi-section pages | Pinned comparison panels | — | — | — | On multi-section pages |
| `/project-recap` | On multi-section pages | — | — | — | — | On multi-section pages |
| `/fact-check` | — | — | — | — | — | — |
| `/generate-slides` | — | — | — | — | — | — |
| `/trace-flow` | Card reveals | Each system layer | Code highlighting | Hero heading | Layer connection lines | Smooth TOC + scroll |
| `/changelog-story` | Milestone card groups | Phase dividers | Timeline draw progress | Hero heading + phase titles | Central timeline line | Smooth TOC + scroll |
| `/deep-dive` | Node reveals per depth | Depth level transitions | Zoom between depths | Hero heading + key terms | Mermaid edge reveals | Smooth TOC + scroll |
| `/migration-guide` | Non-pinned content | Each migration step | Code line transforms | Hero heading + step titles | Step connector SVG path | Smooth TOC + scroll |
| `/dependency-explorer` | Nodes per depth level | Graph during expansion | Depth level reveals | Hero heading | Connection lines between nodes | Smooth TOC + scroll |
| `/onboarding-walkthrough` | File tree + card reveals | Each concept section | Code annotation panels | Hero heading + key terms | Architecture diagram edges | Smooth TOC + scroll |

**Key observations from this table:**

- **`/fact-check` and `/generate-slides`** don't use GSAP or Lenis at all — `/fact-check` edits files in place (no HTML output), and `/generate-slides` uses its own viewport-fit slide engine.
- **The original 6 static commands** use ScrollTrigger `batch` and Lenis only when they produce pages with 4+ sections. Single-section diagrams still use CSS-only load animations.
- **All 6 new scroll-driven commands** use the full GSAP + Lenis stack: batch reveals for cards, pin + scrub for their core interaction pattern, SplitText on headings, DrawSVG on SVG paths, and Lenis for smooth scroll.
- **`scrub`** is the key differentiator between static and scroll-driven pages. Scrub ties animation progress directly to scroll position — scroll 50% through a section and the animation is at 50%. This is what makes code highlighting follow your scroll, depth levels zoom continuously, and timeline paths draw proportionally.

### What Each GSAP Feature Looks Like in Practice

**`ScrollTrigger.batch()` — viewport entry reveals:**
When you scroll a section of cards into view, they don't just appear — they stagger in one after another with a subtle fade-up. The batch groups elements that enter the viewport at the same time and animates them with a configurable stagger delay. This is the `.gs-reveal` class you see in generated HTML.

**`ScrollTrigger.create({ pin: true })` — pinned sections:**
A section stays fixed in the viewport while you continue scrolling. The surrounding content scrolls around it. This is how `/trace-flow` holds a code panel in place while explanation cards scroll in, how `/migration-guide` holds a "before" code panel while the transformation animates, and how `/onboarding-walkthrough` holds each concept until you've scrolled through its explanation.

**`ScrollTrigger.create({ scrub: true })` — scroll-position-driven animation:**
Instead of playing an animation on enter and forgetting about it, scrub ties the animation's progress to your exact scroll position. Scroll to 30% of the section? Animation is at 30%. Scroll backward? Animation reverses. This is what makes the `/deep-dive` zoom transitions feel continuous rather than binary, and what makes `/migration-guide` code transforms respond to scroll direction.

**`SplitText` — editorial text reveals:**
Takes a heading like "Authentication Module" and splits it into individual characters or words. Each character can then be animated independently — creating the staggered entrance effect where letters appear one after another. Used on hero headings across all scroll-driven pages. The effect takes 0.6–1.0 seconds on page load.

**`DrawSVGPlugin` — SVG path drawing:**
Makes SVG paths (lines, curves, connections) appear to draw themselves progressively. Used for Mermaid diagram edges (nodes appear first, then connection lines draw between them), timeline center lines (the `/changelog-story` vertical line draws as you scroll), and step connector paths (the `/migration-guide` pipeline visualization).

### CDN Links (All Free)

Every scroll-driven page loads these from CDN — no install, no build step, no license cost:

```html
<!-- Lenis smooth scroll (2 KB) -->
<script src="https://cdn.jsdelivr.net/npm/lenis@1.3.17/dist/lenis.min.js"></script>

<!-- GSAP core + plugins (53 KB total) -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/ScrollTrigger.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/SplitText.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/DrawSVGPlugin.min.js"></script>
```

Total overhead: ~55 KB (gzipped: ~18 KB). For context, a single hero image is typically 50–200 KB.

### Accessibility: `prefers-reduced-motion`

Every scroll-driven page checks the user's motion preference:

```javascript
const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```

When reduced motion is enabled:
- **Lenis** is not initialized — native browser scroll is used
- **ScrollTrigger batch/pin/scrub** animations are skipped — content is visible immediately with no animation
- **SplitText** effects are skipped — headings render normally
- **DrawSVG** is skipped — SVG paths display fully drawn
- **The page is fully readable** — all content is shown, all sections are navigable, nothing is hidden behind scroll interaction

This is not a degraded experience — it's an alternative presentation that respects the user's preference. The information is identical; only the motion is removed.

---

## Static Page Commands (Original 6)

### 1. `/generate-web-diagram` — The Swiss Army Knife

**What it does:** Takes any topic and produces a styled HTML page with diagrams, tables, or cards.

**When to use it:**
- You want to visualize something that isn't a code diff or plan — an authentication flow, a database schema, a system architecture, a comparison of technologies
- You're explaining a concept to someone (or to yourself)
- You want a shareable artifact instead of a chat explanation

**How:**
```
/generate-web-diagram our authentication flow
/generate-web-diagram WebSocket lifecycle with reconnection logic
/generate-web-diagram comparison of Redis vs Memcached for our session store
```

**Why this and not the others:** This is the only command with no opinion about *what* you're diagramming. The other 11 commands all have a specific data-gathering pipeline (git diffs, plan files, code analysis, etc.). This one is pure "turn my idea into a visual page."

**Rendering approach selection:** The agent automatically picks the rendering approach based on the content:

| Content type | Rendering approach | Example |
|---|---|---|
| Components with connections | Mermaid flowchart/graph | Authentication flow, microservice topology |
| Text-heavy architecture | CSS Grid cards | System overview with descriptions per component |
| Structured data | HTML `<table>` | Feature comparison, API endpoint inventory |
| Metrics and charts | CSS Grid + Chart.js | Performance dashboard, usage statistics |
| Sequential interactions | Mermaid sequence diagram | API request lifecycle, user login flow |
| Entity relationships | Mermaid ER diagram | Database schema, data model |
| State transitions | Mermaid state diagram | Order lifecycle, build pipeline states |
| Hierarchical breakdown | Mermaid mind map | Project structure, technology stack |

**Pro tip:** If you want a specific format, say so: "generate a Mermaid sequence diagram of..." or "generate a data table comparing..." The agent will respect explicit format requests over its automatic routing.

---

### 2. `/diff-review` — Post-Coding Review

**What it does:** Generates a comprehensive visual code review with architecture diagrams, KPI dashboard, Good/Bad/Ugly analysis, decision log, and re-entry context.

**When to use it:**
- You just finished a feature branch and want to review before merging
- You want to understand what someone else changed
- You need a shareable code review artifact for a PR
- You want to capture *why* decisions were made before you forget

**How:**
```
/diff-review                    # your branch vs main (most common)
/diff-review abc123             # a specific commit
/diff-review main..HEAD         # only committed changes, not working tree
/diff-review HEAD               # only uncommitted changes
/diff-review #42                # a pull request (uses gh CLI)
/diff-review feature/auth       # working tree vs a specific branch
```

**Scope detection rules:**

| Argument | What gets diffed |
|---|---|
| *(none)* | Working tree vs `main` (default) |
| Branch name (`main`, `develop`) | Working tree vs that branch |
| Commit hash (`abc123`) | That specific commit's changes (`git show`) |
| `HEAD` | Uncommitted changes only (`git diff` + `git diff --staged`) |
| PR number (`#42`) | Pull request diff via `gh pr diff 42` |
| Range (`abc123..def456`) | Diff between two commits |

**Why this is special:** It doesn't just show *what* changed — it reconstructs *why*. It mines your conversation history, commit messages, and progress docs to build a decision log with confidence indicators. High-confidence rationale (from explicit discussion) gets green cards. Inferred rationale gets blue cards. Unknown rationale gets amber warnings telling you to document before committing.

**The 10 sections it produces:**

1. **Executive summary** — Not a dry before/after. Leads with the *intuition*: why do these changes exist? What problem were they solving, what was the core insight? Then factual scope (X files, Y lines, Z new modules). A reader who only sees this section should understand the essence of the change. This is the visual anchor — hero depth with larger type (20-24px), subtle accent-tinted background, more padding than other sections.

2. **KPI dashboard** — Lines added/removed, files changed, new modules, test counts. Includes a **housekeeping** indicator: whether CHANGELOG.md was updated (green/red badge) and whether docs need changes (green/yellow/red).

3. **Module architecture** — How the file structure changed, with a Mermaid dependency graph of the current state. Includes zoom controls (+/−/reset buttons), Ctrl/Cmd+scroll zoom, and click-and-drag panning.

4. **Major feature comparisons** — Side-by-side before/after panels for each significant area of change (UI, data flow, API surface, config, etc.).

5. **Flow diagrams** — Mermaid flowchart, sequence, or state diagrams for any new lifecycle/pipeline/interaction patterns introduced by the changes.

6. **File map** — Full tree with color-coded new/modified/deleted indicators. Collapsible by default for pages with many sections.

7. **Test coverage** — Before/after test file counts and what's covered.

8. **Code review** — Structured Good/Bad/Ugly analysis:
   - **Good**: Solid choices, improvements, clean patterns worth calling out
   - **Bad**: Concrete issues — bugs, regressions, missing error handling, logic errors
   - **Ugly**: Subtle problems — tech debt introduced, maintainability concerns, things that work now but will bite later
   - **Questions**: Anything unclear or that needs the author's clarification
   - Each item references specific files and line ranges. If nothing to flag in a category, it says "None found" rather than omitting the section.

9. **Decision log** — For each significant design choice, a styled card with:
   - **Decision**: one-line summary
   - **Rationale**: why this approach — constraints, trade-offs, what it enables
   - **Alternatives considered**: what was rejected and why
   - **Confidence**: High (sourced from conversation/docs, green border), Medium (inferred from code, blue border, labeled "inferred"), Low (not recoverable, amber border, with "rationale not recoverable — document before committing" warning)

10. **Re-entry context** — A concise "note from present-you to future-you":
    - **Key invariants**: assumptions the changed code relies on that aren't enforced by types or tests
    - **Non-obvious coupling**: files or behaviors connected in ways that aren't visible from imports alone
    - **Gotchas**: things that would surprise someone modifying this code in two weeks
    - **Don't forget**: follow-up work required (migration, config update, docs)

**Data gathering (what the agent reads before generating):**
- `git diff --stat` and `git diff --name-status` for file-level overview
- Line counts comparing key files between ref and working tree
- New public API surface (exported symbols, public functions, classes, interfaces)
- Feature inventory (actions, keybindings, config fields, event types)
- All changed files in full, including surrounding code paths
- CHANGELOG.md and README.md/docs for housekeeping status
- Conversation history, progress docs, commit messages, and PR descriptions for decision rationale

**Verification checkpoint:** Before generating HTML, the agent produces a structured fact sheet of every claim it will present — every number, every function name, every behavior description — cited to its source. This prevents hallucinated details.

**When NOT to use it:** For tiny changes (renamed a variable, fixed a typo). The overhead of a full 10-section review isn't worth it for trivial diffs.

---

### 3. `/plan-review` — Pre-Coding Validation

**What it does:** Takes a plan/spec/RFC document and cross-references every claim against the actual codebase. Produces current vs. planned architecture diagrams, change-by-change breakdown, risk assessment, and gap analysis.

**When to use it:**
- You wrote (or received) an implementation plan and want to validate it before starting work
- You want to check if a plan's assumptions about current code are actually correct
- You need to identify the blast radius of proposed changes
- You want to surface risks and edge cases before writing code

**How:**
```
/plan-review ~/docs/refactor-plan.md
/plan-review ./specs/auth-redesign.md
/plan-review ~/notes/migration-plan.md ./backend    # plan + specific codebase dir
```

**Inputs:**
- First argument: path to a markdown plan, spec, or RFC document
- Second argument (optional): path to a specific codebase directory (defaults to current working directory)

**Why this exists (and why it's different from `/diff-review`):** `/diff-review` looks **backward** at changes already made. `/plan-review` looks **forward** at changes proposed. The critical difference is that `/plan-review` *verifies assumptions* — it checks whether the files, functions, and behaviors the plan references actually exist and work the way the plan claims. Plans often describe code that has since changed, reference functions by wrong names, or miss dependencies.

**The 9 sections it produces:**

1. **Plan summary** — Leads with the *intuition*: what problem does this plan solve, and what's the core insight behind the approach? Then scope: how many files touched, estimated scale, new modules or tests planned. Hero depth treatment.

2. **Impact dashboard** — Files to modify, create, delete. Estimated lines added/removed. New test files planned. Dependencies affected. Includes a **completeness** indicator: whether the plan covers tests (green/red), docs updates (green/yellow/red), and migration/rollback (green/grey for N/A).

3. **Current architecture** — Mermaid diagram of how the affected subsystem works *today*. Focuses only on the parts the plan touches. With zoom controls.

4. **Planned architecture** — Mermaid diagram of how the subsystem will work *after* the plan is implemented. Uses the same node names and layout direction as the current architecture diagram so the visual diff is obvious. New nodes are highlighted with glow or accent border, removed nodes have strikethrough or reduced opacity, changed edges use a different stroke color.

5. **Change-by-change breakdown** — For each change in the plan, a side-by-side panel:
   - **Left (current):** what the code does now, with relevant snippets or function signatures
   - **Right (planned):** what the plan proposes, with the plan's own code examples if provided
   - **Rationale:** extracted from the plan's reasoning, rejected alternatives, or inline justifications
   - Flags discrepancies where the plan's description of current behavior doesn't match the actual code
   - Flags changes where the plan says *what* but not *why* — pre-implementation cognitive debt

6. **Dependency & ripple analysis** — What other code depends on the files being changed. Table or Mermaid graph showing callers, importers, and downstream effects. Color-coded: covered by plan (green), not mentioned but likely affected (amber), definitely missed (red). Collapsible by default.

7. **Risk assessment** — Styled cards for:
   - **Edge cases** the plan doesn't address
   - **Assumptions** the plan makes that should be verified
   - **Ordering risks** if changes need specific sequencing
   - **Rollback complexity** if things go wrong
   - **Cognitive complexity** — areas where the plan introduces non-obvious coupling, action-at-a-distance behavior, implicit ordering requirements, or contracts that exist only in the developer's memory. Each flag gets a severity indicator (low/medium/high) and a concrete mitigation suggestion.

8. **Plan review** — Structured Good/Bad/Ugly analysis of the plan itself (same card pattern as `/diff-review` code review, with green/red/amber/blue left-border accents).

9. **Understanding gaps** — A closing dashboard:
   - Count of changes with clear rationale vs. missing rationale (visual bar chart or progress indicator)
   - List of cognitive complexity flags with severity
   - Explicit recommendations: "Before implementing, document the rationale for changes X and Y"
   - Makes cognitive debt visible *before* the work starts, when it's cheapest to address

**Data gathering (what the agent reads before generating):**
- The plan file in full — extracts problem statement, proposed changes, rejected alternatives, scope boundaries
- Every file the plan references, plus files that import/depend on those files
- Blast radius mapping: what imports the affected files, what tests exist, config/schema files that need updates, public API surface
- Cross-reference: does the file/function/type the plan references actually exist? Does the plan's description match what the code actually does?

**Verification checkpoint:** Same as `/diff-review` — structured fact sheet of every claim, cited to source.

**When NOT to use it:** When you don't have a written plan. If you're just exploring an idea, use `/generate-web-diagram` to visualize the architecture first, then write a plan, then run `/plan-review` on it.

---

### 4. `/project-recap` — Context Recovery

**What it does:** Scans recent git activity and produces a full mental model snapshot: architecture diagram, activity narrative, decision log, state-of-things dashboard, cognitive debt hotspots, and next steps.

**When to use it:**
- You're coming back to a project after days (or weeks) away
- You're onboarding someone to a project
- You want a snapshot of project health and momentum
- You need to identify where understanding is weakest (cognitive debt)

**How:**
```
/project-recap                  # default: last 2 weeks
/project-recap 2w               # last 2 weeks (explicit)
/project-recap 30d              # last 30 days
/project-recap 3m               # last 3 months
```

**Time window parsing:**

| Argument | Parsed as |
|---|---|
| *(none)* | `2w` (2 weeks, the default) |
| `2w` | `"2 weeks ago"` |
| `30d` | `"30 days ago"` |
| `3m` | `"3 months ago"` |
| Other text | Treated as free-form context, uses default 2w window |

**Why this is different from just reading git log:** `git log` gives you *what* happened. `/project-recap` gives you *what it means*. It groups commits by theme (features, fixes, refactors, infra), extracts decision rationale from commit messages and docs, identifies cognitive debt hotspots (recently changed code with no documented reasoning), and provides the "mental model essentials" — the 5-10 things you need to hold in your head to work effectively.

**The 8 sections it produces:**

1. **Project identity** — Not the README blurb. A *current-state* summary: what this project does, who uses it, what stage it's at (early dev, stable, actively shipping features). Includes version, key dependencies, and the one-sentence "elevator pitch" for someone who forgot what they were building.

2. **Architecture snapshot** — Mermaid diagram of the system as it exists today. Focuses on conceptual modules and their relationships, not every file. Labels nodes with what they do, not just file names. This is the visual anchor — the rest of the page hangs off this diagram. With zoom controls.

3. **Recent activity** — Not raw git log. A human-readable narrative grouped by theme: feature work, bug fixes, refactors, infrastructure. Timeline visualization with the most significant changes called out. For each theme, a one-sentence summary of what happened and why it mattered.

4. **Decision log** — Key design decisions from the time window. Extracted from commit messages, conversation history, plan docs, progress docs. Each entry: what was decided, why, what was considered. This is the highest-value section for fighting cognitive debt — the reasoning that evaporates first.

5. **State of things** — A KPI-style dashboard:
   - What's **working** (stable, shipped, tested)
   - What's **in progress** (uncommitted work, open branches, active TODOs)
   - What's **broken or degraded** (known bugs, failing tests, tech debt items)
   - What's **blocked** (waiting on external input, dependencies, decisions)

6. **Mental model essentials** — The 5-10 things you need to hold in your head to work on this project effectively:
   - Key invariants and contracts (what must always be true)
   - Non-obvious coupling (things connected in ways you wouldn't guess from the file tree)
   - Gotchas (common mistakes, easy-to-forget requirements, things that break silently)
   - Naming conventions or patterns the codebase follows

7. **Cognitive debt hotspots** — Areas where understanding is weakest:
   - Code that changed recently but has no documented rationale
   - Complex modules with no tests
   - Areas where multiple people (or agents) made overlapping changes
   - Files that are frequently modified but poorly understood
   - Each gets a severity (red for high, amber for medium, blue for low) and a concrete suggestion (e.g., "add a doc comment to `buildCoordinationInstructions` explaining the 4 coordination levels — this function is called from 3 places and the behavior is non-obvious")

8. **Next steps** — Inferred from recent activity, open TODOs, project trajectory. Not prescriptive — just "here's where the momentum was pointing when you left." Includes any explicit next-step notes from progress docs or plan files.

**Data gathering (what the agent reads before generating):**
- `README.md`, `CHANGELOG.md`, `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` for project identity
- `git log --oneline --since=<window>`, `git log --stat --since=<window>`, `git shortlog -sn --since=<window>` for activity
- `git status` for uncommitted changes, `git branch --no-merged` for stale branches
- TODO/FIXME comments in recently changed files
- Progress docs (`~/.agent/memory/{project}/progress.md`, `~/.pi/agent/memory/{project}/progress.md`, `.pi/todos/`)
- Recent commit messages for decision rationale
- Key source files to understand module structure and dependencies

**Verification checkpoint:** Same as the others — structured fact sheet, cited to source.

**When NOT to use it:** When you need to review specific code changes (use `/diff-review`). When you need to validate a plan (use `/plan-review`). The recap is for *orientation*, not *analysis*.

---

### 5. `/fact-check` — Accuracy Verification

**What it does:** Takes any document that makes claims about code (an HTML review page, a plan doc, a spec) and verifies every single factual claim against the actual codebase. Corrects errors in-place and adds a verification summary.

**When to use it:**
- After running `/diff-review` or `/plan-review` — to verify the output is accurate
- On any document that references specific files, functions, line counts, or behaviors
- When you suspect a review or plan has stale information
- As a trust-but-verify step before sharing a document

**How:**
```
/fact-check                                          # verifies most recent file in ./.agent/diagrams/
/fact-check ./.agent/diagrams/auth-review.html       # specific HTML review
/fact-check ~/docs/migration-plan.md                 # a markdown plan
```

**Target resolution:**

| Argument | What gets verified |
|---|---|
| *(none)* | Most recently modified `.html` file in `./.agent/diagrams/` |
| Explicit path (`.html`) | That specific HTML file |
| Explicit path (`.md`) | That specific markdown file |
| Any other text file | Extracts and verifies whatever factual claims about code it contains |

**Auto-detected document types:**
- **HTML review pages** (diff-review, plan-review, project-recap): detected from page content, verified against the git ref or plan file the review was based on
- **Plan/spec documents** (markdown): verifies file references, function/type names, behavior descriptions, and architecture claims against the current codebase
- **Any other document**: extracts and verifies whatever factual claims about code it contains

**Why this command exists:** AI-generated reviews can hallucinate details — wrong function names, fabricated line counts, swapped before/after descriptions. `/fact-check` is the antidote. It treats the document as a set of verifiable claims and checks each one against ground truth.

**The 5-phase process:**

1. **Extract claims** — Reads the file and extracts every verifiable factual claim:
   - **Quantitative**: line counts, file counts, function counts, module counts, test counts, any numeric metrics
   - **Naming**: function names, type names, module names, file paths referenced
   - **Behavioral**: descriptions of what code does, how things work, before/after comparisons
   - **Structural**: architecture claims, dependency relationships, import chains, module boundaries
   - **Temporal**: git history claims, commit attributions, timeline entries
   - Skips subjective analysis (opinions, design judgments, readability assessments) — these aren't verifiable facts.

2. **Verify against source** — For each extracted claim, goes to the source:
   - Re-reads every file referenced — checks function signatures, type definitions, behavior descriptions against the actual code
   - For git history claims: re-runs git commands and compares output against the document's numbers
   - For diff-reviews: reads both the ref version (`git show <ref>:file`) and working tree version to verify before/after claims aren't swapped or fabricated
   - For plan docs: verifies that files, functions, and types referenced actually exist and behave as described
   - For project-recaps: re-runs `git log` commands to verify activity narrative and timeline
   - Classifies each claim as: **Confirmed** (matches exactly), **Corrected** (was inaccurate), or **Unverifiable** (can't be checked)

3. **Correct in place** — Edits the file directly using surgical text replacements:
   - Fixes incorrect numbers, function names, file paths, behavior descriptions
   - Fixes before/after swaps (a common error class in review pages)
   - Rewrites fundamentally wrong sections while preserving surrounding structure
   - For HTML: preserves layout, CSS, animations, Mermaid diagrams (unless they contain factual errors)
   - For markdown: preserves heading structure, formatting, and document organization

4. **Add verification summary** — Inserts a verification section:
   - **HTML files**: banner at the top or final section, matching the page's existing styling
   - **Markdown files**: `## Verification Summary` section appended at the end
   - Includes: total claims checked, claims confirmed (with count), corrections made (with brief list), unverifiable claims flagged

5. **Report** — Tells you what was checked, what was corrected, and opens the file.

**What it does NOT do:** It doesn't change opinions, design judgments, or analysis. It doesn't restructure the document. It is purely a fact-checker — it verifies that the data presented matches reality, corrects what doesn't, and leaves everything else alone.

**When NOT to use it:** On documents that don't make factual claims about code (pure opinion pieces, design mockups, meeting notes without technical references).

---

### 6. `/generate-slides` — Presentation Format

**What it does:** Generates a magazine-quality slide deck as a self-contained HTML page with keyboard/touch/scroll navigation.

**When to use it:**
- You need to present findings to a team
- You want a slide format instead of a scrollable page
- You're preparing for a meeting or demo

**How:**
```
/generate-slides our Q1 infrastructure migration
/generate-slides overview of the payment processing pipeline
```

**The `--slides` flag on other commands:** You don't always need `/generate-slides` directly. Any of the other commands can output as slides:

```
/diff-review --slides              # diff review as a slide deck
/plan-review --slides ~/plan.md    # plan review as slides
/project-recap --slides 2w         # project recap as slides
```

**Why slides are a separate format:** Slides are a different medium. Each slide is exactly one viewport tall (100dvh) with no scrolling. Typography is 2-3x larger. The agent composes a **narrative arc** (impact → context → deep dive → resolution) rather than just paginating content. The `--slides` flag tells the command to gather data using its normal pipeline but present it as a story rather than a reference document.

**Content completeness:** Changing the medium does not mean dropping content. Every section, decision, data point, specification, and collapsible detail from the source must appear in the deck. If a plan has 7 sections, the deck covers all 7. A 22-slide deck that covers everything beats a 13-slide deck that looks polished but is missing 40% of the source. Add more slides rather than cutting content.

**10 slide types available:**

| Type | Use for |
|---|---|
| **Title** | Opening slide with topic, subtitle, date |
| **Section Divider** | Transition between major topics |
| **Content** | Body text, bullet points, key takeaways |
| **Split** | Two-column layout (before/after, comparison, text+image) |
| **Diagram** | Mermaid diagrams, architecture visuals |
| **Dashboard** | KPI cards, metrics, progress indicators |
| **Table** | Data tables, comparison matrices |
| **Code** | Code snippets with syntax context |
| **Quote** | Key quotes, pull quotes, important callouts |
| **Full-Bleed** | Edge-to-edge image or background with overlay text |

**Compositional variety:** Consecutive slides must vary their spatial approach — centered, left-heavy, right-heavy, split, edge-aligned, full-bleed. Three centered slides in a row means push one off-axis.

**4 curated presets:**

| Preset | Feel |
|---|---|
| **Midnight Editorial** | Dark, sophisticated, serif headlines |
| **Warm Signal** | Light, approachable, earth tones |
| **Terminal Mono** | Hacker aesthetic, monospace, green/amber |
| **Swiss Clean** | Minimal, precise, geometric sans-serif |

**Navigation controls in the generated deck:**
- Arrow keys (left/right) for previous/next slide
- Touch swipe on mobile
- Mouse wheel/scroll
- Progress bar and nav dots
- Slide counter
- Keyboard hints that auto-fade after first interaction

---

## Scroll-Driven Commands (New 6)

These six commands produce pages that use **GSAP ScrollTrigger + Lenis** for scroll-paced animation. Instead of showing everything at once, content reveals progressively as you scroll — pinned sections hold context while related content scrolls in, SVG paths draw to show connections, code highlights shift to show different regions, and depth levels expand as you advance. The reader controls pacing with their scroll speed.

> **New to GSAP and Lenis?** See the [Understanding GSAP and Lenis](#understanding-gsap-and-lenis) section above for a full explanation of what these libraries are, why they're used, which features each command uses, and how they degrade for accessibility.

**What makes these different from the static commands:**

| Feature | Static pages | Scroll-driven pages |
|---|---|---|
| Animation trigger | Page load | Scroll position |
| Content visibility | Everything visible immediately | Content reveals as you scroll to it |
| Pinned sections | None | Key context stays fixed while related content scrolls |
| Pacing | Reader scans freely | Scroll speed = reading speed |
| SVG animation | Static or load-animated | Edge paths draw on scroll via DrawSVG |
| Text effects | CSS fadeUp | SplitText character/word stagger on scroll |
| Progress feedback | TOC active state only | Scroll progress bar + pin indicators |
| Smooth scroll | Native `scrollIntoView` | Lenis with momentum easing |
| Reduced motion | CSS animations disabled | All GSAP animations skipped, content shown immediately |

---

### 7. `/trace-flow` — Scroll-Driven Code Path Walkthrough

**What it does:** Traces how a request travels through your code — endpoint → middleware → validation → service → database — and presents each layer as a pinned section. You scroll to advance through the layers. The left panel shows code with highlighted lines; the right panel shows the data payload shape at each stage.

**When to use it:**
- You need to understand or explain how a request flows through your system
- You're debugging a multi-layer issue and need to see each step
- You're onboarding someone who needs to trace a critical path
- You want to document a complex flow for a code review or architecture discussion

**How:**
```
/trace-flow "POST /api/orders"
/trace-flow "POST /api/orders" --entry src/routes/orders.ts
/trace-flow "user login flow"
/trace-flow "WebSocket connection lifecycle"
/trace-flow "background job processing pipeline"
```

**Arguments:**
- `$1` (required): The request path or flow to trace. Can be an HTTP endpoint (`"POST /api/orders"`), a descriptive flow name (`"user login flow"`), or any identifiable code path.
- `--entry <file>` (optional): The entry point file. If not provided, the agent searches for route definitions, handlers, or controllers matching the path.

**Example — tracing an API endpoint:**

```
/trace-flow "POST /api/orders" --entry src/routes/orders.ts
```

This produces a page with:
1. A hero section with "POST /api/orders" in large editorial type (SplitText character animation)
2. An overview Mermaid flowchart showing all layers at a glance (edges draw in via DrawSVG on scroll)
3. **Layer 1: Route Handler** — pinned. Left panel shows `orders.ts` with the handler function highlighted. Right panel shows the incoming request shape (`{ items: [...], shipping: {...} }`)
4. **Layer 2: Auth Middleware** — pinned. Code shifts to show auth check. Payload panel shows the validated user context being attached.
5. **Layer 3: Validation** — pinned. Schema validation code highlighted. Payload shows validated/transformed shape.
6. **Layer 4: Order Service** — pinned. Business logic highlighted. Payload shows the constructed order object.
7. **Layer 5: Database** — pinned. Query code highlighted. Payload shows the SQL/ORM call and return shape.
8. Error paths section — what happens when auth fails, validation rejects, DB errors
9. Summary table — all layers with timing, file paths, data transformations

Each pinned layer lasts at most 2x viewport height. A scroll indicator on the right shows progress through each layer. Scrolling is smooth via Lenis.

**Example — tracing a WebSocket lifecycle:**

```
/trace-flow "WebSocket connection lifecycle"
```

The agent finds your WebSocket handler, traces the connection → authentication → message handling → disconnection → cleanup path, and presents each phase as a pinned scroll section with relevant code.

**What the agent reads:** It follows function calls from the entry point through each layer, reading every file involved. For each layer, it captures the file path, function name, data shape at entry/exit, key transformations, error handling, and transitions to the next layer.

**Best for:** Backend developers who need to understand or explain request flows, API endpoints, event processing pipelines, or any multi-step code path.

---

### 8. `/changelog-story` — Scrollytelling Project Timeline

**What it does:** Transforms your git history into a narrative timeline. A central SVG line draws downward as you scroll. Milestone cards animate in by phase. Contributor heatmap cells fill over time. The "current date" header pins and updates as you scroll through the timeline.

**When to use it:**
- You need to present what happened over a sprint/quarter/release cycle
- You want to understand the *arc* of a project — momentum shifts, pivots, phases
- You're preparing a retrospective or stakeholder update
- You want to see contributor activity patterns over time

**How:**
```
/changelog-story                   # default: last 3 months
/changelog-story 2w                # last 2 weeks
/changelog-story 30d               # last 30 days
/changelog-story 3m                # last 3 months (explicit)
/changelog-story 6m                # last 6 months
/changelog-story 1y                # last year
```

**Time window parsing:**

| Argument | Parsed as |
|---|---|
| *(none)* | `3m` (3 months, the default) |
| `2w` | `"2 weeks ago"` |
| `30d` | `"30 days ago"` |
| `3m` | `"3 months ago"` |
| `6m` | `"6 months ago"` |
| `1y` | `"1 year ago"` |
| Other text | Treated as free-form context, uses default 3m window |

**Example — a quarter retrospective:**

```
/changelog-story 3m
```

This produces a page with:
1. Hero: "3 Months of [Project Name]" (SplitText character animation). KPI cards: 142 commits, 4 contributors, 87 files changed, +3,200 / -1,100 lines
2. Central SVG timeline that draws downward as you scroll (DrawSVG + scrub)
3. **Phase 1: "Foundation Sprint" (Jan 5 – Jan 19)** — phase divider pins briefly with SplitText. Milestone cards batch-animate in: "Initial schema design", "Auth module scaffold", "CI pipeline setup". Cards alternate left/right of the timeline.
4. **Phase 2: "Feature Push" (Jan 20 – Feb 10)** — milestone cards for each major feature commit. Cards show commit hash, files touched, and a one-sentence description of what changed.
5. **Phase 3: "Stabilization" (Feb 11 – Feb 28)** — bug fixes, test additions, documentation updates. Milestone cards with green-tinted borders.
6. Sticky "Current Date" header at top updates to show which date you've scrolled to
7. Contributor heatmap — rows for each contributor, columns for weeks. Cells fill with color intensity as you scroll past them (batch animation).
8. File activity map — treemap or bars showing which modules got the most attention
9. Arc summary — narrative capturing where the project was at the start, the momentum shifts, and where it stands now

**Example — a sprint review:**

```
/changelog-story 2w
```

Same structure but compressed: fewer phases, daily rather than weekly granularity, focus on individual commits rather than grouped milestones.

**What the agent reads:** Full `git log` with stats for the time window. Contributor activity per date. Tags and version bumps. CHANGELOG.md entries. It analyzes commit messages and timing to identify natural phases.

**Best for:** Engineering managers preparing retrospectives, teams reviewing sprint progress, anyone who wants to see the story behind the commits.

---

### 9. `/deep-dive` — Progressive Depth Module Exploration

**What it does:** Explores a module at 4 zoom levels: bird's eye → components → internals → implementation detail. As you scroll, the diagram literally zooms in — overview nodes expand to reveal internal structure, labels gain detail, new connections draw in. Scroll backward to collapse deeper levels.

**When to use it:**
- You need to understand a complex module without getting overwhelmed
- You're writing documentation for a module at multiple audience levels
- You want a single artifact that works for both newcomers (overview) and implementers (detail)
- You need to audit a module's internal architecture

**How:**
```
/deep-dive src/auth/
/deep-dive src/auth/ "Authentication Module"
/deep-dive lib/parser.ts "Query Parser"
/deep-dive packages/core/ "Core Package"
```

**Arguments:**
- `$1` (required): Path to the module or directory to explore
- `$2` (optional): Display title. If not provided, derived from the module path.

**Example — exploring an auth module:**

```
/deep-dive src/auth/ "Authentication Module"
```

This produces a page with:
1. Hero: "Authentication Module" (SplitText). KPI cards: 8 files, 23 exports, 1,240 lines, 78% test coverage
2. **Depth 0 — Bird's Eye (pinned, scrub-driven):** The auth module as a single box in a Mermaid diagram. External connections: "HTTP Request" → Auth Module → "User Session". As you scroll, the box opens and transforms into the Depth 1 view.
3. **Depth 1 — Components (pinned, scrub-driven):** The box has expanded into 4 connected components: `JWTValidator`, `SessionStore`, `PermissionResolver`, `AuthMiddleware`. Connection lines draw in via DrawSVG. Labels show one-line purposes. Scrolling further zooms into each component.
4. **Depth 2 — Internals:** Key functions and types for each component. `JWTValidator.verify()`, `SessionStore.get()`, `PermissionResolver.check()`. Code snippets with annotations. Data flow arrows showing how a token becomes a validated session.
5. **Depth 3 — Implementation Detail:** Critical code paths: how JWT expiry is handled (line-by-line annotation in a pinned code panel), the Redis session cache strategy, the permission check algorithm with O(n) complexity note, edge case handling for revoked tokens.
6. Cross-cutting concerns: error handling strategy (throw vs. return), logging integration, config surface (`JWT_SECRET`, `SESSION_TTL`, `REDIS_URL`)
7. Summary table: all 23 exports with type, purpose, and which depth level covers them

**The scroll-to-zoom experience:** The transition between depth levels is continuous, not binary. At Depth 0, you see one box. As you scroll, the box's borders dissolve and the Depth 1 components grow outward from its center. Scroll further and each component opens to reveal Depth 2 content. Scroll backward and deeper levels collapse back into their parents. The scrub parameter ties animation progress directly to scroll position.

**Example — a single-file deep dive:**

```
/deep-dive lib/parser.ts "Query Parser"
```

Works the same way but with internal functions instead of files: Depth 0 = the parser as a black box, Depth 1 = the parser's major phases (tokenize → parse → transform → emit), Depth 2 = key functions per phase, Depth 3 = implementation details of critical algorithms.

**Best for:** Senior engineers auditing modules, tech leads reviewing architecture, anyone writing or reading documentation at multiple levels of detail.

---

### 10. `/migration-guide` — Step-by-Step Animated Migration

**What it does:** Breaks a migration into ordered steps and presents each as a pinned section with animated code transformations. Removed lines slide out red, added lines slide in green, moved lines physically travel to their new position. A persistent progress bar tracks your position across all steps.

**When to use it:**
- You're planning or documenting a migration (framework upgrade, API change, pattern refactor)
- You need developers to follow steps in order without losing their place
- You want a visual guide that shows *how* code transforms, not just before/after snapshots
- You're migrating a shared codebase and need a shareable walkthrough

**How:**
```
/migration-guide "Express 4 to Express 5"
/migration-guide "Express 4 to Express 5" --scope src/server/
/migration-guide "React class components to hooks"
/migration-guide "CommonJS to ESM" --scope src/lib/
/migration-guide "Sequelize to Prisma"
/migration-guide "REST to GraphQL" --scope src/api/
```

**Arguments:**
- `$1` (required): Migration description — what you're migrating from and to
- `--scope <path>` (optional): Directory to focus the analysis on. Without it, the entire project is analyzed.

**Example — Express 4 to Express 5:**

```
/migration-guide "Express 4 to Express 5" --scope src/server/
```

This produces a page with:
1. Hero: "Express 4 → Express 5" (SplitText). KPI cards: 7 steps, 12 files affected, ~340 line changes. Brief description of why this migration matters.
2. Overview pipeline: all 7 steps as circles on an SVG path, color-coded green/amber/red by risk level. The path draws via DrawSVG as you scroll.
3. **Step 1: Update package.json (pinned, scrub-driven)**
   - Step header pins at top: "Step 1 of 7 — Update Dependencies" with green risk badge
   - Before panel: `"express": "^4.18.2"` highlighted in red
   - Animated transformation: the version number slides out, `"^5.0.0"` slides in green
   - Explanation card: "Express 5 requires Node.js 18+. Check your CI and production environment."
   - Gotcha card (amber): "Express 5 drops `app.del()` — use `app.delete()` instead"
4. **Step 2: Replace deprecated middleware (pinned)**
   - Before: `app.use(bodyParser.json())` — line slides out red
   - After: `app.use(express.json())` — line slides in green
   - Explanation: "Body-parser was built into Express since 4.16 but 5.x removes the bundled legacy version"
5. **Step 3: Update error handlers (pinned)** — error handler signature changes from `(err, req, res, next)` behavior
6. ...continues through all 7 steps...
7. Persistent progress bar: "Step 3 of 7" tracks position across all pinned sections
8. Dependency changes table: packages to add, remove, and version-bump
9. Test updates: what test files need changes
10. Rollback plan (collapsed `<details>`): how to revert if something goes wrong
11. Completion checklist: all steps as a reference checklist

**Example — React class to hooks:**

```
/migration-guide "React class components to hooks"
```

Steps include: replace `this.state` with `useState`, replace `componentDidMount` with `useEffect`, extract shared logic into custom hooks, update refs from `createRef` to `useRef`, etc. Each step shows actual component code from the project transforming in place.

**The code transformation animation:** Within each pinned step, scrolling drives the transformation. Lines that are being removed slide left and fade with a red tint. Lines being added slide in from the right with a green tint. Lines that *moved* (e.g., a function extracted to a different location) physically travel from their old position to their new one. The scrub parameter ties all animation to scroll position — scroll backward to see the reverse transformation.

**Best for:** Teams executing shared migrations, developers documenting upgrade paths, anyone who needs a follow-along guide that can't be skipped or done out of order.

---

### 11. `/dependency-explorer` — Scroll-Driven Dependency Graph

**What it does:** Starts with your direct dependencies and progressively reveals transitive depth levels as you scroll. Nodes fly outward from parents, connection lines draw in, and the graph expands with each depth level. Scroll backward to collapse deeper levels.

**When to use it:**
- You need to understand your dependency tree beyond the top level
- You want to trace why a specific transitive dependency is in your tree
- You're auditing for duplicate packages, version conflicts, or security vulnerabilities
- You want to visualize the weight of your dependency chains

**How:**
```
/dependency-explorer
/dependency-explorer --focus lodash
/dependency-explorer --focus react --depth 4
/dependency-explorer --dev
/dependency-explorer --focus express --depth 2
```

**Arguments:**
- `--focus <package>` (optional): Highlight a specific package and all paths leading to it
- `--depth <n>` (optional): Maximum depth to explore (default: 3)
- `--dev` (optional): Include devDependencies (excluded by default)

**Example — exploring all dependencies:**

```
/dependency-explorer
```

This produces a page with:
1. Hero: "[Project Name] Dependencies" (SplitText). KPI cards: 24 direct deps, 387 total transitive, max depth 7, 3 duplicates, 1 vulnerability
2. **Depth 0 — Direct dependencies:** Project root in the center with 24 direct dependencies shown as a ring of cards. Each card shows package name, version, and transitive dep count (e.g., "express v4.18.2 — pulls 32 transitive deps").
3. **Depth 1 (scroll to reveal):** First-level transitive dependencies fly outward from their parent nodes. Connection lines draw in via DrawSVG. `ScrollTrigger.batch()` staggers nodes so they don't appear all at once.
4. **Depth 2 (scroll further):** Second-level transitives expand the graph. The viewport rescales to fit. Pinning keeps the graph centered while new layers appear within it.
5. **Depth 3 (scroll further):** Deeper levels continue. Scroll backward to collapse — nodes fade out, connections retract.
6. Duplicates section: cards highlighting packages at multiple versions, color-coded by conflict severity
7. Security findings: vulnerability cards with severity badges (critical/high/medium/low)
8. Heaviest chains: ranked list of dependency chains by total transitive count
9. Summary table: all direct dependencies with transitive counts, duplicate flags, vulnerability flags

**Example — tracing a specific package:**

```
/dependency-explorer --focus lodash
```

Same graph exploration, but after expanding to show lodash's location in the tree, a pinned spotlight section highlights all paths from root → lodash. Unrelated nodes dim to 20% opacity. Trace lines glow to show which direct dependencies are responsible for pulling lodash in. Useful for answering "why is this in my tree and can I remove it?"

**Example — auditing dev dependencies:**

```
/dependency-explorer --dev
```

Includes devDependencies in the tree. Useful for auditing test framework chains, build tool dependencies, or understanding why `node_modules` is large.

**Best for:** Platform engineers auditing dependency weight, security engineers tracing vulnerability chains, anyone who's ever wondered "why do we have 400 transitive dependencies?"

---

### 12. `/onboarding-walkthrough` — Guided Codebase Introduction

**What it does:** Creates a scroll-paced guided tour of your codebase for new team members. Each concept section pins and plays its explanation before releasing — enforced scroll pacing ensures comprehension. Code annotation panels highlight different regions as you scroll. The file tree reveals progressively.

**When to use it:**
- A new developer is joining the team and needs a curated introduction
- You want to document your codebase's architecture and conventions in an interactive format
- You need a self-serve onboarding artifact that doesn't require a senior engineer's time
- You want to replace scattered README + CONTRIBUTING + tribal knowledge with a single walkthrough

**How:**
```
/onboarding-walkthrough
/onboarding-walkthrough --audience "backend developer joining the team"
/onboarding-walkthrough --audience "frontend intern"
/onboarding-walkthrough --audience "senior engineer from a different stack"
/onboarding-walkthrough --focus "API layer"
/onboarding-walkthrough --audience "new QA engineer" --focus "testing infrastructure"
```

**Arguments:**
- `--audience <description>` (optional): Who the walkthrough is for. Adjusts depth and assumed knowledge. Default: mid-level developer.
- `--focus <area>` (optional): Narrow to a specific area of the codebase. Without it, covers the full codebase.

**Example — full team onboarding:**

```
/onboarding-walkthrough --audience "backend developer joining the team"
```

This produces a page with:
1. Welcome: "[Project Name]" (SplitText). "A backend service that processes payment webhooks and routes them to downstream services." Assumed background: familiar with Node.js/TypeScript, new to this codebase. "By the end, you'll understand: the request lifecycle, how events are processed, the testing strategy, and how to ship your first PR."
2. **The Big Picture (pinned):** Mermaid architecture diagram showing Webhook Receiver → Event Router → Payment Processor → Notification Service → Database. Labels and connections draw in via DrawSVG. Zoom controls with `data-lenis-prevent`. This diagram stays pinned while the overview text scrolls in beside it.
3. **Concept 1: The Event Router (pinned section)**
   - "Event Router" title with SplitText emphasis
   - "This is the heart of the system. Every incoming webhook passes through here. It determines what kind of event it is and routes it to the right processor."
   - File tree highlights: `src/router/index.ts`, `src/router/rules.ts`, `src/router/types.ts` animate in via batch reveal
   - Code walkthrough: `src/router/index.ts` is shown in a pinned panel. As you scroll, different function regions highlight: first `parseEvent()`, then `matchRule()`, then `dispatch()`. Callout cards slide in for each region explaining what it does.
   - Key rule callout (accent card): "Every event type must have a matching rule in `rules.ts`. Adding a new event type without a rule causes a silent drop."
4. **Concept 2: Payment Processing (pinned section)** — same structure for the payment processor
5. **Concept 3: The Notification Pipeline** — same structure
6. **Concept 4: Error Handling and Retries** — how failed events are retried, dead letter queue
7. **Concept 5: The Database Layer** — ORM patterns, migration strategy, query patterns
8. File tree explorer: full project tree reveals directory-by-directory as you scroll. Each directory is color-coded: source (blue), tests (green), config (amber), docs (slate), build (dim).
9. Development setup: step-by-step pinned checklist — clone, install, copy `.env.example`, run migrations, start dev server, run tests
10. Common tasks: expandable cards — "How do I add a new event type?", "How do I write a test?", "How do I deploy?"
11. Patterns and conventions: naming conventions, error handling approach, PR review expectations
12. Gotchas: amber cards — "The webhook signature verification uses a clock tolerance of 5 minutes. If your local clock is off, verification fails silently." "The `PAYMENT_PROCESSOR_URL` env var must include the `/v2` path prefix — the code doesn't append it."
13. Where to go next: suggested first tasks based on the audience's role

**Example — focused onboarding:**

```
/onboarding-walkthrough --focus "API layer" --audience "frontend developer"
```

Narrower scope: only covers the API layer (endpoints, request/response shapes, auth, rate limiting). Adjusted depth: explains things a frontend developer would need to know (how to call endpoints, what responses look like) rather than backend internals.

**The enforced pacing experience:** Unlike a regular docs page where you can skim and skip, each concept section in the walkthrough *pins* — it holds the viewport while you scroll through its explanation. This isn't scroll-jacking (you can always scroll past), but it creates a natural pace that prevents the common onboarding failure of "I opened the README, skimmed for 30 seconds, and now I'm confused." The pin indicator on the right shows your progress through each concept.

**What the agent reads:** README, CONTRIBUTING, CLAUDE.md, ARCHITECTURE.md, package.json, entry points, key abstractions (types/interfaces/classes), config files, development scripts, recent git activity, TODO/FIXME/HACK comments, and ADRs if they exist.

**Best for:** Team leads onboarding new hires, open source maintainers creating contributor guides, anyone who wants their codebase to be self-documenting in a way that actually works.

---

## 13. Automatic Table Rendering (No Command Needed)

**What it does:** When the skill is loaded and the agent is about to render an ASCII table with 4+ rows or 3+ columns, it automatically generates an HTML page instead.

**You don't invoke this.** It happens proactively. The agent will tell you the file path and open it in the browser. You still get a brief text summary in chat, but the actual table is the HTML page.

**Trigger threshold:** 4+ rows or 3+ columns. Below that, ASCII is fine.

**What kinds of tables get auto-converted:**
- Requirement audits (request vs plan)
- Feature comparisons
- Status reports
- Configuration matrices
- Test result summaries
- Dependency lists
- Permission tables
- API endpoint inventories
- Any structured rows and columns meeting the threshold

**Why:** ASCII tables with pipes and dashes are unreadable beyond trivial sizes. The HTML version gets sticky headers, alternating row backgrounds, row hover highlights, status indicator badges (colored dots, not emoji), responsive horizontal scrolling, column width hints, proper text wrapping, code formatting in cells, and secondary detail text. It's the same content but actually scannable.

---

## Scroll-Driven Animation Features (Infrastructure)

Even when using the original 6 static commands, multi-section pages now benefit from scroll-driven enhancements. These aren't new commands — they're improvements to how existing pages animate and behave. See [Understanding GSAP and Lenis](#understanding-gsap-and-lenis) for what the underlying libraries are and how they work.

### What Changed

| Before | After |
|---|---|
| All `fadeUp` animations fire on page load, even for content below the fold | Below-the-fold content uses `gs-reveal` — animates when scrolled into view |
| TOC uses native `scrollIntoView({ behavior: 'smooth' })` with inconsistent browser easing | Lenis smooth scroll with configurable momentum and easing |
| Page titles use the same fadeUp as everything else | Hero headings use SplitText for editorial character/word stagger |
| Mermaid diagrams appear fully formed | DrawSVG animates edges drawing in on scroll |
| Before/after panels scroll past quickly | Pinned comparison panels hold the "before" while "after" scrolls in |
| No reading progress indication beyond TOC active state | Thin progress bar at viewport top, width driven by scroll position |

### When These Apply

The scroll enhancements apply automatically when the agent generates a multi-section page (4+ sections). Single-section diagrams still use CSS load animations. The agent decides based on content length — you don't need to request it.

### Reduced Motion

All scroll-driven features respect `prefers-reduced-motion: reduce`:
- Lenis is skipped — native browser scroll is used
- ScrollTrigger animations are skipped — content is visible immediately
- SplitText effects are skipped — headings display normally
- DrawSVG is skipped — SVG paths are fully drawn immediately
- Pinned sections still pin (pinning is a layout feature) but without animation
- The scroll progress bar is hidden
- The page is fully readable without any animation

To test: enable "Reduce motion" in your OS accessibility settings, then reload the page.

---

## Recommended Workflows

### Starting a Feature

```
/generate-web-diagram the system I'm about to change    # understand current state
# write your plan
/plan-review ~/plan.md                                   # validate before coding
```

**Why this order:** Visualizing first helps you write a better plan. The plan review then catches what you missed — wrong assumptions about current code, ripple effects, unaddressed edge cases. Fix the plan before writing code.

### Finishing a Feature

```
/diff-review                                             # review your changes
/fact-check                                              # verify the review is accurate
```

**Why the two-step pipeline:** `/diff-review` produces rich analysis — architecture diagrams, code review, decision log. But it can hallucinate details (wrong function names, fabricated line counts). `/fact-check` immediately after catches and corrects those errors. The result is a review you can trust and share.

### Presenting Your Work

```
/diff-review --slides                                    # slide deck of your changes
```

**Or for a standalone presentation:**
```
/generate-slides summary of our new auth system
```

### Returning to a Project

```
/project-recap                                           # rebuild mental model (last 2 weeks)
/project-recap 30d                                       # if you've been away longer
```

**Why this works:** The recap gives you architecture snapshot, decision log, state-of-things dashboard, and cognitive debt hotspots — everything you need to resume productive work without re-reading every file.

### Understanding a Complex Flow

```
/trace-flow "POST /api/orders" --entry src/routes/orders.ts
```

**Why this over a static diagram:** When a request crosses 5+ layers, a static diagram shows the topology but not the *narrative*. `/trace-flow` pins each layer so you can read the code, see the data shape, and understand the transformation before moving to the next layer. You control the pace.

### Preparing a Sprint Retrospective

```
/changelog-story 2w                                      # last sprint
/changelog-story 3m                                      # full quarter
```

**Why this over git log:** `git log` is a firehose. `/changelog-story` identifies phases, groups milestones, animates a timeline, and builds a contributor heatmap. It tells the *story* behind the commits — useful for retrospectives, stakeholder updates, and release notes.

### Understanding a Complex Module

```
/deep-dive src/auth/ "Authentication Module"
```

**Why this over reading the code:** You *could* read every file in `src/auth/`, but you'd start at an arbitrary depth level and miss the forest for the trees. `/deep-dive` starts at the highest level (what does this module do?) and lets you zoom in progressively. It's the difference between being handed a box of puzzle pieces and being shown the completed puzzle first.

### Planning a Migration

```
/migration-guide "Express 4 to Express 5" --scope src/server/
```

**Why this over a written migration doc:** Written migration docs are either too terse (a table of X→Y) or too verbose (20 pages nobody reads). `/migration-guide` is step-by-step with animated code transforms — developers see exactly how their code changes and can follow along at their own pace. The persistent progress bar prevents the common failure of "I lost my place at step 4."

### Auditing Dependencies

```
/dependency-explorer --focus lodash
/dependency-explorer --dev
```

**Why this over `npm ls`:** `npm ls` dumps a tree that's too large to parse visually. `/dependency-explorer` reveals depth levels progressively — you see direct deps first, then expand. The `--focus` flag traces specific packages through the tree. Duplicate and vulnerability detection is built in.

### Onboarding a New Team Member

```
/onboarding-walkthrough --audience "backend developer joining the team"
```

**Why this over pointing them at the README:** READMEs are reference docs — they list facts but don't teach. The walkthrough is a guided tour with enforced pacing: each concept pins and explains itself before releasing. It surfaces tribal knowledge (gotchas, naming conventions, implicit contracts) that never makes it into docs.

**For a focused introduction:**
```
/onboarding-walkthrough --focus "API layer" --audience "frontend developer"
```

### The Full Feature Lifecycle

```
# 1. Understand the codebase
/deep-dive src/payments/ "Payment Module"

# 2. Understand the current state
/generate-web-diagram the payment processing pipeline

# 3. Plan
# ... write your plan ...
/plan-review ~/plan.md

# 4. Build
# ... implement the feature ...

# 5. Review
/diff-review
/fact-check

# 6. Document the flow
/trace-flow "POST /api/payments/charge"

# 7. Prepare migration guide (if breaking changes)
/migration-guide "Payment API v1 to v2" --scope src/payments/

# 8. Present
/diff-review --slides

# 9. Onboard others
/onboarding-walkthrough --focus "payments" --audience "new backend engineer"
```

### Onboarding a Complete Team

```
# 1. Full codebase walkthrough
/onboarding-walkthrough --audience "mid-level developer joining the team"

# 2. Project history and context
/changelog-story 3m

# 3. Deep dive into the most complex module
/deep-dive src/core/ "Core Engine"

# 4. Key API flow walkthrough
/trace-flow "POST /api/main-endpoint"

# 5. Dependency landscape
/dependency-explorer
```

---

## Output Location

All generated HTML files are written to `./.agent/diagrams/` with descriptive filenames based on content (e.g., `auth-architecture.html`, `diff-review-feature-branch.html`, `trace-flow-post-api-orders.html`). The agent opens them in your browser automatically and tells you the file path so you can re-open or share them.

The directory persists across sessions.

## Path Resolution

The slash commands in `prompts/` (copied to `.claude/commands/`) reference skill files using **relative paths** like `./references/css-patterns.md` and `./templates/architecture.html`. These paths resolve relative to your **working directory** (where you launched Claude Code), not relative to where the command file lives.

This means the full repository — `SKILL.md`, `references/`, and `templates/` — must be present in your working directory for the commands to work correctly. The prompt files alone are not sufficient; they are entry points that tell the agent *what* to read, while the references and templates are *what gets read*.

### What lives where

| Location | Contents | Purpose |
|---|---|---|
| `SKILL.md` | Core workflow, diagram types, anti-slop rules | Agent reads this to know how to generate |
| `references/` | CSS patterns, Mermaid theming, font pairings, nav, slides, scroll animations | Agent reads before each generation for concrete code patterns |
| `templates/` | 5 HTML reference files with distinct palettes | Agent reads the matching template to absorb layout and style |
| `prompts/` | 12 markdown prompt templates | Copied to `.claude/commands/` to register as slash commands |
| `.claude/commands/` | Copies of `prompts/*.md` | Where Claude Code discovers slash commands |

### How it works in practice

**Working inside the skill repo** (development or learning): paths resolve automatically because `./references/` and `./templates/` are right there.

**Installed via `pi install` or `git clone` to a skills directory**: the package manager or install script handles path resolution. The skill files live in the skills directory and the agent knows where to find them.

**Manual install to a separate project**: clone the full repo somewhere accessible and copy only the prompts:

```bash
git clone https://github.com/nicobailon/visual-explainer.git ~/.claude/skills/visual-explainer
cp ~/.claude/skills/visual-explainer/prompts/*.md ~/.claude/commands/
```

In this setup, the relative paths in the commands (e.g., `./references/css-patterns.md`) will not resolve from an arbitrary project directory. The agent falls back to reading from the skill's install location if it can find it, or generates from memory if it cannot. Output quality is highest when the reference files are accessible.

---

## Quick Reference Card

| Command | One-liner | Key argument |
|---|---|---|
| `/generate-web-diagram` | Turn any topic into a visual HTML page | Topic description |
| `/diff-review` | Visual code review with decision log | Git ref (default: `main`) |
| `/plan-review` | Validate a plan against real code | Path to plan file |
| `/project-recap` | Rebuild mental model from git history | Time window (default: `2w`) |
| `/fact-check` | Verify factual accuracy of a document | Path to file (default: latest) |
| `/generate-slides` | Magazine-quality slide deck | Topic description |
| `/trace-flow` | Scroll-driven request path walkthrough | Request path + optional `--entry` |
| `/changelog-story` | Scrollytelling project timeline | Time window (default: `3m`) |
| `/deep-dive` | Progressive depth module exploration | Module path + optional title |
| `/migration-guide` | Animated step-by-step migration | Migration description + optional `--scope` |
| `/dependency-explorer` | Scroll-driven dependency graph | Optional `--focus`, `--depth`, `--dev` |
| `/onboarding-walkthrough` | Guided codebase introduction | Optional `--audience`, `--focus` |
| `--slides` | Add to any command for slide deck output | Flag on existing commands |
