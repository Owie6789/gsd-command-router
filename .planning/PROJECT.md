# GSD Command Router

**Version:** 1.0.0  
**Analysis Date:** 2026-04-19  
**Status:** Active Development

---

## Core Value

The GSD Command Router is a **single-file interactive documentation site** that helps users discover, filter, and understand 50+ GSD (Get Shit Done) CLI commands across 8 categories.

It transforms a static command reference into an **explorable, decision-guided experience** where users can:
- Search and filter commands by category
- Use an interactive wizard to find the right command for their need
- View compact cheatsheets or detailed command documentation
- Understand the core GSD workflow visually

---

## Problem Statement

GSD has 50+ commands across 8 categories (Core, Workstream, Spike, UI, Phase, Session, Quality, Util). New users face:
- **Discovery friction** — hard to know which command fits their need
- **Context switching** — jumping between README, examples, and actual usage
- **Memory burden** — too many commands to memorize

The Command Router solves this by providing a **single, searchable, decision-guided interface** that reduces time-to-first-command.

---

## Current State

**Architecture:** Single-File Application (SFA)
- **File:** `gsd.html` (1930 lines)
- **Technology:** Vanilla JavaScript, CSS custom properties, semantic HTML5
- **Dependencies:** None (zero npm packages, zero build step)
- **External:** Fontshare CDN for Satoshi font
- **Deployment:** Static file hosting (GitHub Pages, Netlify, Vercel, etc.)

**Features Implemented:**
- ✅ Command list with search and category filtering
- ✅ Expandable command details (flags, usage, when, related)
- ✅ Interactive wizard (8-step decision tree)
- ✅ Core workflow visualization (3 stages, 19 steps)
- ✅ Cheatsheet view (compact reference table)
- ✅ Concepts view (8 key GSD patterns)
- ✅ Keyboard shortcuts (`/` to focus search, `Escape` to clear)
- ✅ Copy-to-clipboard with toast feedback
- ✅ Responsive design (mobile breakpoints at 860px, 700px)

**Technical Debt:**
- ❌ No automated tests (zero coverage)
- ❌ No linting or formatting enforcement
- ❌ No CI/CD pipeline
- ❌ Hardcoded command data (manual updates)
- ❌ No state persistence (resets on reload)
- ❌ Full re-render on every search keystroke (performance)
- ❌ XSS vulnerability risk (minimal escaping)
- ❌ No Content Security Policy

---

## Project Goals

### V1 Goals (Current Milestone)

**Enhance usability and code quality without breaking the single-file simplicity:**

1. **Search & Filter Improvements**
   - Add runtime filter (filter commands by AI tool: Claude, GPT, Gemini, etc.)
   - Add multi-select filtering (combine category + runtime)
   - Improve search relevance (fuzzy matching, command name priority)

2. **Keyboard Navigation**
   - Arrow keys to navigate command list
   - `Enter` to expand/collapse command
   - `Escape` to close expanded command or clear search
   - Tab navigation through interactive elements

3. **State Persistence**
   - Remember last active view in `localStorage`
   - Persist search query and active filters across reloads
   - Restore wizard state if user navigates away mid-flow

4. **Code Quality**
   - Extract JavaScript to external `app.js` file
   - Extract CSS to external `styles.css` file
   - Add ESLint + Prettier configuration
   - Add basic unit tests for utility functions

### V2 Goals (Future Milestone)

**Advanced features for power users:**

1. **Command History & Favorites**
   - Track recently viewed commands (localStorage)
   - Mark favorite commands for quick access
   - Export/import favorites as JSON

2. **Analytics & Insights**
   - Anonymous search analytics (opt-in)
   - Most-viewed commands dashboard
   - Search terms with no results (identify gaps)

3. **Export & Sharing**
   - Export filtered commands as Markdown or PDF
   - Generate shareable URLs with query params (`?filter=core&search=phase`)
   - Print-optimized stylesheet

4. **Theme System**
   - Light/dark theme toggle
   - Persist theme preference
   - High-contrast mode for accessibility

### V3 Goals (Future Milestone)

**Integration and extensibility:**

1. **Live Command Data**
   - Fetch command data from upstream GSD repo at build time
   - Cache with fallback to embedded data
   - Show "last updated" timestamp

2. **Command Comparison**
   - Select multiple commands
   - Side-by-side comparison view
   - Highlight differences in flags, usage, runtimes

3. **Interactive Tutorials**
   - Guided walkthroughs for common workflows
   - "Try it now" sandbox (if AI API integration added)
   - Progress tracking

---

## Constraints

### Technical Constraints

- **Single-file portability:** Must remain runnable as `file://` without a server
- **Zero build step:** No bundlers, transpilers, or npm install required
- **No backend:** Purely client-side, no API server, no database
- **Browser-only:** Modern browser with ES6+ support (Chrome 61+, Firefox 54+, Safari 11+)

### Design Constraints

- **Dark theme only** (for now) — light theme is a V2 feature
- **Satoshi font** via Fontshare CDN (or system font fallback)
- **Mobile-responsive** — must work on tablets and phones

### Process Constraints

- **Solo developer workflow** — no team coordination, no sprints
- **Agent-assisted implementation** — user is visionary, agent is builder
- **GSD methodology** — goal-backward planning, observable success criteria

---

## Success Metrics

### Usability Metrics

| Metric | Current | Target V1 | Target V2 |
|--------|---------|-----------|-----------|
| Time to find command (avg) | ~15s | <10s | <5s |
| Search success rate | ~70% | >85% | >95% |
| Wizard completion rate | Unknown | >60% | >75% |
| Return visitor rate | Unknown | >40% | >60% |

### Code Quality Metrics

| Metric | Current | Target V1 | Target V2 |
|--------|---------|-----------|-----------|
| Test coverage | 0% | 40% | 80% |
| Lint errors | N/A | 0 | 0 |
| Performance (first paint) | ~500ms | <300ms | <200ms |
| Lighthouse score | Unknown | >90 | >95 |

---

## Key Decisions

### Architecture Decisions

1. **Keep single-file option:** Even after splitting into `.html`/`.css`/`.js`, provide a build script that recombines into a single portable file.

2. **No framework adoption:** Vanilla JS is sufficient. Adding React/Vue/Svelte would increase complexity without proportional benefit.

3. **localStorage over cookies:** No backend means no sessions. Use `localStorage` for persistence.

4. **No analytics by default:** Privacy-first. Analytics must be opt-in and anonymous.

### Technology Decisions

1. **Vitest for testing:** Lightweight, fast, built-in coverage. Compatible with potential future Vite adoption.

2. **ESLint + Prettier:** Standard combo for code quality. Minimal config overhead.

3. **GitHub Pages for hosting:** Free, simple, integrates with existing GitHub repo.

4. **No TypeScript (yet):** JSDoc annotations can provide type hints without build step. Revisit in V3.

---

## Risk Assessment

### High Risk

- **Data drift:** Hardcoded `CMDS` array can become outdated from upstream GSD repo.
  - *Mitigation:* Add build-time fetch script, show "last updated" date.

- **XSS vulnerability:** Minimal HTML escaping could allow injection if data source is compromised.
  - *Mitigation:* Use proper escaping library, add CSP meta tag.

### Medium Risk

- **Font CDN dependency:** Fontshare CDN is a single point of failure.
  - *Mitigation:* Self-host font files, add system font fallback.

- **Performance degradation:** Full re-render on every search keystroke will lag with larger command sets.
  - *Mitigation:* Add debouncing, consider virtual scrolling.

### Low Risk

- **Browser compatibility:** ES6+ features are widely supported.
  - *Mitigation:* Document minimum browser versions, use caniuse.com checks.

---

## Related Projects

- **GSD Core:** `https://github.com/gsd-build/get-shit-done` — The actual CLI tool
- **Owivara Bot:** WhatsApp bot with GSD plugin integration (separate codebase)

---

## Glossary

| Term | Definition |
|------|------------|
| **GSD** | Get Shit Done — AI-assisted software development workflow |
| **Command Router** | This project — interactive documentation site |
| **Runtime** | AI coding tool (Claude Code, GPT Engineer, Gemini, etc.) |
| **Wizard** | Interactive decision tree for command discovery |
| **View** | Full-screen UI section (Commands, Wizard, Flow, Cheatsheet, Concepts) |
| **SFA** | Single-File Application — all code in one `.html` file |

---

*PROJECT.md created: 2026-04-19*
