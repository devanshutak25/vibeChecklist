# Vibe Coding: The Pre-Production & Prompting Checklist

A structured guide for getting high-fidelity code output from LLMs. Designed to eliminate the most common failure modes: wrong architecture, wrong aesthetics, wrong scope.

---

## How to Use This Document

**Do not read this document end-to-end for every project.**

1. Start with **Section 1: Project Routing** — it takes 2 minutes and tells you exactly which sections apply.
2. Always read **Section 2: Universal Pre-Production** — it applies to everything.
3. Read your **project-type section** (Section 3) — only the one that matches.
4. If your project has a web UI, read **Section 4: UI/UX Design Process** — depth varies by how design-critical your project is.
5. If Project Routing flagged your project as Medium or Complex, read **Section 5: Scope Estimation & Staging**.
6. If your project is Complex, read **Section 6: PRD Checklist**.
7. Read **Section 7: Reference Material Guide** — applies to everything, but depth varies by project.
8. Read **Section 8: Prompting Session Tactics** — this is your in-session playbook.
9. Bookmark **Section 9: Course Correction** — read it when things go wrong mid-session.
10. Bookmark **Section 10: Anti-Patterns** — skim before your first prompt, reference when debugging failures.

---

## Section 1: Project Routing

Answer these questions before anything else. They determine your path through this document.

### 1.1 — What Are You Building?

| If your project is primarily... | Read Section 3 subsection... |
|---|---|
| A web app, dashboard, landing page, SaaS product | 3A: Web Apps & SaaS |
| A game (any engine, any platform) | 3B: Games |
| A visual/immersive experience (VR, AR, installations, generative art) | 3C: Visual & Immersive |
| A CLI tool, automation script, or system utility | 3D: CLI Tools & Automation |
| A data pipeline, API, or backend service | 3E: Data Pipelines & Backend |
| A Python application or script (ML, data science, desktop app, tooling) | 3F: Python Applications & Scripts |
| A portfolio, marketing site, blog, or design-focused static site | 3G: Website Design & Static Sites |

If your project spans multiple categories, read all relevant subsections and note where requirements conflict — those conflicts are where LLMs fail hardest.

### 1.2 — How Complex Is It?

Use this rough heuristic:

**Simple** (single prompt, < 300 lines of output expected):
- One file or a handful of files
- No persistent state or database
- No authentication
- No multi-step user flows
- Examples: a utility script, a single React component, a landing page, a single-page portfolio, a CLI tool

**Medium** (3-8 staged prompts, 300-2000 lines):
- Multiple files/modules with dependencies between them
- Some state management
- 2-5 distinct user flows or features
- One integration (API, database, auth)
- Examples: a CRUD app, a simple game, a multi-page marketing site with blog, a multi-page site with forms, a data pipeline with multiple stages

**Complex** (requires PRD, 10+ staged prompts or iterative sessions):
- Multiple interacting systems
- Complex state management (multiplayer, real-time, undo/redo)
- 5+ distinct user flows
- Multiple integrations
- Performance-sensitive
- Examples: a multiplayer game, a SaaS product, an agency site with CMS, animations, and multi-language, a VR experience with physics, a distributed backend

**Write down your complexity level. You'll reference it throughout this document.**

| Complexity | Required Sections |
|---|---|
| Simple | 2, 3 (your type), 4 (if web UI), 7, 8 |
| Medium | 2, 3 (your type), 4 (if web UI), 5, 7, 8, 9 |
| Complex | 2, 3 (your type), 4 (if web UI), 5, 6, 7, 8, 9 |

---

## Section 2: Universal Pre-Production

**Every project, every time. No exceptions.**

This section covers what you must define before writing a single prompt, regardless of project type or complexity.

### 2.1 — Define the Core Output

Before you think about tech stack or features, write one sentence:

> "When this is done, a user will be able to ___."

If you can't fill that blank in one sentence, your scope is too vague. The LLM will fill the ambiguity with generic assumptions, and you'll get generic output.

**For yourself:** Write this sentence.
**For the LLM:** This sentence goes at the top of your first prompt.

### 2.2 — Tech Stack Declaration

Declare every technology explicitly. LLMs default to what's statistically most common in their training data — which may not be what you want.

Checklist:
- [ ] **Language and version** — "Python 3.12", not "Python". "TypeScript 5.4", not "TypeScript".
- [ ] **Framework and version** — "Next.js 16 App Router", not "React". "Godot 4.5 GDScript", not "Godot".
- [ ] **Key libraries** — List them. If you have preferences (e.g., "use Zustand not Redux", "use Tailwind not CSS modules"), state them.
- [ ] **Runtime/environment** — "Node 22", "Browser only", "Raspberry Pi 5 ARM64", "Quest 3S".
- [ ] **Database** — If any. Include whether it's local (SQLite) or remote (PostgreSQL on Supabase).
- [ ] **Package manager** — npm, pnpm, yarn, pip, uv, cargo. State it.
- [ ] **Build system** — Vite, Webpack, esbuild, pyproject.toml. State it if non-default.

⚠️ **Warning:** If you leave any of these unspecified, the LLM will pick one silently. You won't notice until you're three prompts deep and the architecture is built around a choice you didn't want.

### 2.3 — Constraints and Non-Negotiables

These are the things the LLM must not violate. They go in every prompt (or in a system prompt/project context if your tool supports it).

- [ ] **Platform constraints** — Mobile-only? Desktop-only? Must work offline? Specific OS?
- [ ] **Performance constraints** — "Must render at 72fps on Quest 3S", "Must handle 10k concurrent connections", "Must process 1M rows in < 30 seconds".
- [ ] **Size constraints** — "Bundle must be under 200KB", "Docker image under 500MB".
- [ ] **Compatibility constraints** — "Must work in Safari 16+", "Must support Python 3.10+".
- [ ] **Legal/licensing constraints** — "No GPL dependencies", "Must use MIT-compatible libraries only".
- [ ] **Security constraints** — "No eval()", "All user input sanitized", "CORS restricted to specific origins".

### 2.4 — What Already Exists

If you're building on top of existing code, the LLM needs to know:

- [ ] **Existing file structure** — Paste your directory tree. Use `tree -L 3 --dirsfirst` or equivalent.
- [ ] **Key existing files** — Paste the content of files the LLM will need to interface with. Don't paste everything — paste the interfaces, types, schemas, and entry points.
- [ ] **Conventions already in use** — Naming conventions, file organization patterns, error handling patterns. If your existing code uses a specific pattern, state it or the LLM will introduce a conflicting one.
- [ ] **State of the codebase** — "This is a fresh project", "This is a working prototype with tech debt", "This is production code, be conservative".

⚠️ **Warning:** "Just look at my code and figure it out" is a prompt that produces generic output. The LLM will pattern-match on superficial structure and miss your actual architectural decisions.

### 2.5 — Define "Done"

How will you know the output is correct? Define this before prompting.

- [ ] **Functional requirements** — What specific behaviors must work?
- [ ] **A test case or walkthrough** — "When I click X, Y should happen, and Z should update." Even informal, this gives the LLM a concrete target.
- [ ] **What "good enough" looks like** — If you're prototyping, say so. "Working prototype, rough edges acceptable, no error handling needed yet." This prevents the LLM from gold-plating.
- [ ] **What failure looks like** — "If the state desyncs, the whole thing is broken." This focuses the LLM on critical correctness paths.

---

## Section 3: Project-Type Specific Checklists

**Read only the subsection that matches your project type from Section 1.**

---

### 3A — Web Apps & SaaS

#### Pre-Production Additions

- [ ] **User flows** — Map every distinct flow the user goes through. "Sign up → verify email → onboarding wizard → dashboard" is a flow. Each flow is a discrete unit the LLM should build and test independently.
- [ ] **Page/route inventory** — List every page/route. For each, note: what data it needs, what actions are possible, what other pages it links to.
- [ ] **State management architecture** — Decide this upfront. Where does state live (URL params, React state, global store, server)? What state is shared between components? What state persists across sessions?
- [ ] **Authentication model** — If any. Provider (Supabase Auth, NextAuth, custom)? Roles/permissions? Protected routes?
- [ ] **API surface** — If you have a backend: list endpoints, methods, request/response shapes. Even rough shapes. Without this, the LLM invents an API surface that won't match your backend.
- [ ] **Responsive behavior** — "Desktop only", "Mobile-first", "Responsive with these breakpoints". If you don't specify, you'll get inconsistent responsive behavior across components.
- [ ] **Design system/tokens** — Colors, typography, spacing, border radius. If you have a Figma file or existing theme, provide the token values directly. If not, at minimum declare: primary color, background color, font family, whether dark/light/both.

⚠️ **Warning:** The #1 failure in web app generation is state management. If you don't specify where state lives and how it flows, the LLM will create a tangled mix of local state, context, and prop drilling that becomes impossible to modify by prompt 3.

#### Aesthetic Specification

- [ ] **Reference sites** — Provide 2-3 URLs of sites whose visual style you want. Be specific: "I want the card layout from X, the color palette from Y, the typography from Z."
- [ ] **Component library** — If using one (shadcn/ui, Radix, MUI), state it. If not, state "custom components, no library."
- [ ] **Animation expectations** — "No animations", "Subtle transitions on navigation", "Heavy motion design with scroll-triggered animations". LLMs either add zero animation or go overboard — there's no middle ground without guidance.
- [ ] **Content density** — "Dense dashboard with lots of data", "Spacious marketing site with large whitespace". This single descriptor changes layout decisions dramatically.

---

### 3B — Games

#### Pre-Production Additions

- [ ] **Engine and version** — Explicit. "Godot 4.5 GDScript", "Unreal Engine 5.7 Blueprints + C++", "browser-based with Phaser 3".
- [ ] **Core game loop** — Describe the 10-second loop. What does the player do repeatedly? "Steer projectile → hit targets → score points → projectile resets." This is the single most important thing to get right; everything else is decoration.
- [ ] **Input scheme** — Keyboard? Gamepad? Touch? Mouse? VR controllers? Gaze-based? List every input and what it does.
- [ ] **Entity inventory** — List every game object: player, enemies, projectiles, pickups, obstacles, UI elements. For each: what properties it has, how it behaves, what it interacts with.
- [ ] **Physics/collision model** — What collides with what? What happens on collision? Is physics simulated or faked? Rigidbody or kinematic?
- [ ] **Game state machine** — Map the states: Main Menu → Playing → Paused → Game Over → Score Screen. What transitions between them? What persists across states?
- [ ] **Difficulty/progression** — How does the game get harder? What parameters change over time?
- [ ] **Score/reward system** — What earns points? What's the feedback loop? What gets saved?

⚠️ **Warning:** LLMs are unreliable at game physics tuning. Provide actual numbers: "gravity = 980", "player speed = 300 pixels/sec", "jump height = 120 pixels". If you say "make the jump feel good", you'll get default engine values that feel like nothing.

#### Asset Specification

- [ ] **Art style** — "Pixel art 16x16", "Low-poly 3D", "Vector flat design", "Hand-drawn sketch". Provide reference images.
- [ ] **Audio expectations** — "No audio", "Placeholder beeps", "Specify sound events but don't implement". This prevents the LLM from ignoring audio entirely or spending tokens on audio code you'll replace.
- [ ] **Resolution/viewport** — "1920x1080 fixed", "Responsive to browser window", "Quest 3S native resolution". Include target framerate.

---

### 3C — Visual & Immersive Experiences

#### Pre-Production Additions

- [ ] **Platform and rendering target** — "Quest 3S standalone (mobile GPU)", "Desktop WebGL", "LED wall 4K output", "Projection mapping". This determines every technical decision downstream.
- [ ] **Interaction model** — "Passive viewing", "Gaze-based interaction", "Hand tracking", "Controller-based", "Touchscreen kiosk". Define every interaction point.
- [ ] **Scene inventory** — List every scene/environment. For each: what's in it, what the user can do, how they transition to the next.
- [ ] **Shader/material requirements** — If you need custom shaders, describe the visual effect in detail. "Translucent material that reveals geometry behind it on gaze" is actionable. "Cool shader" is not.
- [ ] **Performance budget** — Draw call limit, triangle count, texture memory. Immersive experiences on mobile GPUs fail on performance first.
- [ ] **Deployment model** — "Kiosk mode, auto-restart on crash", "WebXR served from local network", "Sideloaded APK". This affects architecture (error recovery, loading, state persistence).

⚠️ **Warning:** LLMs significantly underestimate GPU constraints on mobile/standalone VR. Always specify your hardware target and provide performance budgets in concrete numbers. "Must run at 72fps on Quest 3S" is a constraint that reshapes the entire approach.

#### Visual Reference Requirements

- [ ] **Mood board or reference images** — 3-5 images minimum. Describe what you're taking from each: "The color palette from this image", "The spatial density from this scene", "The material quality from this render."
- [ ] **Lighting model** — "Baked lighting only", "Single directional light + ambient", "Full dynamic lighting". On mobile GPUs, this is a performance-defining decision.
- [ ] **Post-processing** — "None (mobile)", "Bloom + color grading only", "Full post-processing stack". List specific effects.

---

### 3D — CLI Tools & Automation

#### Pre-Production Additions

- [ ] **Interface specification** — List every command, flag, and argument. For each: name, type, default value, description. Example:
  ```
  command: convert
  args: input_file (required, path), output_file (optional, path, default: stdout)
  flags: --format (string, default: "json"), --verbose (bool, default: false)
  ```
- [ ] **Input/output formats** — What goes in, what comes out. JSON, CSV, plain text, binary? If structured, provide a sample of the actual data.
- [ ] **Error behavior** — What happens on bad input? Missing files? Network failures? "Exit with code 1 and a human-readable error message" vs. "Retry 3 times with exponential backoff."
- [ ] **Environment assumptions** — What OS? What's installed? What permissions are needed? "Runs on Ubuntu 22.04 with Docker available" vs. "Must work on Windows, Mac, and Linux with no dependencies beyond Python 3.10+."
- [ ] **Idempotency** — Can it be run twice safely? This matters for automation scripts that might get re-triggered.

⚠️ **Warning:** LLMs consistently forget to handle edge cases in CLI tools: empty input, permission denied, disk full, interrupted mid-write. If your tool writes files, specify what happens on partial failure.

---

### 3E — Data Pipelines & Backend

#### Pre-Production Additions

- [ ] **Data flow diagram** — Even ASCII art. "Source A → Transform X → Store in DB → Expose via API → Consumer Z". Every hop in the pipeline is a potential failure point the LLM needs to know about.
- [ ] **Schema definitions** — Actual schemas. Table definitions, JSON schemas, protobuf definitions. Paste them. Don't describe them in prose.
- [ ] **Volume and throughput** — "100 records per day" and "1M records per hour" require completely different architectures. State your numbers.
- [ ] **Failure and retry semantics** — "At-least-once delivery", "Exactly-once processing", "Best-effort, drop on failure". This changes everything.
- [ ] **Observability requirements** — "Structured JSON logging to stdout", "Prometheus metrics on /metrics", "No observability needed, this is a script."
- [ ] **Authentication and authorization** — How do services authenticate to each other? How do external consumers authenticate?
- [ ] **Deployment target** — "Docker Compose on a single VPS", "Kubernetes", "AWS Lambda", "Runs as a systemd service". This affects how you structure config, secrets, and lifecycle.

⚠️ **Warning:** LLMs produce backend code that works for the happy path. They rarely implement proper transaction handling, connection pooling, graceful shutdown, or retry logic unless explicitly asked. If your pipeline handles money, health data, or anything you can't afford to lose, specify error handling for every stage.

---

### 3F — Python Applications & Scripts

#### Pre-Production Additions

- [ ] **Script vs. application** — Is this a run-once script, a long-running process, a library/package, or a full application with UI? This determines structure.
- [ ] **Dependency management** — pip + requirements.txt, Poetry, uv, conda? If you have existing dependency files, paste them.
- [ ] **Type hinting expectations** — "Full type hints, strict mypy compatible", "Type hints on public interfaces only", "No type hints, this is a quick script."
- [ ] **Entry point and invocation** — "Run as `python main.py`", "Installable CLI via pyproject.toml entry_points", "Imported as a library", "Runs as a FastAPI server."
- [ ] **Data handling** — If processing data: input format, output format, sample data. If using pandas/numpy/polars, state which. LLMs default to pandas even when polars or pure Python would be better.
- [ ] **Concurrency model** — "Single-threaded is fine", "Use asyncio", "Use multiprocessing for CPU-bound work", "Use threading for I/O-bound work". If unspecified, LLMs write synchronous code even when async would be dramatically better.
- [ ] **Configuration** — "Hardcoded constants are fine", "Environment variables", "Config file (TOML/YAML)", "CLI arguments via argparse/click/typer".
- [ ] **Logging** — "Print statements are fine", "Use stdlib logging with level=INFO", "Use structlog with JSON output".
- [ ] **Testing expectations** — "No tests needed", "pytest with basic coverage of core functions", "Full test suite with fixtures and mocks."

⚠️ **Warning:** LLMs produce Python that works but is often non-idiomatic. Common issues: using `os.path` instead of `pathlib`, bare `except:` clauses, not using context managers for file/resource handling, mutable default arguments. If you care about code quality, state "follow modern Python idioms, use pathlib, type hints, context managers."

---

### 3G — Website Design & Static Sites

For portfolios, marketing sites, blogs, agency sites, documentation sites, and any project where **visual design quality is the primary deliverable** — not app functionality.

#### Pre-Production Additions

- [ ] **Content inventory** — List every page and its sections. For each section: what content goes there (headline, body copy, images, testimonials, CTAs), and whether that content is final or placeholder. LLMs generate lorem ipsum layouts that don't survive real content.
- [ ] **Site map & navigation structure** — Page hierarchy, header nav items, footer nav, mobile nav pattern (hamburger, bottom tabs, slide-out). How deep does the hierarchy go? Is there a search?
- [ ] **Visual hierarchy per page** — What's above the fold? What's the primary CTA? What order do sections appear? What draws the eye first? Describe this per page, not just globally.
- [ ] **Content-to-layout ratio** — "Image-heavy portfolio with minimal text", "Long-form text-heavy blog", "Mixed media marketing site with video heroes". This single descriptor changes layout architecture.
- [ ] **CMS / static generation** — Static HTML, Astro, Hugo, Eleventy, WordPress, headless CMS (Sanity, Contentful, Strapi)? If static: how will content be updated? If CMS: who updates it and how technical are they?
- [ ] **SEO requirements** — Meta titles and descriptions per page, Open Graph images, structured data (JSON-LD), XML sitemap, canonical URLs, robots.txt. If you don't specify, the LLM will skip all of it.
- [ ] **Asset pipeline** — Image optimization strategy (WebP/AVIF with fallbacks?), font loading (self-hosted vs Google Fonts vs Adobe Fonts), SVG icon system (inline, sprite, or icon font), favicon set.
- [ ] **Performance budget** — Core Web Vitals targets: LCP under 2.5s, CLS under 0.1, INP under 200ms. Total page weight budget. These constraints prevent the LLM from producing a visually impressive but slow-loading site.

⚠️ **Warning:** LLMs default to app-framework complexity for websites. They'll reach for Next.js with client-side state management when a static site generator or even plain HTML/CSS would be simpler, faster, and more appropriate. If you're building a marketing site, explicitly state: "This is a static site. No client-side JavaScript framework unless absolutely necessary."

#### Aesthetic Specification

- [ ] **Typography system** — Heading scale (h1 through h6 sizes), body text size, font pairings (heading font + body font), line height per context (headings: 1.1-1.3, body: 1.5-1.75), maximum line length (45-75 characters).
- [ ] **Color palette** — Primary, secondary, accent, neutrals (at least 5 shades from near-white to near-black), and semantic colors (success, warning, error, info). Provide exact hex or HSL values. "Blue and white" produces a different site every time you prompt.
- [ ] **Spacing system** — Base unit and scale. Example: 4px base → 4, 8, 12, 16, 24, 32, 48, 64, 96. Every margin, padding, and gap should come from this scale.
- [ ] **Layout grid** — Column count (12-column is standard), gutter width, max content width (e.g., 1200px), breakpoints with specific pixel values.
- [ ] **Image & media treatment** — Aspect ratios for hero images, thumbnails, and cards. Border radius, shadow treatment, hover effects. Whether images are full-bleed, contained, or masked.
- [ ] **Micro-interactions** — Hover states for links, buttons, and cards. Scroll-triggered animations (fade-in, parallax, sticky headers). Page transitions. Specify what moves, how fast (in ms), and what easing function. "Subtle animations" means nothing — "fade-in on scroll, 400ms ease-out, triggered at 20% viewport intersection" is actionable.

⚠️ **Warning:** Without specific typography and spacing values, LLMs produce visually generic sites that look like a Bootstrap template from 2019. The difference between a generic and a polished site is almost entirely in the type scale, spacing rhythm, and color palette — not the layout.

---

## Section 4: UI/UX Design Process

**Read this if your project has any web-based user interface.** This section covers the design system decisions that determine whether your output looks designed or default. Depth varies: for a CLI tool, skip this entirely. For a marketing site or SaaS dashboard, this is critical.

### 4.1 — Typography

- [ ] **Font family selection** — Name the exact fonts. "Inter for body, Fraunces for headings" not "a modern sans-serif". If using Google Fonts, state it. If self-hosting, state it. The loading strategy affects performance.
- [ ] **Type scale** — Pick a ratio and generate your scale. Common ratios: Minor Third (1.200), Major Third (1.250), Perfect Fourth (1.333). Example at 1.250 with 16px base: 16, 20, 25, 31, 39, 49px. Every heading and text size should come from the scale.
- [ ] **Heading weights** — Which weight for each heading level? h1 at 700, h2 at 600, h3 at 500? Or all the same weight with size doing the differentiation?
- [ ] **Body line-height** — 1.5 to 1.75 for body text. 1.1 to 1.3 for headings. State both.
- [ ] **Maximum line length** — 45-75 characters for body text. This means setting `max-width` on text containers, not just hoping the layout handles it.

⚠️ **Warning:** LLMs default to Inter or Roboto for everything. They set font sizes arbitrarily (18px here, 22px there, 15px elsewhere) rather than following a scale. If you don't provide a type scale, you'll get inconsistent typography that looks "off" without being obviously wrong.

### 4.2 — Color System

- [ ] **Define as tokens, not prose** — Provide colors as a structured list of CSS custom properties or design tokens. `--color-primary-500: #2563EB` is unambiguous. "A nice blue" is not.
- [ ] **Full palette with shades** — For each color: at minimum a light, base, and dark variant. For neutrals: a full scale from near-white to near-black (50, 100, 200, ... 900, 950).
- [ ] **Light/dark mode** — If supporting both: define token values for each mode. If only one: state which and don't let the LLM add a theme toggle you didn't ask for.
- [ ] **Contrast ratios** — WCAG AA minimum: 4.5:1 for normal text, 3:1 for large text. Verify your color pairings meet this. LLMs regularly produce low-contrast text, especially for secondary/muted colors.
- [ ] **Semantic mapping** — Success (green), warning (amber/yellow), error (red), info (blue). These appear in forms, alerts, badges, and status indicators. Define the exact hues.

⚠️ **Warning:** LLMs produce low-contrast text and apply colors inconsistently unless given explicit tokens. They'll use your primary color for a button, then a slightly different shade for a link, then another shade for an icon — because they're interpreting "blue" each time rather than referencing a token.

### 4.3 — Spacing & Layout

- [ ] **Base spacing unit** — Pick a base (4px or 8px are standard) and derive all spacing from it. Every margin, padding, and gap should be a multiple of this base.
- [ ] **Spacing scale** — Define the steps: 4, 8, 12, 16, 24, 32, 48, 64, 96, 128. Provide this as a reference. No value should exist outside this scale.
- [ ] **Grid system** — Column count, gutter width, and max content width. Example: "12-column grid, 24px gutters, 1200px max-width, centered."
- [ ] **Section spacing rhythm** — How much vertical space between major page sections? Between components within a section? Define both. Example: "80px between sections, 32px between components within a section."
- [ ] **Component internal padding** — Cards, buttons, inputs, modals — each needs consistent internal spacing. Define it once: "Cards: 24px padding. Buttons: 12px 24px. Inputs: 12px 16px."

⚠️ **Warning:** LLMs mix spacing values randomly — 12px here, 15px there, 20px elsewhere, 18px somewhere else. The result looks subtly wrong but is hard to diagnose. A consistent spacing scale is the single highest-leverage design decision you can make.

### 4.4 — Responsive Strategy

- [ ] **Breakpoints** — Define exact pixel values: mobile (< 640px), tablet (640-1024px), desktop (> 1024px). Or whatever your project needs. Don't rely on the LLM's defaults.
- [ ] **What changes at each breakpoint** — For each breakpoint: what layout shifts occur? Does the nav collapse to a hamburger? Do grid columns reduce? Do font sizes decrease? Specify per breakpoint.
- [ ] **Mobile-first vs desktop-first** — State your approach. "Write mobile styles first, add complexity at larger breakpoints" or "Design for desktop, adapt down." This changes how media queries are written.
- [ ] **Touch target sizes** — Minimum 44x44px for interactive elements on touch devices. If you don't specify, LLMs produce tiny links and buttons that are frustrating on mobile.

⚠️ **Warning:** LLMs generate desktop layouts first and bolt on media queries as an afterthought. The mobile version is always an afterthought — cramped layouts, overflowing text, hidden content. If mobile matters, state "mobile-first" and describe the mobile layout before the desktop one.

### 4.5 — Visual Consistency Checklist

- [ ] **Border radius** — Pick one value (or a small set: small 4px, medium 8px, large 16px, full/pill) and use it everywhere. LLMs mix 4px, 6px, 8px, 12px randomly.
- [ ] **Shadow system** — Define elevation levels: subtle (cards at rest), medium (dropdowns, popovers), heavy (modals). Provide the actual `box-shadow` values.
- [ ] **Icon style** — Outline or filled? Consistent stroke width? What size (16px, 20px, 24px)? Which icon set (Lucide, Heroicons, Phosphor)? Mixing icon styles is immediately noticeable.
- [ ] **Button hierarchy** — Primary (filled), secondary (outlined), ghost/text. What colors, padding, and border radius for each? What are the hover, active, disabled, and loading states?
- [ ] **Form element styling** — Inputs, selects, checkboxes, radio buttons, textareas. These must look like they belong to the same design system. Define border color, focus ring color, error state, and placeholder color.

⚠️ **Warning:** LLMs produce components that look individually acceptable but inconsistent as a system. A card with 8px radius next to a button with 4px radius next to an input with 6px radius. Each is fine alone; together they look undesigned.

### 4.6 — Accessibility Baseline

- [ ] **Color contrast** — All text meets WCAG AA (4.5:1 normal, 3:1 large). Test your palette pairings before prompting, not after.
- [ ] **Focus indicators** — Visible focus rings on all interactive elements. Specify the style: "2px solid blue outline with 2px offset" not "default browser focus."
- [ ] **Alt text strategy** — Decorative images get `alt=""`. Informative images get descriptive alt text. State this so the LLM doesn't leave `alt` attributes empty or fill them with filenames.
- [ ] **Heading hierarchy** — One `h1` per page. Headings don't skip levels (no h2 → h4). This is semantic structure, not just visual styling.
- [ ] **Landmark regions** — `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`. Every page should use these. LLMs often produce `<div>` soup instead.
- [ ] **Keyboard navigation** — All interactive elements reachable via Tab. Logical tab order. Escape closes modals. Enter/Space activates buttons. Test this explicitly.

⚠️ **Warning:** LLMs skip accessibility unless you explicitly require it. They omit focus styles, produce empty `alt` attributes, use `<div>` with `onClick` instead of `<button>`, and break heading hierarchy for visual reasons. If accessibility matters, state it as a constraint in every prompt, not just the first one.

---

## Section 5: Scope Estimation & Prompt Staging

**Read this if your project is Medium or Complex.**

### 5.1 — Estimating Prompt Count

Use this formula as a rough guide:

```
Distinct features or modules × 1.5 = approximate prompt count
```

The 1.5 multiplier accounts for: integration between modules, bug fixes, and refinement passes.

If your number exceeds 15, you almost certainly need a PRD (Section 6) and should consider whether this is a project better built with an IDE + LLM-assist rather than pure vibe coding.

### 5.2 — Decomposition Strategy

Break your project into prompts using these principles:

**Each prompt should produce a unit that can be tested independently.** If you can't verify the output of a prompt in isolation, the scope is wrong.

**Dependencies flow forward, never backward.** Prompt 3 can depend on the output of Prompt 2, but if Prompt 3 requires changing what Prompt 2 produced, your decomposition is wrong.

**The first prompt is always the skeleton.** It should produce:
- Project structure (files, folders)
- Core data types / schemas / interfaces
- Entry point that runs (even if it does nothing useful)
- The single most critical feature — the one everything else hangs off

**Subsequent prompts add features one at a time.** Each prompt:
1. References what already exists (paste the current state or relevant files)
2. Adds exactly one feature or module
3. Specifies how it integrates with existing code
4. Defines how to verify it works

### 5.3 — Staging Template

For each prompt in your sequence, fill this out before prompting:

```
Prompt N: [name]
  Depends on: [which previous prompts]
  Adds: [what new capability]
  Modifies: [which existing files, if any]
  Verify by: [how you'll test this works]
  Risk: [what's most likely to go wrong]
```

This takes 5 minutes and saves hours of debugging tangled prompt chains.

### 5.4 — When to Restart vs. Continue

**Restart the session/conversation if:**
- The LLM's architecture is fundamentally wrong (wrong state management pattern, wrong data model)
- You're on prompt 5+ and every prompt requires fixing the previous prompt's output
- The LLM starts "forgetting" earlier context and introducing contradictions

**Continue the session if:**
- Issues are localized (a single function is wrong, a style is off, an edge case is missing)
- The architecture is sound but details need refinement
- You're adding new features to a working base

---

## Section 6: PRD Checklist

**Read this if your project is Complex (Section 1.2).**

A PRD for vibe coding is not the same as a corporate PRD. It's a structured context document that gives the LLM (and you) a complete picture of what's being built. It lives alongside your prompt — you paste relevant sections into each prompt.

### 6.1 — PRD Structure

Build your PRD with these sections. Each section has a specific purpose for the LLM.

#### 6.1.1 — One-Paragraph Summary
A dense paragraph covering: what it is, who it's for, what problem it solves, and what "done" looks like. This goes at the top of every prompt.

#### 6.1.2 — User Personas (if applicable)
Only include if your app has meaningfully different user types. For each persona: what they do, what they need, what they don't care about. Keep each to 2-3 sentences. Skip this for tools, scripts, and single-user projects.

#### 6.1.3 — Feature Inventory
A flat list of every feature. For each:
- **Name** — short label
- **Priority** — Must-have, Should-have, Nice-to-have
- **Complexity** — Simple, Medium, Complex
- **Dependencies** — which other features must exist first
- **Acceptance criteria** — 1-3 concrete statements of what "working" means

This list becomes your prompt decomposition plan (Section 5).

#### 6.1.4 — Data Model
Every entity, its fields, its relationships. Use actual schema notation your tech stack uses (Prisma schema, SQLAlchemy models, TypeScript interfaces, GDScript class definitions). Not prose. Not ER diagrams described in words.

Paste this into any prompt that touches data.

#### 6.1.5 — Architecture Overview
A brief description of how the system is structured:
- What runs where (client, server, edge, device)
- How components communicate (REST, WebSocket, signals, direct function calls)
- What state lives where

Include a simple ASCII diagram if the architecture has more than 2 components.

#### 6.1.6 — API Contracts (if applicable)
For every endpoint or interface between components:
- Method and path (or signal name, or function signature)
- Request shape with types
- Response shape with types
- Error cases

#### 6.1.7 — Non-Functional Requirements
Performance, security, accessibility, scalability requirements that apply globally. These go in every prompt as constraints.

#### 6.1.8 — Open Questions
Things you haven't decided yet. Write them down. When prompting, either resolve them before the relevant prompt or explicitly tell the LLM to make a decision and document it: "I haven't decided X yet. Pick the simpler option and note what you chose."

### 6.2 — PRD Anti-Patterns

- **Too long** — If your PRD is over 3000 words, the LLM won't attend to all of it equally. Front-load the most important information.
- **Prose-heavy** — Schemas, contracts, and inventories should be structured data, not paragraphs. The LLM parses structured formats more reliably.
- **Aspirational features** — Don't include features you "might" want. Every feature in the PRD will influence architecture decisions. Include only what you're building now.
- **No priorities** — Without priority labels, the LLM treats everything as equally important and may gold-plate a Nice-to-have while half-implementing a Must-have.

---

## Section 7: Reference Material Guide

**Read this for every project.** How you provide references determines whether the LLM builds what you imagined or what it hallucinated.

### 7.1 — Types of References and How to Provide Them

#### Visual/Aesthetic References
- **Best:** Screenshot or image file provided directly to the LLM (if it supports vision). Annotate the screenshot: "I want the layout from this, not the colors."
- **Good:** URL of a live site + specific description of what to take from it. "Look at stripe.com/docs — I want the same sidebar navigation pattern, the same code block styling, but with my color palette."
- **Acceptable:** Detailed verbal description with named reference points. "Card-based grid layout like Notion's gallery view. Cards have a thumbnail on top, title below, 2 lines of metadata, subtle drop shadow on hover."
- **Bad:** "Make it look modern and clean." This produces generic output.

#### Code/Architecture References
- **Best:** Paste the actual code you want to match or extend. If it's an API you're integrating with, paste the docs or response shape.
- **Good:** Link to a specific GitHub file or documentation page + what to take from it.
- **Acceptable:** Name a well-known pattern: "Use the repository pattern for data access", "Structure it like a Next.js App Router project."
- **Bad:** "Use best practices." The LLM's "best practices" are an average of everything in its training data.

#### Behavioral References
- **Best:** Provide a concrete walkthrough: "User clicks the card → card expands to full screen with a 300ms spring animation → content fades in after expansion completes → clicking outside collapses it."
- **Good:** Name a reference interaction: "Expand behavior should feel like iOS App Store's Today tab card expansion."
- **Acceptable:** Describe the feel: "Snappy, no perceptible lag, transitions under 200ms, no layout shift."
- **Bad:** "Make it feel smooth." This means nothing concrete.

### 7.2 — Reference Quantity

- **Simple project:** 1-2 references for overall direction are enough.
- **Medium project:** 2-3 references for aesthetic + 1-2 for architecture/patterns.
- **Complex project:** Compile a reference document with categorized references (visual, behavioral, architectural). Paste relevant subsets into each prompt.

### 7.3 — Reference Conflicts

If your references conflict (e.g., site A's layout but site B's spacing, and they're incompatible), you must resolve the conflict before prompting. The LLM will not ask you to resolve it — it'll silently pick one or produce a broken hybrid.

---

## Section 8: Prompting Session Tactics

**Your in-session playbook.**

### 8.1 — First Prompt Structure

Your first prompt should follow this structure. Reorder sections by importance for your project, but include all of them.

```
[1-2 sentence summary of what you're building]

Tech stack: [from Section 2.2]

Constraints: [from Section 2.3]

What exists already: [from Section 2.4, or "Fresh project, no existing code"]

[Your project-type-specific details from Section 3]

[Design tokens from Section 4, if applicable]

[References from Section 7]

Build: [The specific deliverable for this prompt — the skeleton or first feature]

When done, I should be able to: [verification step from Section 2.5]
```

### 8.2 — Follow-Up Prompt Structure

After the first prompt, every subsequent prompt should:

```
[Paste or reference the current state of the project — file tree, key files, or 
"the code from your previous response"]

Current status: [what works, what doesn't]

Next: [exactly one feature or change]

This should: [concrete behavior description]

Don't change: [list anything that's currently working that you don't want touched]

[Any new references relevant to this specific addition]
```

The "Don't change" line is critical. Without it, LLMs refactor working code to accommodate new features, introducing regressions.

### 8.3 — Context Management

LLMs have finite context windows. As your project grows across prompts:

**What to always include:**
- Project summary (1-2 sentences, never omit)
- Tech stack and constraints (paste verbatim every time)
- File tree of current state
- The specific files being modified (paste full content)

**What to include selectively:**
- Files adjacent to what's being modified (interfaces, types, shared utilities)
- Relevant sections of the PRD

**What to omit:**
- Files unrelated to the current prompt
- Previous conversation history (the LLM doesn't need to re-read your discussion about the previous feature)
- Completed, stable modules that aren't being touched

### 8.4 — Handling LLM Ambiguity

If the LLM asks a clarifying question, that's fine — answer it. But if the LLM makes a choice without asking and you don't like it:

**Don't say:** "That's wrong, fix it."
**Say:** "Change X to Y. Specifically: [describe the exact change with concrete details]. Keep everything else the same."

Vague corrections produce vague fixes. The LLM will change something, but not necessarily the right thing.

### 8.5 — Token Efficiency

Ways to reduce token waste and improve output quality:

- **Front-load critical constraints.** The LLM attends most strongly to the beginning and end of your prompt. Put non-negotiable constraints at the top.
- **Use structured formats for structured information.** Paste schemas as code, not as prose descriptions of schemas.
- **Don't repeat yourself across sections.** If your tech stack is declared at the top, don't mention it again inline.
- **Ask for specific outputs.** "Output only the modified files" is better than letting the LLM decide what to include.
- **Specify what you don't want.** "No comments in the code", "Don't include a README", "Don't add error handling yet — I'll request that separately."

### 8.6 — Testing Between Prompts

After every prompt's output, before the next prompt:

1. **Run it.** Does it start? Does it crash?
2. **Test the new feature.** Does the specific thing this prompt was supposed to add actually work?
3. **Regression check.** Do the previously working features still work?
4. **Note exact issues.** Not "it's broken" — write down what you clicked, what happened, and what should have happened instead.

Feed these exact observations into the next prompt. "When I click Submit with an empty name field, the app crashes with `TypeError: Cannot read property 'length' of undefined` at line 42 of Form.jsx" gives the LLM everything it needs. "The form doesn't work" gives it nothing.

---

## Section 9: Course Correction

**Bookmark this section. Read it when things go wrong.**

### 9.1 — Diagnosis

When the output isn't what you wanted, identify which category the problem falls into:

| Problem | Symptom | Fix Strategy |
|---|---|---|
| **Wrong architecture** | The fundamental structure is wrong. State management is a mess, data flows incorrectly, components are split wrong. | Restart (9.2) |
| **Wrong feature behavior** | The architecture is fine but a feature doesn't behave correctly. | Targeted correction (9.3) |
| **Wrong aesthetics** | Functionality works but it looks/feels wrong. | Aesthetic patch (9.4) |
| **Accumulating bugs** | Each fix introduces new bugs, nothing is stable. | Stabilization pass (9.5) |
| **LLM confusion** | The LLM contradicts itself, forgets context, or produces nonsensical output. | Context reset (9.6) |

### 9.2 — Restart Protocol

When architecture is fundamentally wrong:

1. **Do not try to fix it incrementally.** An LLM working within a broken architecture will produce increasingly broken code trying to work around foundational issues.
2. **Identify what went wrong.** Usually: ambiguous state management spec, missing data model, or underspecified component boundaries.
3. **Rewrite your first prompt** with the missing specification added.
4. **Start a new session/conversation.** The old conversation's context is contaminated with bad patterns the LLM will repeat.
5. **Salvage what worked.** If individual components from the failed attempt are good, paste them into the new session as "existing code to integrate."

### 9.3 — Targeted Correction

When a specific feature is wrong:

```
The [feature name] doesn't work correctly.

Current behavior: [exactly what happens]
Expected behavior: [exactly what should happen]
Steps to reproduce: [click/action sequence]

The relevant code is in [file name]:
[paste the relevant section]

Fix only this behavior. Don't restructure or refactor anything else.
```

Key phrases that prevent LLM over-correction:
- "Fix only this behavior."
- "Don't restructure or refactor anything else."
- "Keep the existing function signatures."
- "Don't change any file except [X]."

### 9.4 — Aesthetic Patch

When it works but looks wrong:

```
The functionality is correct — don't change any behavior or logic.

Visual issues:
1. [Specific issue: "The card padding is too tight, increase to 24px"]
2. [Specific issue: "The header font should be 600 weight, currently looks like 400"]
3. [Specific issue: "Add 8px gap between the icon and label"]

Only modify styles/CSS/classes. Don't touch component structure or logic.
```

Be pixel-specific. "Too much padding" is ambiguous. "Reduce padding from 32px to 16px" is concrete.

### 9.5 — Stabilization Pass

When bugs are accumulating:

1. **Stop adding features.**
2. **List every known bug** with reproduction steps.
3. **Prompt the LLM to fix bugs only:**

```
Do not add any new features or make any improvements.

Fix these bugs only:
1. [Bug with reproduction steps]
2. [Bug with reproduction steps]
3. [Bug with reproduction steps]

After each fix, explain what you changed and why. If fixing one bug would 
require changing the approach to another, flag it instead of doing it.
```

4. **Verify all bugs are fixed before resuming feature work.**

### 9.6 — Context Reset

When the LLM is confused, contradicting itself, or producing nonsensical output:

1. **Start a new conversation/session.**
2. **Paste the current state of your project** (file tree + key files), not the conversation history.
3. **Describe the project fresh** as if the LLM has never seen it.
4. **State what's working and what the next task is.**

This is often necessary after 10-15 exchanges in a single conversation. LLM attention degrades over long conversations.

---

## Section 10: Anti-Patterns

**Things that reliably break LLM code generation. Skim before your first prompt, reference when debugging failures.**

### 10.1 — Prompting Anti-Patterns

| Anti-Pattern | Why It Fails | What To Do Instead |
|---|---|---|
| "Build me a full app" in one prompt | LLM produces shallow implementation of everything, deep implementation of nothing. Output is too large to verify. | Decompose per Section 5. One feature per prompt. |
| Describing behavior in prose instead of steps | "The dashboard should be interactive" gives zero actionable information. | "Clicking a chart bar filters the table below to show only rows from that category." |
| Not specifying error handling | LLM ignores errors entirely or adds generic try/catch that swallows everything. | Specify per-operation: "If the API call fails, show a toast with the error message and keep the form data intact." |
| "Make it better" / "Improve this" | The LLM doesn't know your quality bar. It will make arbitrary changes. | Specify exactly what to change: "Increase contrast ratio of body text to 7:1. Add loading skeleton to the card grid. Fix the jittery scroll on the sidebar." |
| Pasting entire codebase | Floods the context window. LLM attends weakly to most of it and may produce changes to files you didn't want touched. | Paste only files being modified + their interfaces. |
| Asking for multiple unrelated changes | LLM gets confused about scope, makes partial changes to each. | One change per prompt. Batch only if changes are to the same file and closely related. |
| "Use best practices" / "Make it production-ready" | Means something different in every context. LLM picks an arbitrary subset. | Name specific practices: "Add input validation with zod schemas", "Use connection pooling with a pool size of 10", "Add structured logging with correlation IDs." |
| Not declaring what exists | LLM creates duplicate utilities, conflicting type definitions, or reimplements what's already there. | Always paste file tree + relevant interfaces at the top of each prompt. |

### 10.2 — Code Generation Anti-Patterns (What LLMs Do Wrong)

These are patterns LLMs reliably produce that you should watch for and correct:

- **Silent state mutations.** LLMs create functions that modify objects in place without making it obvious. This causes bugs that are hard to trace. Watch for objects and arrays being modified instead of copied.
- **Implicit type coercion.** Particularly in JavaScript/TypeScript. LLMs write loose comparisons, concatenate strings and numbers, or pass wrong types through function boundaries.
- **Orphaned event listeners and subscriptions.** LLMs add event listeners in setup/mount but forget cleanup in teardown/unmount. This causes memory leaks and ghost behavior.
- **Hardcoded magic values.** LLMs embed constants directly in logic instead of extracting named constants. This makes tuning impossible.
- **Shallow error handling.** `catch(e) { console.log(e) }` everywhere. The error is logged but nothing recovers, retries, or informs the user.
- **God components/functions.** LLMs write monolithic components that handle state, rendering, and side effects all in one. This makes targeted modification in later prompts nearly impossible.
- **Incorrect async patterns.** LLMs produce race conditions, unhandled promise rejections, and missing `await` keywords. If your project uses async code, verify every async boundary.
- **Overengineering on simple projects.** For a small script, the LLM may generate a full class hierarchy with abstract base classes when a flat function would suffice. Match complexity to project scope in your prompt.
- **Dependency sprawl.** LLMs add dependencies you didn't ask for. Review every import. If you see a library you didn't specify, question whether it's necessary.

### 10.3 — Communication Anti-Patterns

- **Not reading the output.** Skipping ahead to the next prompt without verifying the current output means errors compound. Always run and test before prompting again.
- **Sunk cost continuation.** Continuing to patch a broken architecture because you've already invested 8 prompts. See Section 9.2 — restart is often faster.
- **Assuming the LLM remembers.** Even within a single conversation, after 10+ exchanges, the LLM's effective attention on early context is degraded. Re-state critical constraints.
- **Anthropomorphizing errors.** "The LLM doesn't understand my vision" — it never did. It's pattern-matching on your words. If the output is wrong, your words were ambiguous. Revise the specification, not your expectations of the LLM.

---

## Quick Reference Card

Cut this out and keep it next to your prompt window.

```
BEFORE FIRST PROMPT:
□ One-sentence "done" statement
□ Tech stack with versions
□ Constraints and non-negotiables
□ What already exists (or "fresh project")
□ Project-type-specific prep (Section 3)
□ Design tokens defined if web UI (Section 4):
  □ Typography: fonts, type scale, line heights
  □ Colors: full palette as hex/HSL tokens
  □ Spacing: base unit + scale
  □ Layout: grid, breakpoints, max-width
  □ Consistency: border radius, shadows, buttons
  □ Accessibility: contrast ratios, focus styles
□ References gathered (visual, code, behavioral)
□ Complexity assessed, prompts staged if Medium/Complex
□ PRD written if Complex

EVERY PROMPT:
□ Project summary (1-2 sentences)
□ Tech stack + constraints
□ Design tokens (if web UI)
□ Current file tree
□ Files being modified (full content)
□ Exactly one task
□ "Don't change" list
□ Verification step

AFTER EVERY PROMPT:
□ Run it
□ Test new feature
□ Regression check
□ Note exact issues for next prompt

WHEN THINGS GO WRONG:
□ Identify: architecture, behavior, aesthetic, or accumulation?
□ Architecture wrong → restart with better spec
□ Behavior wrong → targeted fix with reproduction steps  
□ Aesthetic wrong → visual-only patch with pixel values
□ Bugs accumulating → freeze features, fix all bugs first
□ LLM confused → new session, paste current state fresh
```
