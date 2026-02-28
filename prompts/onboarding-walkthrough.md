---
description: Generate a guided codebase introduction for new team members — scroll-paced walkthrough at their own speed
---
Load the visual-explainer skill, then generate a guided onboarding walkthrough as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template at `./templates/scroll-showcase.html`, the scroll animation patterns at `./references/scroll-animations.md`, CSS patterns at `./references/css-patterns.md`, and libraries at `./references/libraries.md` before generating. Use a warm editorial or paper/ink aesthetic — approachable, not intimidating. Vary fonts and palette from previous diagrams.

**Input parsing** — determine the audience and scope:
- `--audience <description>`: Optional audience description (e.g., `"backend developer joining the team"`, `"frontend intern"`, `"senior engineer from a different stack"`). Adjusts depth and assumed knowledge.
- `--focus <area>`: Optional focus area (e.g., `"API layer"`, `"deployment pipeline"`, `"data model"`). Narrows the walkthrough to a specific concern.
- If no arguments, create a comprehensive full-codebase walkthrough assuming a mid-level developer audience.

**Data gathering phase** — run these to understand the codebase:

1. **Project identity.** Read `README.md`, `CONTRIBUTING.md`, `CLAUDE.md`, `ARCHITECTURE.md`, or any similar docs. Read `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` for name, version, dependencies, scripts. Identify the project's purpose, stack, and conventions.

2. **File structure.** List the top-level directory tree. Identify the major directories and their roles. Read any existing documentation about project structure.

3. **Entry points.** Find the main entry point(s): `main.ts`, `index.js`, `app.py`, `main.go`, `src/lib.rs`, etc. Read them to understand how the application starts.

4. **Key abstractions.** Identify the 5-10 most important types, classes, or interfaces. Read their definitions. These are the mental models a new developer needs first.

5. **Configuration.** Read config files: `.env.example`, environment setup, build config, CI/CD. What does a new developer need to set up before they can run the project?

6. **Development workflow.** Read `CONTRIBUTING.md` or infer from package.json scripts: how to install, run, test, lint, build. Are there git hooks, pre-commit checks, or other workflow tools?

7. **Architecture patterns.** Identify the architectural pattern(s) in use: MVC, hexagonal, event-driven, microservices, monolith, etc. How do requests flow through the system? What's the data model?

8. **Recent activity.** `git log --oneline -20` and `git shortlog -sn --since="3 months ago"` for a sense of active areas and contributors.

9. **Tribal knowledge.** Look for TODO/FIXME/HACK/XXX comments, especially in frequently-changed files. These are the things that don't make it into docs. Read any ADRs (Architecture Decision Records) if they exist.

**Verification checkpoint** — before generating HTML:
- Every file path, function name, and type name verified against the codebase
- Every architectural claim checked against actual code structure
- Setup instructions tested or cross-referenced with existing docs
- No assumptions presented as facts

**Page structure** — scroll-paced walkthrough using GSAP pin + scrub:

1. **Welcome section** — project name and a warm, jargon-free one-sentence description. The audience and assumed background. "By the end of this walkthrough, you'll understand: [3-4 concrete outcomes]." SplitText on the project name. Above the fold.

2. **The big picture** — a high-level architecture diagram (Mermaid) showing the major components and how they connect. This is the mental model the rest of the walkthrough builds on. Pinned briefly while labels and connections draw in via DrawSVG. Zoom controls with `data-lenis-prevent`.

3. **Concept-by-concept walkthrough** — each major concept is a pinned section that plays its explanation before releasing. Enforced scroll pacing ensures the new developer absorbs each concept before moving on. For each concept:
   - **Concept name** with SplitText emphasis on key terms (e.g., "Repository Pattern", "Middleware Chain")
   - **Why it matters** — one paragraph, jargon-free
   - **Where to find it** — file tree reveals progressively via `ScrollTrigger.batch()`, highlighting the relevant files
   - **Code walkthrough** — pinned code panel where different regions highlight as the user scrolls, with callout cards sliding in to explain each section
   - **Key rule** — the one thing you must remember about this concept, styled as a prominent callout card

4. **File tree explorer** — the full project tree, revealed progressively on scroll. Each major directory group batch-animates in. Hover or scroll highlights show the purpose of each directory. Color-coded by role (source, tests, config, docs, build artifacts).

5. **Development setup** — step-by-step setup instructions as a scroll-driven checklist. Each step pins briefly to ensure it's read. Includes: clone, install, environment config, database setup (if any), running locally, running tests.

6. **Common tasks** — "How do I...?" section with expandable cards for common developer tasks: adding a new endpoint, writing a test, deploying, debugging locally. Each card has the command or file to modify and a brief explanation.

7. **Patterns and conventions** — the unwritten rules: naming conventions, error handling approach, testing strategy, code review expectations, git workflow. Styled as a reference card grid.

8. **Gotchas and tribal knowledge** — amber-tinted cards for things that would surprise a newcomer. Things that break silently, non-obvious coupling, known quirks. Each card has a severity indicator and a "what to do" suggestion.

9. **Where to go next** — suggested first tasks, key people to ask, useful documentation links, and the areas of the codebase to explore first based on the audience's role.

**GSAP patterns to use:**
- Concept sections: `ScrollTrigger.create({ pin: true })` for enforced pacing — each concept must be scrolled through
- Code annotation panels: `scrub` to control which code region is highlighted as the user scrolls
- File tree: `ScrollTrigger.batch()` for progressive directory reveals
- `SplitText` on key terms and the project name
- `DrawSVGPlugin` on architecture diagram connections
- Lenis smooth scroll with `data-lenis-prevent` on code blocks and Mermaid diagrams
- Scroll progress bar
- `prefers-reduced-motion` fallback: show all concepts expanded, skip pin animations, keep scroll pacing structure via visible section breaks

Include responsive section navigation. Write to `./.agent/diagrams/` and open in browser.

Ultrathink.

$@
