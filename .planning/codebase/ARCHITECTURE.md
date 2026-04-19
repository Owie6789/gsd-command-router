# Architecture

**Analysis Date:** 2026-04-19

## Pattern Overview

**Overall:** Single-File Application (SFA) / Single Page Application (SPA)

**Key Characteristics:**
- Zero-build architecture — no bundler, transpiler, or build step
- All code (HTML, CSS, JavaScript) in a single `gsd.html` file (1762 lines)
- Client-side rendering with vanilla JavaScript
- CSS custom properties for theming
- Component-based UI with view switching

## Layers

**Presentation Layer:**
- Purpose: Visual styling and responsive layout
- Location: `<style>` block in `gsd.html` (lines 10-234)
- Contains: CSS custom properties, component styles, media queries
- Depends on: None (self-contained)
- Used by: All DOM elements

**Structure Layer:**
- Purpose: Semantic HTML structure and view containers
- Location: `<body>` in `gsd.html` (lines 236-354)
- Contains: Top bar, sidebar navigation, main content area, view sections
- Depends on: CSS classes from presentation layer
- Used by: JavaScript rendering functions

**Logic Layer:**
- Purpose: Application state, routing, and rendering
- Location: `<script>` block in `gsd.html` (lines 356-1760)
- Contains: Data models, state management, view functions, event handlers
- Depends on: Browser DOM API, Clipboard API
- Used by: User interactions

## Data Flow

**View Navigation Flow:**

1. User clicks sidebar item or calls `switchView(viewId)`
2. `switchView()` removes `.active` class from all views
3. Target view receives `.active` class (display: block)
4. View-specific render function is invoked (e.g., `renderWizard()`)
5. DOM is updated with rendered content

**Command Filtering Flow:**

1. User types in search input or clicks filter chip
2. `searchVal` or `activeFilter` state is updated
3. `renderCommands()` calls `getFiltered()`
4. Filtered command list is rendered to `#cmd-list`
5. Empty state shown if no matches

**Wizard Decision Flow:**

1. User selects option in `renderWizardStep(node)`
2. Option's `next` index loads next question from `WIZARD_TREE`
3. History is tracked in `wizardHistory` array
4. Final option with `result` property displays recommended command
5. User can copy usage or navigate to full command detail

**State Management:**
- Global state variables: `activeView`, `activeFilter`, `searchVal`, `openCmd`
- Wizard state: `wizardHistory` (array), `wizardStep` (number)
- No persistence — state resets on page reload
- No external state management library

## Key Abstractions

**Command Object:**
- Purpose: Represents a single GSD slash command
- Examples: All entries in `CMDS` array (lines 365-1120)
- Pattern: `{ id, cmd, cat, short, desc, flags, usage, when, related, runtimes }`

**Category Metadata:**
- Purpose: Maps category codes to display labels and badge styles
- Examples: `CAT` object (lines 1125-1134)
- Pattern: `{ core, workstream, spike, ui, phase, session, quality, util }`

**View Component:**
- Purpose: Encapsulates a full-screen view with its own rendering logic
- Examples: `#view-commands`, `#view-wizard`, `#view-flow`, `#view-cheatsheet`, `#view-concepts`
- Pattern: `<section class="view" id="view-{name}">` with corresponding `render{Name}()` function

**Wizard Tree:**
- Purpose: Decision tree for command recommendation
- Examples: `WIZARD_TREE` array (lines 1281-1364)
- Pattern: `{ q, sub, opts: [{ label, sub, next? , result? }] }`

## Entry Points

**Initial Load:**
- Location: `gsd.html` line 1760 — `renderCommands()`
- Triggers: Page load / DOM ready
- Responsibilities: Render initial command list view

**View Switching:**
- Location: `switchView(viewId)` function (line 1189)
- Triggers: Sidebar navigation clicks
- Responsibilities: Hide all views, show target view, invoke view-specific renderer

**Command Detail Toggle:**
- Location: `.cmd-row` click handler (line 1267)
- Triggers: User clicks command row
- Responsibilities: Toggle `openCmd` state, re-render command list with expanded detail

## Error Handling

**Strategy:** Minimal — relies on browser's native error handling

**Patterns:**
- Optional chaining for safe property access (e.g., `CAT[c.cat] || CAT.util`)
- Empty state rendering when filters return no results
- Try-catch on clipboard operations via Promise `.then()`

## Cross-Cutting Concerns

**Logging:** None — no console logging in production code

**Validation:** Input sanitization via `escHtml()` function (line 1282) for XSS prevention

**Authentication:** Not applicable — static file with no backend

**Clipboard Integration:** `copyText()` utility (line 1169) uses Clipboard API with toast feedback

**Keyboard Shortcuts:** 
- `/` focuses search input (line 1336)
- `Escape` clears search (line 1340)
- `Enter` / `Space` activates focused command row (line 1274)

---

*Architecture analysis: 2026-04-19*
