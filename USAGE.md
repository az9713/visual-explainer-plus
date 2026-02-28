# Visual Explainer — Command Usage Guide

A complete guide to using the 6 slash commands and 1 automatic behavior effectively. Covers when to use each, how to invoke them, why they exist, and recommended workflows.

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

---

## 1. `/generate-web-diagram` — The Swiss Army Knife

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

**Why this and not the others:** This is the only command with no opinion about *what* you're diagramming. The other 5 commands all have a specific data-gathering pipeline (git diffs, plan files, etc.). This one is pure "turn my idea into a visual page."

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

## 2. `/diff-review` — Post-Coding Review

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

## 3. `/plan-review` — Pre-Coding Validation

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

## 4. `/project-recap` — Context Recovery

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

## 5. `/fact-check` — Accuracy Verification

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

## 6. `/generate-slides` — Presentation Format

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

## 7. Automatic Table Rendering (No Command Needed)

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

### Onboarding Someone

```
/project-recap 3m                                        # full 3-month context
/generate-web-diagram our system architecture             # clean architecture visual
```

### Validating Any Document

```
/fact-check ~/docs/any-document-with-code-claims.md      # verify against codebase
```

### The Full Feature Lifecycle

```
# 1. Understand
/generate-web-diagram the system I'm about to change

# 2. Plan
# ... write your plan ...
/plan-review ~/plan.md

# 3. Build
# ... implement the feature ...

# 4. Review
/diff-review
/fact-check

# 5. Present
/diff-review --slides
```

---

## Output Location

All generated HTML files are written to `./.agent/diagrams/` with descriptive filenames based on content (e.g., `auth-architecture.html`, `diff-review-feature-branch.html`). The agent opens them in your browser automatically and tells you the file path so you can re-open or share them.

The directory persists across sessions.

## Path Resolution

The slash commands in `prompts/` (copied to `.claude/commands/`) reference skill files using **relative paths** like `./references/css-patterns.md` and `./templates/architecture.html`. These paths resolve relative to your **working directory** (where you launched Claude Code), not relative to where the command file lives.

This means the full repository — `SKILL.md`, `references/`, and `templates/` — must be present in your working directory for the commands to work correctly. The prompt files alone are not sufficient; they are entry points that tell the agent *what* to read, while the references and templates are *what gets read*.

### What lives where

| Location | Contents | Purpose |
|---|---|---|
| `SKILL.md` | Core workflow, diagram types, anti-slop rules | Agent reads this to know how to generate |
| `references/` | CSS patterns, Mermaid theming, font pairings, nav, slides | Agent reads before each generation for concrete code patterns |
| `templates/` | 4 HTML reference files with distinct palettes | Agent reads the matching template to absorb layout and style |
| `prompts/` | 6 markdown prompt templates | Copied to `.claude/commands/` to register as slash commands |
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
