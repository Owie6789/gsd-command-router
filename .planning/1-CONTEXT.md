# Phase 1 Context: Foundation & Search

**Phase Number:** 1
**Created:** 2026-04-19
**Status:** Ready for Planning

---

## Phase Goal

Users can find the right command faster with enhanced search, runtime filtering, and keyboard navigation.

---

## Implementation Decisions (Locked)

These decisions are final. Planner must respect them. Executor must implement them exactly.

### DECISION-01: Runtime Filter Configuration

**Decision:** Users can configure which runtimes to display via a Settings modal.

**Rationale:**
- 15 runtimes is too many to show as filter chips
- Users typically work with 2-4 specific AI tools
- Configuration should be accessible but not clutter the main UI
- Settings modal pattern allows future expansion (theme, shortcuts, etc.)

**Implementation Details:**
- Gear icon in top bar next to GitHub link
- Modal contains checklist of all 15 runtimes with "Select All" / "Clear All" buttons
- Selected runtimes persist in localStorage (key: `gsd:runtimePreferences`)
- Default: All 15 runtimes selected (backward compatible)
- Filter chips row only shows selected runtimes

**Files to Modify:**
- `gsd.html` — Add settings modal HTML, gear icon, runtime persistence logic
- Future: `app.js` (if split in Phase 4)

**Out of Scope:**
- Per-command runtime visibility
- Runtime popularity tracking
- Auto-suggest based on search history

---

### DECISION-02: Multi-Select Runtime Filter UI

**Decision:** Hybrid approach — show 3-4 popular runtimes as chips, rest in "More" dropdown.

**Rationale:**
- Most users work with Claude Code, Cursor, or Copilot (show these prominently)
- Power users can expand "More" to access less common runtimes
- Balances visibility with screen real estate
- Chips are toggleable (multi-select, not radio buttons)

**Implementation Details:**
- Default visible chips: Claude Code, Cursor, Copilot, Windsurf
- "More" chip expands dropdown with remaining 11 runtimes as checkboxes
- Each chip has active state (filled accent color when selected)
- Multiple runtimes can be selected simultaneously
- Selecting a runtime in "More" dropdown adds it to visible chips

**UI Behavior:**
- Click chip → toggle selection
- If no runtimes selected → show all commands (runtime filter inactive)
- If 1+ runtimes selected → filter commands by selected runtimes (OR logic within runtime filter)
- Visual indicator: chip count badge when 5+ runtimes selected

**Out of Scope:**
- Drag-and-drop reordering of chips
- Runtime grouping by category (CLI vs IDE extension)
- Custom runtime aliases

---

### DECISION-03: Filter Combination Logic (AND/OR Toggle)

**Decision:** Users can toggle between AND and OR logic when combining category + runtime filters.

**Rationale:**
- AND is more common (narrow down: "Core commands for Claude Code only")
- OR is useful for exploration ("Show me Core commands OR anything that works with Gemini")
- Explicit toggle prevents confusion about why certain commands appear/disappear
- Educates users about filter behavior

**Implementation Details:**
- Toggle switch positioned between category chips and runtime chips
- Label: "Match: ALL (AND)" / "Match: ANY (OR)"
- AND mode: `command.categories.includes(filter) && command.runtimes.some(r => selectedRuntimes.includes(r))`
- OR mode: `command.categories.includes(filter) || command.runtimes.some(r => selectedRuntimes.includes(r))`
- Toggle state persists in localStorage (key: `gsd:filterLogic`)
- Default: AND (most intuitive)

**UI Behavior:**
- Toggle visible only when both category AND runtime filters are active
- Automatically hide toggle if only one filter type is active
- Tooltip on hover: "AND = show only matching both filters, OR = show matching either filter"

**Out of Scope:**
- NOT logic (exclude filters)
- Nested boolean logic (A AND (B OR C))
- Filter presets or saved combinations

---

### DECISION-04: Search Relevance Ranking

**Decision:** Weighted scoring system — Command name (50%), description (30%), flags (15%), when-to-use (5%).

**Rationale:**
- Users searching for "plan" want `/gsd-plan-phase` ranked higher than commands mentioning "plan" in description
- Weights reflect likelihood of user intent
- Prevents irrelevant results from ranking high due to keyword stuffing in long descriptions
- Fuzzy matching for typos (e.g., "planing" → "planning")

**Implementation Details:**
- Scoring algorithm:
  ```javascript
  score = 0
  if (cmd.name.toLowerCase().includes(query)) score += 50
  if (cmd.short.toLowerCase().includes(query)) score += 30
  if (cmd.flags.join(' ').toLowerCase().includes(query)) score += 15
  if (cmd.when.toLowerCase().includes(query)) score += 5
  // Fuzzy matching bonus: +10 for 1-character typo match
  ```
- Results sorted by score descending
- Commands with score 0 excluded from results
- Exact matches rank higher than partial matches within same field

**Edge Cases:**
- Multi-word queries: "plan phase" → boost commands matching both words
- Single character queries: minimum 2 characters to trigger search (prevent overload)
- Special characters: strip `/` and `-` before matching (e.g., "/gsd-plan-phase" matches "plan")

**Out of Scope:**
- Stemming/lemmatization ("planning" → "plan")
- Synonym matching ("create" → "new")
- Search analytics or "no results" tracking

---

### DECISION-05: Keyboard Navigation (Visible Only)

**Decision:** Arrow keys navigate only visible (filtered) commands.

**Rationale:**
- Most intuitive behavior — what you see is what you navigate
- Prevents confusion from "invisible" commands receiving focus
- Simpler implementation (no need to track hidden indices)
- Aligns with standard list navigation patterns (file explorers, dropdowns)

**Implementation Details:**
- `ArrowDown` → move focus to next visible command in list
- `ArrowUp` → move focus to previous visible command
- `Home` → jump to first visible command
- `End` → jump to last visible command
- `Enter` → expand/collapse focused command
- Focus ring follows DECISION-07 (outline ring style)
- Loop navigation: reaching end wraps to start (optional, configurable)

**Focus Management:**
- Search input is separate from command list navigation
- Tab into command list → focus first visible command
- Shift+Tab from command list → focus search input
- Escape behavior follows DECISION-06 (smart sequential)

**Out of Scope:**
- Type-ahead search (typing "p" jumps to commands starting with P)
- Multi-select with Space key
- Drag-and-drop reordering via keyboard

---

### DECISION-06: Escape Key Behavior (Smart Sequential)

**Decision:** Progressive dismissal — 1st press closes expanded command, 2nd clears search, 3rd blurs focus.

**Rationale:**
- Respects user's current context
- Prevents accidental data loss (clearing search unintentionally)
- Aligns with mental model of "backing out" progressively
- Standard pattern in complex UIs (IDEs, design tools)

**Implementation Details:**
```javascript
let escapeCount = 0
let escapeTimer = null

onEscape() {
  if (expandedCommand) {
    closeExpandedCommand()
    escapeCount = 0
  } else if (searchVal) {
    searchVal = ""
    searchInput.value = ""
    renderCommands()
    escapeCount = 0
  } else {
    searchInput.blur()
    escapeCount = 0
  }
}
```
- Counter resets after 2 seconds of inactivity
- Visual feedback: subtle animation on each action
- Works regardless of current focus (search input, command list, sidebar)

**Edge Cases:**
- If wizard view is active: 1st press closes wizard step, 2nd clears search, 3rd blurs
- If settings modal is open: 1st press closes modal (bypasses command close)
- If no command expanded AND search empty: blur immediately (skip to 3rd action)

**Out of Scope:**
- Undo after clearing search
- Escape to close browser tab (browser default)
- Customizable escape behavior

---

### DECISION-07: Focus Indicator Style

**Decision:** Standard outline ring (2-3px solid border) for WCAG compliance.

**Rationale:**
- Highest visibility for keyboard users
- WCAG AA compliant out of the box
- No custom CSS complexity
- Works across all browsers without vendor prefixes

**Implementation Details:**
- CSS: `:focus { outline: 2px solid var(--accent); outline-offset: 2px; }`
- Accent color (#e8ff47) provides high contrast on dark background
- `outline-offset` prevents outline from overlapping element border
- Remove default browser outline: `:focus:not(:focus-visible) { outline: none; }`
- Apply to all interactive elements: buttons, chips, search input, command rows

**Elements Requiring Focus Styles:**
- Search input
- Category filter chips
- Runtime filter chips
- Command rows (when focused via keyboard)
- Copy buttons
- Sidebar navigation items
- Wizard options
- Settings modal buttons

**Out of Scope:**
- Custom focus animations
- Per-element focus colors
- Focus indicators for mouse users (only keyboard)

---

### DECISION-08: Search Debouncing

**Decision:** 300ms delay (industry standard balanced approach).

**Rationale:**
- 200ms feels too fast (re-renders on every other keystroke)
- 400-500ms feels laggy (noticeable delay before results update)
- 300ms is the sweet spot used by Google, GitHub, VS Code
- Prevents excessive re-renders while maintaining responsive feel

**Implementation Details:**
```javascript
let debounceTimer = null

searchInput.addEventListener('input', (e) => {
  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => {
    searchVal = e.target.value
    renderCommands()
  }, 300)
})
```
- Cancel previous timer on new keystroke
- Immediate render on first keystroke (no delay for initial feedback)
- Debounce only applies to search input, not filter changes
- Filter chip clicks trigger immediate re-render (no debounce)

**Performance Optimization:**
- If command list exceeds 100 items, increase debounce to 400ms
- Use `requestAnimationFrame` for render if frame drops detected
- Future: virtual scrolling for >500 commands (Phase 5)

**Out of Scope:**
- Typing speed detection (adaptive debounce)
- Web Worker for search computation
- Caching previous search results

---

## Gray Areas (Already Discussed & Decided)

| Gray Area | Decision |
|-----------|----------|
| Runtime filter: how many to show? | User-configurable via Settings modal |
| Multi-select runtime UI pattern? | Hybrid: 4 popular as chips + "More" dropdown |
| Category + runtime combination logic? | Toggleable AND/OR (default: AND) |
| Search ranking algorithm? | Weighted: name 50%, short 30%, flags 15%, when 5% |
| Keyboard navigation scope? | Visible (filtered) commands only |
| Escape key behavior? | Smart sequential (close → clear → blur) |
| Focus indicator style? | Standard outline ring with accent color |
| Search debounce delay? | 300ms balanced |

---

## Constraints (From PROJECT.md)

### Technical Constraints
- **Single-file portability:** Must remain runnable as `file://` without a server
- **Zero build step:** No bundlers, transpilers, or npm install required
- **No backend:** Purely client-side, no API server, no database
- **Browser support:** Modern browser with ES6+ (Chrome 61+, Firefox 54+, Safari 11+)

### Design Constraints
- **Dark theme only** (for now)
- **Satoshi font** via Fontshare CDN (or system font fallback)
- **Mobile-responsive** — must work on tablets and phones

### Process Constraints
- **Solo developer workflow** — no team coordination
- **Agent-assisted implementation** — user is visionary, agent is builder

---

## Success Criteria (From ROADMAP.md)

Phase 1 is complete when ALL of the following are true:

1. ✅ User can filter commands by runtime (e.g., "show only Claude Code commands")
2. ✅ User can combine category + runtime filters with AND/OR toggle
3. ✅ User can navigate command list using arrow keys (up/down)
4. ✅ User can expand/collapse command with Enter key
5. ✅ User can clear search with Escape key (smart sequential: close → clear → blur)
6. ✅ User can configure which runtimes to display in Settings modal
7. ✅ Search uses weighted ranking (command name prioritized)
8. ✅ Search has 300ms debouncing
9. ✅ All interactive elements have visible focus rings
10. ✅ Settings modal persists runtime preferences in localStorage

---

## Dependencies

**Phase 1 depends on:** Nothing (first phase)

**Future phases depend on Phase 1:**
- Phase 2 (State & Persistence) — builds on Phase 1 filter state management
- Phase 3 (Export & Sharing) — needs Phase 1 search/filter to export current view
- Phase 5 (Performance & Polish) — improves Phase 1 search with virtual scrolling

---

## Risks & Mitigations

### Risk: Runtime preferences complexity
**Likelihood:** Medium  
**Impact:** Medium  
**Mitigation:** Start with simple "Select All / Clear All" buttons, add granular controls in Phase 2 if needed.

### Risk: AND/OR toggle confusion
**Likelihood:** Low  
**Impact:** Medium  
**Mitigation:** Clear labeling ("Match ALL" vs "Match ANY"), tooltip explanation, default to AND.

### Risk: Weighted search scoring bugs
**Likelihood:** Medium  
**Impact:** Low  
**Mitigation:** Test with known queries (e.g., "plan" should rank plan-phase #1), add console logging for debugging.

### Risk: Keyboard navigation conflicts
**Likelihood:** Low  
**Impact:** Low  
**Mitigation:** Test with screen reader, ensure Tab/Shift+Tab work independently of arrow keys.

---

## Files to Create/Modify

### New Files
- None (Phase 1 enhances existing `gsd.html`)

### Modified Files
- `gsd.html` — Add:
  - Settings modal HTML structure
  - Gear icon in top bar
  - Runtime filter chips with "More" dropdown
  - AND/OR toggle switch
  - Keyboard navigation event listeners
  - Smart sequential Escape handler
  - Weighted search scoring in `getFiltered()`
  - Debounced search input handler
  - Focus ring CSS
  - localStorage persistence for runtime preferences

### Future Files (Phase 4+)
- `app.js` — Extract JavaScript (Phase 4)
- `styles.css` — Extract CSS (Phase 4)
- `*.test.js` — Unit tests (Phase 4)

---

## Out of Scope (Explicitly Deferred)

The following are **NOT** in Phase 1:

- ❌ Command history tracking (Phase 2)
- ❌ Favorites system (Phase 2)
- ❌ Export to Markdown/PDF (Phase 3)
- ❌ Shareable URLs (Phase 3)
- ❌ Code splitting into separate files (Phase 4)
- ❌ ESLint/Prettier setup (Phase 4)
- ❌ Unit tests (Phase 4)
- ❌ Light/dark theme toggle (Phase 5)
- ❌ Virtual scrolling (Phase 5)
- ❌ Accessibility audit beyond focus rings (Phase 5)
- ❌ Analytics or search tracking
- ❌ Fuzzy matching beyond 1-character typo
- ❌ Search synonyms or stemming
- ❌ Filter presets or saved combinations

---

## Next Steps

1. **Review this CONTEXT.md** — User confirms decisions are correct
2. **Run `/gsd-plan-phase 1`** — AI generates detailed implementation plan with tasks
3. **Plan verification** — User reviews plan for accuracy
4. **Execute Phase 1** — AI implements features task by task
5. **Verify success criteria** — User tests all 10 criteria manually

---

*1-CONTEXT.md created: 2026-04-19*
*Decisions captured via structured discussion with user*
