# Coding Conventions

**Analysis Date:** 2026-04-19

## Project Overview

**Type:** Single-file static web application

**Primary File:** `gsd.html` (1762 lines)

**Purpose:** Interactive command router/documentation UI for the GSD (Get Shit Done) CLI tool

---

## Naming Patterns

### Files

- **HTML files:** `kebab-case.html` (e.g., `gsd.html`)
- **Images:** `kebab-case-description-size.png` (e.g., `gsd-logo-2000-transparent.png`)

### CSS Classes

- **Pattern:** `kebab-case` with semantic naming
- **Prefix convention:** Component-based prefixes

```css
/* Layout components */
.app, .topbar, .sidebar, .main, .view

/* Sub-components use parent prefix */
.topbar-brand, .topbar-right, .topbar-link
.sidebar-section, .sidebar-label, .sidebar-item, .sidebar-dot
.cmd-list, .cmd-row, .cmd-name, .cmd-short, .cmd-detail, .cmd-meta

/* State modifiers */
.active, .open, .on, .done, .show
```

### CSS Custom Properties

- **Pattern:** `--category-name` with numeric suffixes for variants

```css
:root {
  /* Colors */
  --bg, --bg1, --bg2, --bg3          /* Background variants */
  --line, --line2                     /* Border colors */
  --t0, --t1, --t2, --t3              /* Text colors (0 = lightest) */
  --accent, --accent-dim, --accent-dim2
  --red, --blue, --purple, --green, --orange, --pink, --teal
  
  /* Spacing/sizing */
  --radius, --radius-sm, --radius-lg
}
```

### JavaScript Variables

- **Constants:** `UPPER_SNAKE_CASE`

```javascript
const RUNTIMES = [...];
const CMDS = [...];
const CAT = {...};
const WIZARD_TREE = [...];
const FLOW_STAGES = [...];
const CONCEPTS = [...];
```

- **State variables:** `camelCase`

```javascript
let activeView = "commands";
let activeFilter = "all";
let searchVal = "";
let openCmd = null;
let wizardHistory = [];
let wizardStep = 0;
```

- **Function names:** `camelCase`

```javascript
function copyText(text, btn) { ... }
function showToast(msg) { ... }
function switchView(id) { ... }
function renderCommands() { ... }
function renderWizardStep(node) { ... }
function escHtml(s) { ... }
```

---

## Code Style Patterns

### HTML Structure

- **Semantic elements:** `<header>`, `<nav>`, `<main>`, `<section>`
- **Class naming:** Descriptive, component-based
- **Comments:** Box-drawing characters for section dividers

```html
<!-- ── Top bar ───────────────────────────── -->
<header class="topbar">...</header>

<!-- ── Sidebar ───────────────────────────── -->
<nav class="sidebar" id="sidebar">...</nav>

<!-- ── Main ─────────────────────────────── -->
<main class="main">...</main>
```

### CSS Organization

- **Order:** Reset → Variables → Layout → Components → Utilities → Responsive
- **Section comments:** Box-drawing dividers (40+ dashes)

```css
/* ── Layout ─────────────────────────────── */
.app { ... }

/* ── Top bar ─────────────────────────────── */
.topbar { ... }

/* ── Sidebar ─────────────────────────────── */
.sidebar { ... }

/* ── Responsive ──────────────────────────── */
@media (max-width: 860px) { ... }
```

### JavaScript Structure

- **Section comments:** Box-drawing with 61 equals signs

```javascript
/* ═══════════════════════════════════════════════════════════
   DATA — sourced from https://github.com/gsd-build/get-shit-done
   ═══════════════════════════════════════════════════════════ */

/* ═══════════════════════════════════════════════════════════
   STATE
   ═══════════════════════════════════════════════════════════ */

/* ═══════════════════════════════════════════════════════════
   UTILITIES
   ═══════════════════════════════════════════════════════════ */
```

### Function Declarations

- **Named functions:** Use `function` keyword (not arrow functions)

```javascript
function copyText(text, btn) {
  navigator.clipboard.writeText(text).then(() => {
    ...
  });
}

function showToast(msg) {
  const t = document.getElementById("toast");
  ...
}
```

- **Callbacks:** Use arrow functions

```javascript
list.querySelectorAll(".cmd-row").forEach(row => {
  row.addEventListener("click", () => {
    const id = row.dataset.id;
    openCmd = openCmd === id ? null : id;
    renderCommands();
  });
});
```

### Template Literals

- **Multi-line HTML:** Use template literals with consistent indentation

```javascript
card.innerHTML = `
  <div class="wizard-step active">
    <div class="wizard-progress">${dots}</div>
    <div class="wizard-q">${escHtml(node.q)}</div>
    <div class="wizard-sub">${escHtml(node.sub)}</div>
    ...
  </div>`;
```

### Data Structures

- **Command objects:** Consistent property ordering

```javascript
{
  id: "command-id",
  cmd: "/gsd-command",
  cat: "category",
  short: "Short description",
  desc: "Full description",
  flags: ["--flag1", "--flag2"],
  usage: "/gsd-command arg",
  when: "When to use",
  related: ["other-command"],
  runtimes: RUNTIMES
}
```

---

## Documentation Standards

### Inline Comments

- **Section headers:** Box-drawing characters for visual separation
- **Data attribution:** Comments document source of data

```javascript
/* ═══════════════════════════════════════════════════════════
   DATA — sourced from https://github.com/gsd-build/get-shit-done
   Verified April 2025. Commands, flags, and descriptions match
   the official README and CHANGELOG exactly.
   ═══════════════════════════════════════════════════════════ */
```

### JSDoc

- **Not used:** Single-file application with self-documenting code structure

---

## Import Organization

- **Not applicable:** No module system — all code in single HTML file

---

## Error Handling

**Patterns:**

- **Graceful fallbacks:** Default values for missing data

```javascript
const meta = CAT[c.cat] || CAT.util;
```

- **Null checks:** Guard clauses before DOM operations

```javascript
const v = document.getElementById("view-" + id);
if (v) v.classList.add("active");
```

- **Event delegation:** Use `closest()` for nested event targets

```javascript
const chip = e.target.closest(".chip");
if (!chip) return;
```

---

## Logging

**Framework:** `console` only (no external logging library)

**Patterns:**

- No console statements in production code
- Errors handled silently with fallbacks

---

## Function Design

**Size:** Functions are focused and single-purpose (10-50 lines typical)

**Parameters:**

- Few parameters (1-3 typical)
- Related parameters grouped as object when needed

**Return Values:**

- Most functions return `void` (DOM manipulation)
- Helper functions return transformed values

```javascript
function escHtml(s) {
  return String(s)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;");
}
```

---

## Module Design

**Exports:** Not applicable — single file, no module system

**Barrel Files:** Not applicable

---

## Responsive Design

**Breakpoints:**

```css
@media (max-width: 860px) {
  /* Tablet/mobile layout changes */
  .app { grid-template-columns: 1fr; }
  .sidebar { flex-direction: row; overflow-x: auto; }
  .cmd-row { grid-template-columns: 1fr auto; }
  .cmd-short { display: none; }
}
```

---

## Accessibility

**Patterns observed:**

- **ARIA attributes:** `role="button"`, `aria-expanded`

```html
<div class="cmd-row" role="button" tabindex="0" aria-expanded="false">
```

- **Keyboard navigation:** Enter/Space key handlers

```javascript
row.addEventListener("keydown", e => {
  if (e.key === "Enter" || e.key === " ") {
    e.preventDefault();
    row.click();
  }
});
```

- **Focus management:** Keyboard shortcut to focus search

```javascript
if (e.key === "/" && document.activeElement !== search) {
  e.preventDefault();
  search.focus();
}
```

---

*Convention analysis: 2026-04-19*
