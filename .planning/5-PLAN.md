# Phase 5 Plan: Performance & Polish

**Phase Number:** 5
**Created:** 2026-04-19
**Status:** Ready for Execution

---

## Overview

**Goal:** Application feels responsive and accessible. Visual polish to prevent flat appearance.

**Success Criteria:** 6 items (see 5-CONTEXT.md)

**Locked Decisions:** 3 items (see 5-CONTEXT.md)

**Files to Modify:** `gsd.html`

**Estimated Effort:** 5 tasks × ~2 hours = 10 hours

---

## Tasks

### TASK-01: Virtual Scrolling Infrastructure

**Dependencies:** None

**Acceptance Criteria:**
- [ ] `VIRTUAL_SCROLL_THRESHOLD = 100` constant added
- [ ] Placeholder function `renderVirtualizedCommands()` with TODO comment
- [ ] Comment in `renderCommands()` explaining when to activate virtual scrolling
- [ ] Documentation added to AI Maintenance Guide

**Implementation Notes:**
```javascript
// Add near top of JavaScript section (after appState)
const VIRTUAL_SCROLL_THRESHOLD = 100; // Activate virtual scrolling when commands exceed this

// Add placeholder function (after renderCommands)
function renderVirtualizedCommands() {
  // TODO: Implement virtual scrolling when command count exceeds VIRTUAL_SCROLL_THRESHOLD
  // For now, delegate to standard renderCommands()
  // 
  // Implementation notes for future:
  // 1. Only render visible commands (e.g., 20 at a time)
  // 2. Use scroll position to determine which commands to render
  // 3. Add spacer divs to maintain scroll height
  // 4. See: https://github.com/bvaughn/react-virtualized (reference only)
  // 
  // To activate:
  // - Replace renderCommands() call with renderVirtualizedCommands()
  // - Implement windowing logic
  // - Test with 100+ commands
  renderCommands();
}
```

**Update renderCommands():**
```javascript
function renderCommands() {
  // Check if virtual scrolling should be activated
  const filtered = getFiltered();
  if (filtered.length > VIRTUAL_SCROLL_THRESHOLD) {
    console.warn(`Command count (${filtered.length}) exceeds threshold (${VIRTUAL_SCROLL_THRESHOLD}). Consider activating virtual scrolling.`);
    // For now, continue with standard rendering
  }
  // ... existing code ...
}
```

**Update AI Maintenance Guide:**
Add section under "COMMON PATTERNS":
```markdown
### Virtual Scrolling (Performance Optimization)

When command count exceeds 100, activate virtual scrolling:

1. Find `renderVirtualizedCommands()` function
2. Implement windowing logic (render only visible commands)
3. Replace `renderCommands()` call with `renderVirtualizedCommands()`

Current implementation: Standard rendering (fine for <100 commands)
```

**Verification:**
- Constant exists and is 100
- Placeholder function exists with TODO comment
- Warning logs when commands exceed threshold
- AI Maintenance Guide updated

---

### TASK-02: Accessibility Audit — Commands View

**Dependencies:** None

**Acceptance Criteria:**
- [ ] Test keyboard navigation (Tab, Enter, Escape, Arrows)
- [ ] Test screen reader (NVDA or VoiceOver)
- [ ] Document all issues found
- [ ] Fix all verified issues

**Audit Checklist:**
```markdown
## Commands View Audit

### Keyboard Navigation
- [ ] Search input focusable with Tab
- [ ] Filter chips focusable and activatable with Enter/Space
- [ ] Command rows focusable
- [ ] Arrow keys navigate command list
- [ ] Enter expands/collapses command
- [ ] Escape closes expanded command
- [ ] Escape clears search (double press)
- [ ] Favorite button focusable and activatable
- [ ] Copy button focusable and activatable
- [ ] Related commands focusable and activatable

### Screen Reader
- [ ] Search input has aria-label
- [ ] Filter chips have aria-label (e.g., "Filter by Core category")
- [ ] Command rows announce command name and description
- [ ] Favorite button has aria-label ("Add to favorites" or "Remove from favorites")
- [ ] Copy button has aria-label ("Copy usage to clipboard")
- [ ] Expanded state announced ("Expanded" or "Collapsed")
- [ ] Related commands section has heading

### Focus Management
- [ ] Focus visible on all elements (accent ring)
- [ ] Focus order is logical (left to right, top to bottom)
- [ ] No focus traps
- [ ] Focus restored after actions (copy, favorite)

### Issues Found:
(List each issue with severity: Critical, Major, Minor)

1. 
2. 
3. 
```

**Implementation Notes:**
- Use Windows NVDA or Mac VoiceOver for screen reader testing
- Test with keyboard only (unplug mouse)
- Document issues in checklist above
- Fix each issue systematically

**Common Fixes:**
```javascript
// Add aria-label to search input
<input type="text" id="search" aria-label="Search commands, descriptions, flags...">

// Add aria-label to favorite button
<button class="favorite-btn" 
        aria-label="${isFavorite ? 'Remove from favorites' : 'Add to favorites'}">

// Add aria-label to copy button
<button class="copy-btn" aria-label="Copy usage to clipboard">

// Add role and aria-expanded to command rows
<div class="cmd-row" 
     role="button" 
     tabindex="0" 
     aria-expanded="${isOpen}">

// Add heading to related commands section
<div class="detail-block">
  <label>Related Commands</label>
  <div class="related-list" role="list">
    ${c.related.map(r => 
      `<span class="related-cmd" role="listitem" ...>`
    ).join("")}
  </div>
</div>
```

**Verification:**
- All checklist items pass
- No critical or major issues remain
- Minor issues documented (fix or defer)

---

### TASK-03: Accessibility Audit — Other Views

**Dependencies:** TASK-02 (use same audit approach)

**Acceptance Criteria:**
- [ ] Audit Wizard view
- [ ] Audit Settings modal
- [ ] Audit Sidebar navigation
- [ ] Audit all other views (Flow, Cheatsheet, Concepts, FAQ, Compare, Quick Start)
- [ ] Document all issues found
- [ ] Fix all verified issues

**Audit Checklists:**

```markdown
## Wizard View Audit

### Keyboard Navigation
- [ ] Wizard options focusable and activatable with Enter/Space
- [ ] Back button focusable and activatable
- [ ] Restart button focusable
- [ ] Progress indicators visible

### Screen Reader
- [ ] Wizard question announced
- [ ] Wizard options announced as list
- [ ] Progress announced ("Step 1 of 3")
- [ ] Result announced clearly

### Issues Found:
1. 
2. 

---

## Settings Modal Audit

### Keyboard Navigation
- [ ] Modal opens with Enter on gear icon
- [ ] All settings focusable
- [ ] Tab cycles through all inputs
- [ ] Escape closes modal
- [ ] Focus trapped inside modal (can't tab outside)
- [ ] Focus restored to gear icon when modal closes

### Screen Reader
- [ ] Modal has role="dialog"
- [ ] Modal has aria-modal="true"
- [ ] Modal has aria-labelledby (points to title)
- [ ] All inputs have labels
- [ ] Color swatches have aria-labels ("Yellow theme", "Red theme", etc.)
- [ ] Runtime checkboxes have labels

### Issues Found:
1. 
2. 

---

## Sidebar Navigation Audit

### Keyboard Navigation
- [ ] All nav items focusable
- [ ] Enter activates nav item
- [ ] Active item indicated visually and to screen reader
- [ ] Section headers not focusable (decorative)

### Screen Reader
- [ ] Sidebar has role="navigation"
- [ ] Section headers have appropriate heading level
- [ ] Active state announced ("All Commands, selected")

### Issues Found:
1. 
2. 

---

## All Views Audit

### Global
- [ ] Skip-to-content link present
- [ ] Page has <h1>
- [ ] Heading hierarchy logical (h1 → h2 → h3)
- [ ] Landmark regions defined (<main>, <nav>, <header>)
- [ ] Color contrast meets WCAG AA (4.5:1 normal text, 3:1 large)

### Issues Found:
1. 
2. 
```

**Implementation Notes:**
- Test each view systematically
- Use same approach as TASK-02
- Fix issues as found, or document for batch fixing

**Common Fixes:**
```javascript
// Add skip-to-content link (after <body>)
<a href="#main-content" 
   style="position:absolute;left:-9999px" 
   onfocus="this.style.left='0'">
  Skip to content
</a>

// Add landmark roles
<nav class="sidebar" role="navigation" aria-label="Main navigation">
<main id="main-content" role="main">
<header class="topbar" role="banner">

// Add modal attributes
<div class="modal-overlay" 
     id="settings-modal" 
     role="dialog" 
     aria-modal="true" 
     aria-labelledby="modal-title">
  <div class="modal-content">
    <h2 id="modal-title">Settings</h2>

// Trap focus in modal (add to openSettingsModal)
function openSettingsModal() {
  // ... existing code ...
  const focusableElements = modal.querySelectorAll('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
  const firstFocusable = focusableElements[0];
  const lastFocusable = focusableElements[focusableElements.length - 1];
  
  // Trap focus
  modal.addEventListener('keydown', e => {
    if (e.key === 'Tab') {
      if (e.shiftKey && document.activeElement === firstFocusable) {
        e.preventDefault();
        lastFocusable.focus();
      } else if (!e.shiftKey && document.activeElement === lastFocusable) {
        e.preventDefault();
        firstFocusable.focus();
      }
    }
  });
}
```

**Verification:**
- All view checklists completed
- No critical or major issues remain
- All views keyboard navigable

---

### TASK-04: Fix All Verified Accessibility Issues

**Dependencies:** TASK-02, TASK-03

**Acceptance Criteria:**
- [ ] All critical issues fixed
- [ ] All major issues fixed
- [ ] Minor issues fixed or documented for future
- [ ] Re-test all views to verify fixes
- [ ] No regressions introduced

**Implementation Notes:**
- Group fixes by category (ARIA, keyboard, focus, contrast)
- Test each fix before moving to next
- Keep git commits atomic (one fix per commit)

**Verification:**
- Run full keyboard navigation test on all views
- Run screen reader test on key views (Commands, Settings)
- Verify no features broken by fixes

---

### TASK-05: Background Grid Pattern

**Dependencies:** None

**Acceptance Criteria:**
- [ ] Subtle grid/white pattern added to background
- [ ] Pattern visible but not distracting
- [ ] Pattern on edges/corners (not center content)
-   - Pattern works at different zoom levels (100%, 125%, 150%)
- [ ] Pattern doesn't interfere with content readability

**Implementation Notes:**
```css
/* Add to CSS section (after body styles) */

/* Option 1: Subtle grid pattern (full background) */
body {
  background: 
    linear-gradient(
      to right,
      rgba(255,255,255,0.03) 1px,
      transparent 1px
    ),
    linear-gradient(
      to bottom,
      rgba(255,255,255,0.03) 1px,
      transparent 1px
    ),
    var(--bg);
  background-size: 24px 24px;
}

/* Option 2: Edge vignette with subtle pattern (recommended) */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: 
    radial-gradient(
      ellipse at center,
      transparent 0%,
      transparent 50%,
      rgba(255,255,255,0.02) 100%
    ),
    linear-gradient(
      to right,
      rgba(255,255,255,0.03) 1px,
      transparent 1px
    ),
    linear-gradient(
      to bottom,
      rgba(255,255,255,0.03) 1px,
      transparent 1px
    );
  background-size: 100% 100%, 24px 24px, 24px 24px;
  pointer-events: none;
  z-index: -1;
}
```

**Recommended Approach:** Option 2 (edge vignette + grid)
- Vignette adds depth (darker center, lighter edges)
- Grid pattern adds texture
- `pointer-events: none` ensures pattern doesn't interfere with clicks
- `z-index: -1` keeps pattern behind content

**Verification:**
- Pattern visible on large monitor (1920px+)
- Pattern subtle (barely noticeable at first glance)
- Pattern doesn't distract from content
- Pattern works at 100%, 125%, 150% zoom
- Pattern visible in corners/edges, less prominent in center

---

## Verification Protocol

**Execute after all tasks complete. All 6 criteria MUST pass.**

### Criterion 1: Virtual Scrolling Infrastructure
1. Search for `VIRTUAL_SCROLL_THRESHOLD` constant
2. **Expected:** Constant exists, value is 100
3. Search for `renderVirtualizedCommands()` function
4. **Expected:** Function exists with TODO comment
5. Check AI Maintenance Guide
6. **Expected:** Virtual scrolling section added

### Criterion 2: Accessibility Audit Completed
1. Review audit checklists in PLAN.md
2. **Expected:** All checklists filled out
3. **Expected:** Issues documented with severity

### Criterion 3: Accessibility Issues Fixed
1. Re-run keyboard navigation test on Commands view
2. **Expected:** All items focusable, no dead ends
3. Re-run screen reader test on Commands view
4. **Expected:** All elements announced correctly
5. Re-run keyboard test on Settings modal
6. **Expected:** Focus trapped, Escape closes, focus restored

### Criterion 4: Keyboard Navigation Works on ALL Views
1. Navigate to each view (Commands, Wizard, Flow, Cheatsheet, Concepts, FAQ, Compare, Quick Start)
2. Test Tab, Enter, Escape on each view
3. **Expected:** All views fully navigable with keyboard only

### Criterion 5: Background Grid Pattern
1. View site on large monitor (1920px+)
2. **Expected:** Subtle grid/vignette pattern visible
3. Zoom to 150%
4. **Expected:** Pattern scales correctly, still subtle
5. Read content for 2+ minutes
6. **Expected:** Pattern not distracting

### Criterion 6: No Breaking Changes
1. Test all Phase 1 features (search, filters, keyboard nav)
2. **Expected:** All working correctly
3. Test all Phase 1.1 features (theme colors, fonts, settings)
4. **Expected:** All working correctly
5. Test all Phase 2 features (history, favorites, persistence)
6. **Expected:** All working correctly

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Accessibility fixes break existing features | Test each fix individually, keep git history clean |
| Over-engineering virtual scrolling | Add constants and placeholder only, document "activate when 100+ commands" |
| Background pattern too distracting | Start with very low opacity (3%), increase only if barely visible |
| Accessibility audit scope creep | Stick to verified issues only, document "nice-to-haves" for future |

---

## Out of Scope (Explicitly Deferred)

The following are **NOT** in Phase 5:

- ❌ Light theme (user wants dark-only)
- ❌ Full virtual scrolling implementation (just prep)
- ❌ WCAG AAA compliance (AA is sufficient)
- ❌ Automated accessibility testing (no test infrastructure)
- ❌ Performance profiling/optimization (debouncing already done)
- ❌ Animation/transition polish
- ❌ Mobile-specific accessibility (focus on desktop first)

---

*5-PLAN.md created: 2026-04-19*
*Ready for execution via /gsd-execute-phase 5*
