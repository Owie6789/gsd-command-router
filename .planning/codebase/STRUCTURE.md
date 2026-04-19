# Codebase Structure

**Analysis Date:** 2026-04-19

## Directory Layout

```
Gsd/
├── gsd.html                          # Single-file web application (1762 lines)
├── gsd-logo-2000-transparent.png     # Brand logo asset
├── .planning/
│   └── codebase/
│       ├── ARCHITECTURE.md           # Architecture documentation
│       ├── STRUCTURE.md              # This file
│       ├── CONVENTIONS.md            # Coding conventions (pre-existing)
│       └── TESTING.md                # Testing patterns (pre-existing)
└── get-shit-done/
    └── .git/                         # Git repository (GSD core project)
```

## Directory Purposes

**Root (`Gsd/`):**
- Purpose: Container for the GSD documentation site
- Contains: Single HTML file, logo asset, planning documentation
- Key files: `gsd.html`

**Planning (`.planning/codebase/`):**
- Purpose: Structured documentation about the codebase itself
- Contains: Architecture, structure, conventions, and testing documentation
- Key files: `ARCHITECTURE.md`, `STRUCTURE.md`, `CONVENTIONS.md`, `TESTING.md`

**GSD Core (`get-shit-done/`):**
- Purpose: The actual GSD command-line tool repository (submodule or separate project)
- Contains: Full GSD implementation with plugins, skills, and workflow logic
- Key files: Not analyzed in this pass

## Key File Locations

**Entry Points:**
- `gsd.html`: Complete web application — serves as HTML, CSS, and JavaScript bundle

**Configuration:**
- None — zero-config static file

**Core Logic:**
- `gsd.html` lines 356-1760: All JavaScript logic
  - Lines 365-1120: `CMDS` array (60+ command definitions)
  - Lines 1125-1134: `CAT` category metadata
  - Lines 1137-1142: State variables
  - Lines 1145-1186: Utility functions
  - Lines 1189-1205: View switching
  - Lines 1208-1280: Commands view rendering
  - Lines 1281-1364: Wizard decision tree
  - Lines 1367-1524: Wizard rendering logic
  - Lines 1527-1615: Core workflow flow rendering
  - Lines 1618-1651: Cheatsheet rendering
  - Lines 1654-1724: Concepts rendering
  - Lines 1727-1755: Sidebar navigation handler
  - Lines 1758-1760: Initialization

**Styling:**
- `gsd.html` lines 10-234: All CSS
  - Lines 13-42: CSS custom properties (design tokens)
  - Lines 44-92: Layout styles (app grid, topbar, sidebar, main)
  - Lines 94-152: View and page header styles
  - Lines 154-183: Search and filter styles
  - Lines 185-234: Command list, wizard, cheatsheet, flow, concepts styles

**Assets:**
- `gsd-logo-2000-transparent.png`: Brand logo used in topbar

## Naming Conventions

**Files:**
- `gsd-{feature}.html` pattern for potential future splits (currently single file)
- `{FEATURE}.md` for documentation (uppercase, hyphenated)

**CSS Classes:**
- BEM-inspired: `.cmd-row`, `.cmd-name`, `.cmd-detail`, `.cmd-short`
- View-specific: `.view-commands`, `.view-wizard`, `.view-flow`
- State classes: `.active`, `.open`, `.on`, `.done`, `.show`
- Utility prefixes: `.badge-`, `.btn-`, `.chip-`

**JavaScript Functions:**
- `render{View}()` — View rendering: `renderCommands()`, `renderWizard()`, `renderFlow()`
- `switchView()` — Navigation
- `{action}{Target}()` — Actions: `copyText()`, `showToast()`
- `escHtml()` — Sanitization
- `getFiltered()` — Data filtering

**Data Structures:**
- `CMDS` — Command array (uppercase plural)
- `CAT` — Category metadata (uppercase abbreviation)
- `WIZARD_TREE` — Wizard decision tree (uppercase with underscore)
- `FLOW_STAGES` — Workflow stages (uppercase with underscore)
- `CONCEPTS` — Concept cards array (uppercase plural)

## Where to Add New Code

**New Command:**
- Add object to `CMDS` array in `gsd.html` (after line 1118, before closing bracket)
- Follow existing schema: `{ id, cmd, cat, short, desc, flags, usage, when, related, runtimes }`
- Category must match a key in `CAT` object

**New View:**
1. Add `<section class="view" id="view-{name}">` in main content area (after line 352)
2. Add sidebar navigation item (after line 302)
3. Add `render{Name}()` function (before line 1724)
4. Call renderer in `switchView()` (add to line 1201-1204)

**New CSS Component:**
- Add to `<style>` block (before line 234, within appropriate section comment)
- Use existing CSS custom properties for colors and spacing
- Follow responsive pattern with `@media (max-width: 860px)` if needed

**New Wizard Question:**
- Add to `WIZARD_TREE` array (line 1281-1364)
- Use `next: N` to point to next question index, or `result: "cmd-id"` for final result
- Update `totalSteps` in `renderWizardStep()` if adding new depth level

## Special Directories

**`.planning/`:**
- Purpose: GSD-generated structured artifacts and documentation
- Generated: Yes — by GSD commands
- Committed: Yes — preserved across sessions

**`.planning/codebase/`:**
- Purpose: Codebase analysis documents for GSD's own structure
- Generated: Manually or via `/gsd-map-codebase`
- Committed: Yes — reference documentation

## File Size Breakdown

| Section | Lines | Percentage |
|---------|-------|------------|
| HTML Structure | ~120 | 7% |
| CSS Styles | ~225 | 13% |
| JavaScript Data (CMDS, CAT, etc.) | ~760 | 43% |
| JavaScript Logic (functions) | ~600 | 34% |
| Blank/Comments | ~57 | 3% |
| **Total** | **1762** | **100%** |

## Component Breakdown

**Views (5 total):**
1. `#view-commands` — Command list with search, filters, expandable details
2. `#view-wizard` — Interactive decision tree for command recommendation
3. `#view-flow` — Core workflow sequence visualization
4. `#view-cheatsheet` — Compact command reference table
5. `#view-concepts` — Key concept cards explaining GSD patterns

**Data Models:**
- `CMDS`: 60+ command objects across 8 categories
- `RUNTIMES`: 15 supported AI coding tools
- `CAT`: 8 category metadata objects
- `WIZARD_TREE`: 8-step decision tree with 28 options
- `FLOW_STAGES`: 3 workflow stages with 19 total steps
- `CONCEPTS`: 8 key concept cards

---

*Structure analysis: 2026-04-19*
