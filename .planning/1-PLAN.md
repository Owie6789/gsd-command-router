# Phase 1 Plan: Foundation & Search

**Phase Number:** 1
**Created:** 2026-04-19
**Status:** Ready for Execution

---

## Overview

**Goal:** Users can find the right command faster with enhanced search, runtime filtering, and keyboard navigation.

**Success Criteria:** 10 items (see ROADMAP.md)

**Locked Decisions:** 8 items (see 1-CONTEXT.md)

**Files to Modify:** `gsd.html`

**Estimated Effort:** 10 tasks × ~2 hours = 20 hours

---

## Tasks

### TASK-01: Codebase Analysis & Setup

**Dependencies:** None

**Acceptance Criteria:**
- [ ] Understand current search/filter architecture
- [ ] Identify all functions that need modification
- [ ] Map CSS variables and structure
- [ ] Create mental model of state management

**Implementation Notes:**
- Read `getFiltered()` function (current search logic)
- Read `renderCommands()` function (current rendering)
- Read sidebar filter event listeners
- Note: `CMDS` array has `runtimes` property on each command
- Note: `RUNTIMES` constant has 15 runtime strings
- Current state: `activeFilter`, `searchVal`, `openCmd`

**Verification:**
- Can explain how search currently works
- Can identify where to add runtime filter logic

---

### TASK-02: Add Settings Modal HTML Structure

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] Gear icon added to top bar next to GitHub link
- [ ] Settings modal HTML added (hidden by default)
- [ ] Modal has checklist of all 15 runtimes
- [ ] "Select All" and "Clear All" buttons work
- [ ] "Save" button closes modal and persists selection
- [ ] "Cancel" button closes modal without saving

**Implementation Notes:**
- Add gear icon SVG in `.topbar-right` div
- Modal structure:
  ```html
  <div id="settings-modal" class="modal" style="display:none">
    <div class="modal-content">
      <h2>Settings</h2>
      <div class="runtime-list"><!-- 15 checkboxes --></div>
      <div class="modal-actions">
        <button id="select-all">Select All</button>
        <button id="clear-all">Clear All</button>
        <button id="save-settings">Save</button>
        <button id="cancel-settings">Cancel</button>
      </div>
    </div>
  </div>
  ```
- Add CSS for modal overlay and content box
- Modal should be centered, with backdrop blur

**Verification:**
- Click gear icon → modal opens
- Click Save → modal closes
- Click Cancel → modal closes

---

### TASK-03: Settings Modal Logic & Persistence

**Dependencies:** TASK-02

**Acceptance Criteria:**
- [ ] Gear icon click opens modal
- [ ] "Select All" checks all 15 runtime checkboxes
- [ ] "Clear All" unchecks all runtime checkboxes
- [ ] "Save" persists to localStorage as `gsd:runtimePreferences`
- [ ] "Cancel" does not persist changes
- [ ] Modal loads saved preferences on open
- [ ] Default: all 15 runtimes selected if no saved preference

**Implementation Notes:**
- localStorage key: `gsd:runtimePreferences`
- Value: JSON array of selected runtime names
- Functions to add:
  ```javascript
  function openSettingsModal()
  function closeSettingsModal(save)
  function loadRuntimePreferences()
  function saveRuntimePreferences(selected)
  function renderRuntimeChecklist(selected)
  ```
- Event listeners for all buttons

**Verification:**
- Select 5 runtimes, Save, reload page → re-open settings, 5 should be checked
- Clear All, Save, reload → none checked
- First time: all 15 should be checked by default

---

### TASK-04: Runtime Filter Chips (Hybrid UI)

**Dependencies:** TASK-03

**Acceptance Criteria:**
- [ ] Filter chips row shows only selected runtimes from settings
- [ ] First 4 popular runtimes shown as chips (Claude Code, Cursor, Copilot, Windsurf)
- [ ] "More" chip shows dropdown with remaining selected runtimes
- [ ] Click chip → toggle selection
- [ ] Multiple chips can be selected (multi-select, not radio)
- [ ] Visual active state for selected chips
- [ ] If no runtimes selected, show all commands (filter inactive)

**Implementation Notes:**
- Add runtime chips container after category chips
- Chip structure:
  ```html
  <button class="chip runtime-chip" data-runtime="Claude Code">Claude Code</button>
  <button class="chip runtime-chip" data-runtime="More">More ▼</button>
  ```
- "More" dropdown appears on hover/click
- Selected state: `chip.active` class (accent background)
- Function: `renderRuntimeChips()` called after settings save

**Verification:**
- Select 6 runtimes in settings → 4 chips + "More" with 2 in dropdown
- Click "Claude Code" chip → filters to Claude Code commands
- Click again → removes filter

---

### TASK-05: AND/OR Filter Toggle

**Dependencies:** TASK-04

**Acceptance Criteria:**
- [ ] Toggle switch appears between category and runtime chips
- [ ] Label shows "Match: ALL (AND)" or "Match: ANY (OR)"
- [ ] Toggle only visible when both category AND runtime filters active
- [ ] AND mode: command must match both filters
- [ ] OR mode: command matches either filter
- [ ] Toggle state persists in localStorage as `gsd:filterLogic`
- [ ] Default: AND

**Implementation Notes:**
- Toggle HTML:
  ```html
  <div class="filter-toggle">
    <span>Match:</span>
    <button class="toggle-btn" data-logic="and">ALL</button>
    <button class="toggle-btn" data-logic="or">ANY</button>
  </div>
  ```
- Modify `getFiltered()` to check `filterLogic` state
- AND logic: `matchesCategory && matchesRuntime`
- OR logic: `matchesCategory || matchesRuntime`

**Verification:**
- Select "Core" category + "Claude Code" runtime
- Toggle to AND → see only Core commands for Claude Code
- Toggle to OR → see Core commands + all Claude Code commands

---

### TASK-06: Weighted Search Scoring

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] Search uses weighted scoring algorithm
- [ ] Command name matches: 50 points
- [ ] Short description matches: 30 points
- [ ] Flags match: 15 points
- [ ] When-to-use matches: 5 points
- [ ] Results sorted by score descending
- [ ] Commands with score 0 excluded
- [ ] Multi-word queries boost commands matching all words

**Implementation Notes:**
- Replace current `getFiltered()` search logic with:
  ```javascript
  function scoreCommand(cmd, query) {
    let score = 0
    const q = query.toLowerCase()
    if (cmd.cmd.toLowerCase().includes(q)) score += 50
    if (cmd.short.toLowerCase().includes(q)) score += 30
    if (cmd.flags.join(' ').toLowerCase().includes(q)) score += 15
    if (cmd.when.toLowerCase().includes(q)) score += 5
    return score
  }
  ```
- Filter: `CMDS.filter(cmd => scoreCommand(cmd, searchVal) > 0)`
- Sort: `.sort((a, b) => scoreCommand(b, searchVal) - scoreCommand(a, searchVal))`

**Verification:**
- Search "plan" → `/gsd-plan-phase` ranks #1
- Search "review" → `/gsd-review` ranks #1
- Search "fast" → `/gsd-fast` ranks #1

---

### TASK-07: Search Debouncing (300ms)

**Dependencies:** TASK-06

**Acceptance Criteria:**
- [ ] Search input has 300ms debounce
- [ ] Typing doesn't trigger immediate re-render
- [ ] Stop typing for 300ms → search executes
- [ ] Previous timer cleared on new keystroke
- [ ] Debounce only on search input, not filter changes

**Implementation Notes:**
- Add debounce timer variable: `let searchDebounceTimer = null`
- Modify search input listener:
  ```javascript
  searchInput.addEventListener('input', (e) => {
    clearTimeout(searchDebounceTimer)
    searchDebounceTimer = setTimeout(() => {
      searchVal = e.target.value
      renderCommands()
    }, 300)
  })
  ```

**Verification:**
- Type quickly: "plan-phase" → should see only 1 re-render after stopping
- Use browser DevTools Performance tab to verify no excessive re-renders

---

### TASK-08: Keyboard Navigation (Arrow Keys)

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] ArrowDown moves focus to next visible command
- [ ] ArrowUp moves focus to previous visible command
- [ ] Home jumps to first visible command
- [ ] End jumps to last visible command
- [ ] Enter expands/collapses focused command
- [ ] Tab into command list focuses first visible command
- [ ] Shift+Tab from command list focuses search input
- [ ] Navigation loops: end wraps to start (optional)

**Implementation Notes:**
- Track focused command index: `let focusedCommandIndex = -1`
- Add keydown listener on command list:
  ```javascript
  cmdList.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowDown') {
      e.preventDefault()
      focusNextVisibleCommand()
    } else if (e.key === 'ArrowUp') {
      e.preventDefault()
      focusPrevVisibleCommand()
    } else if (e.key === 'Enter') {
      e.preventDefault()
      toggleFocusedCommand()
    }
    // ... etc
  })
  ```
- Focus style: `outline: 2px solid var(--accent); outline-offset: 2px;`

**Verification:**
- Use Tab to focus search, then Tab again → first command focused
- Arrow Down → next command focused
- Enter → command expands
- Arrow Down 10 times → wraps to top (if looping enabled)

---

### TASK-09: Smart Sequential Escape Handler

**Dependencies:** TASK-08

**Acceptance Criteria:**
- [ ] 1st Escape press closes expanded command
- [ ] 2nd Escape press clears search
- [ ] 3rd Escape press blurs focus
- [ ] Counter resets after 2 seconds
- [ ] Works regardless of current focus
- [ **Edge Case**]: If settings modal open, 1st Escape closes modal

**Implementation Notes:**
- Add escape counter: `let escapeCount = 0; let escapeTimer = null`
- Global keydown listener:
  ```javascript
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
      handleEscape()
    }
  })
  
  function handleEscape() {
    if (document.getElementById('settings-modal').style.display === 'block') {
      closeSettingsModal(false)
      return
    }
    if (openCmd) {
      openCmd = null
      renderCommands()
    } else if (searchVal) {
      searchVal = ''
      document.getElementById('search').value = ''
      renderCommands()
    } else {
      document.getElementById('search').blur()
    }
    resetEscapeCounter()
  }
  ```

**Verification:**
- Expand command, press Escape → closes
- Type in search, press Escape → clears
- Press Escape again → search blurs
- Wait 3 seconds, press Escape twice quickly → should work as expected

---

### TASK-10: Focus Ring CSS & Integration Testing

**Dependencies:** TASK-02, TASK-08

**Acceptance Criteria:**
- [ ] All interactive elements have focus rings
- [ ] Focus ring: 2px solid accent color with 2px offset
- [ ] Remove default browser outline
- [ ] Test all 10 success criteria manually
- [ ] Document any bugs found and fix them

**Implementation Notes:**
- Add CSS:
  ```css
  :focus {
    outline: 2px solid var(--accent);
    outline-offset: 2px;
  }
  :focus:not(:focus-visible) {
    outline: none;
  }
  ```
- Apply to: buttons, chips, search input, command rows, modal elements
- Create test checklist for all 10 criteria

**Verification:**
- Tab through entire UI → every interactive element has focus ring
- Run full verification protocol (see below)

---

## Verification Protocol

**Execute after all tasks complete. All 10 criteria MUST pass.**

### Criterion 1: Runtime Filter
1. Open Settings (gear icon)
2. Select only "Claude Code" and "Cursor"
3. Save settings
4. Click "Claude Code" chip
5. **Expected:** Only commands supporting Claude Code shown

### Criterion 2: Category + Runtime Combination
1. Click "Core" category chip
2. Click "Claude Code" runtime chip
3. Toggle to AND
4. **Expected:** Only Core commands for Claude Code
5. Toggle to OR
6. **Expected:** Core commands + all Claude Code commands

### Criterion 3: Arrow Key Navigation
1. Press Tab to focus search
2. Press Tab again
3. **Expected:** First command focused (visible ring)
4. Press Arrow Down 3 times
5. **Expected:** Focus moves down 3 commands

### Criterion 4: Enter to Expand/Collapse
1. Focus a collapsed command
2. Press Enter
3. **Expected:** Command expands
4. Press Enter again
5. **Expected:** Command collapses

### Criterion 5: Smart Sequential Escape
1. Expand a command
2. Press Escape
3. **Expected:** Command collapses
4. Type in search box
5. Press Escape
6. **Expected:** Search clears
7. Press Escape again
8. **Expected:** Search box blurs

### Criterion 6: Settings Modal Runtime Config
1. Click gear icon
2. **Expected:** Modal opens with 15 runtime checkboxes
3. Click "Select All"
4. **Expected:** All 15 checked
5. Click "Clear All"
6. **Expected:** All 15 unchecked
7. Select 5 runtimes, click Save
8. Reload page, open settings
9. **Expected:** Same 5 runtimes checked

### Criterion 7: Weighted Search Ranking
1. Type "plan" in search
2. **Expected:** `/gsd-plan-phase` is first result
3. Type "review"
4. **Expected:** `/gsd-review` is first result

### Criterion 8: Search Debouncing
1. Open DevTools Performance tab
2. Type "plan-phase" quickly
3. **Expected:** Only 1-2 re-renders after stopping (not on every keystroke)

### Criterion 9: Focus Rings
1. Press Tab repeatedly
2. **Expected:** Every focused element has visible yellow/green ring
3. Ring should not overlap element border (offset present)

### Criterion 10: Runtime Persistence
1. Open Settings, select 3 runtimes
2. Save, reload page
3. **Expected:** Only 3 runtime chips visible
4. Open Settings again
5. **Expected:** Same 3 runtimes checked

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Modal CSS conflicts | Test in isolation first, use z-index 999 |
| localStorage undefined | Check `typeof localStorage !== 'undefined'` |
| Keyboard nav conflicts | Test with screen reader, ensure Tab works independently |
| Debounce feels laggy | Adjust to 250ms if 300ms feels slow |
| AND/OR toggle confusing | Add tooltip, clear labeling |

---

## Out of Scope (Explicitly Deferred)

- ❌ Command history tracking (Phase 2)
- ❌ Favorites system (Phase 2)
- ❌ Export functionality (Phase 3)
- ❌ Code splitting (Phase 4)
- ❌ Unit tests (Phase 4)
- ❌ Light theme (Phase 5)
- ❌ Virtual scrolling (Phase 5)

---

*1-PLAN.md created: 2026-04-19*
*Ready for execution via /gsd-execute-phase 1*
