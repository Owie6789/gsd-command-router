# Phase 2 Context: State & Persistence

**Phase Number:** 2
**Created:** 2026-04-19
**Status:** Ready for Planning

---

## Phase Goal

User preferences and activity persist across browser sessions.

---

## Prior Context (From Phase 1)

These decisions from Phase 1 are LOCKED and must be respected:

### DECISION-01: Runtime Filter Configuration ✅
- Settings modal with gear icon
- localStorage key: `gsd:runtimePreferences`
- Already implemented — Phase 2 extends this pattern

### DECISION-02: Theme Color System ✅
- Dynamic RGBA calculation for accent-dim/accent-dim2
- applyThemeColor() helper function
- Settings apply on Save button click
- localStorage key: `gsd:accentColor`

### DECISION-03: Font Preferences ✅
- UI font with system fallbacks
- Code font separate from UI font
- localStorage keys: `gsd:uiFont`, `gsd:codeFont`

---

## Phase 2 Gray Areas (To Be Decided)

### GRAY-01: Command History Storage

**Question:** How many recently viewed commands to track?

**Options:**
1. **Last 10** — Lightweight, simple array, no UI scrolling needed
2. **Last 25** — Balanced, might need scrollable dropdown
3. **Last 50+** — Full history, needs pagination/search UI

**Recommendation:** Start with **Last 10** — can expand later. localStorage has ~5MB limit, and 10 commands is sufficient for most workflows.

**Implementation Pattern:**
```javascript
// Array of command IDs, newest first
const history = ['/gsd-new-project', '/gsd-plan-phase', ...];
localStorage.setItem('gsd:history', JSON.stringify(history.slice(0, 10)));
```

---

### GRAY-02: Favorites System UI

**Question:** How do users mark commands as favorites?

**Options:**
1. **Star icon in command row** — Always visible, one click
2. **Right-click context menu** — Cleaner UI, less discoverable
3. **Keyboard shortcut** — Press 'f' when focused, power user feature

**Recommendation:** **Star icon in command row** — Most discoverable. Can add keyboard shortcut as enhancement later.

**Implementation Pattern:**
```javascript
// Add star button to .cmd-meta
const starBtn = document.createElement('button');
starBtn.className = 'favorite-btn';
starBtn.innerHTML = isFavorite ? '★' : '☆';
starBtn.addEventListener('click', () => toggleFavorite(cmd.id));
```

---

### GRAY-03: History & Favorites UI Location

**Question:** Where to display history and favorites?

**Options:**
1. **Sidebar section** — Under categories, always visible
2. **Dropdown in top bar** — Clock/star icon, compact
3. **Separate tab/view** — Like Cheatsheet, dedicated space

**Recommendation:** **Sidebar section** under categories — Most convenient for frequent access. Can collapse if needed.

**HTML Structure:**
```html
<div class="sidebar-section">
  <div class="sidebar-label">History</div>
  <div id="history-list">
    <!-- Recent commands -->
  </div>
</div>
<div class="sidebar-section">
  <div class="sidebar-label">Favorites</div>
  <div id="favorites-list">
    <!-- Favorited commands -->
  </div>
</div>
```

---

### GRAY-04: State Persistence Scope

**Question:** What state should persist across reloads?

**Already Persisted (Phase 1):**
- ✅ Runtime preferences
- ✅ Theme color
- ✅ UI font
- ✅ Code font

**Phase 2 Additions:**
- [ ] Last active view (Commands/Wizard/Flow/etc.)
- [ ] Search query
- [ ] Active category filter
- [ ] Active runtime filters
- [ ] Command history (last N viewed)
- [ ] Favorites list

**Recommendation:** Persist ALL of the above. Users expect their working state to be preserved.

**localStorage Keys:**
```javascript
'gsd:lastView'        // e.g., 'commands', 'wizard'
'gsd:searchQuery'     // e.g., '/gsd-plan'
'gsd:activeFilter'    // e.g., 'core', 'all'
'gsd:runtimeFilter'   // e.g., ['Claude Code', 'Cursor']
'gsd:history'         // e.g., ['/gsd-new-project', ...]
'gsd:favorites'       // e.g., ['/gsd-plan-phase', '/gsd-ship']
```

---

### GRAY-05: History Tracking Trigger

**Question:** When do we add a command to history?

**Options:**
1. **On command expand** — When user clicks to view details
2. **On copy usage** — When user clicks "Copy" button
3. **On navigate to related** — When user clicks related command link

**Recommendation:** **On command expand** — Clear user intent to view command. Copy and navigate are secondary actions.

**Implementation:**
```javascript
function trackHistory(cmdId) {
  let history = loadHistory();
  // Remove if already exists (avoid duplicates)
  history = history.filter(id => id !== cmdId);
  // Add to front
  history.unshift(cmdId);
  // Keep only last 10
  history = history.slice(0, 10);
  saveHistory(history);
}

// Call in cmd-row click handler
cmdRow.addEventListener('click', () => {
  trackHistory(cmd.id);
  // ... expand logic
});
```

---

### GRAY-06: Clear Data Controls

**Question:** How do users clear history/favorites?

**Options:**
1. **Individual clear buttons** — Next to each section in sidebar
2. **Settings modal** — "Clear History" and "Clear Favorites" buttons
3. **Both** — Quick clear in sidebar, full control in Settings

**Recommendation:** **Settings modal** — Keeps UI clean. Add confirmation dialog to prevent accidental deletion.

**Implementation:**
```html
<!-- In Settings modal -->
<div class="settings-section">
  <h3>Data Management</h3>
  <button class="modal-btn" id="clear-history">Clear History</button>
  <button class="modal-btn" id="clear-favorites">Clear Favorites</button>
</div>
```

---

## Implementation Decisions (Locked for Phase 2)

### DECISION-01: History Size
**Decision:** Track last **10 commands**

**Rationale:**
- Lightweight storage
- Sufficient for most workflows
- Can expand to 25+ in future phase if needed

**Implementation:**
- Array of command IDs (strings)
- Newest first (unshift to add)
- Slice to 10 after each addition
- localStorage key: `gsd:history`

---

### DECISION-02: Favorites UI
**Decision:** Star icon in command row

**Rationale:**
- Most discoverable pattern
- One-click toggle
- Consistent with common patterns (GitHub, Gmail, etc.)

**Implementation:**
- Star button (★/☆) in `.cmd-meta` section
- Filled star for favorited, outline for not favorited
- Click to toggle
- Add to favorites array or remove
- localStorage key: `gsd:favorites`

---

### DECISION-03: History & Favorites Location
**Decision:** Sidebar sections under categories

**Rationale:**
- Always visible — no extra clicks
- Natural extension of existing sidebar
- Easy to access while browsing commands

**Implementation:**
- Two new `.sidebar-section` divs
- History section first (more frequently used)
- Favorites section second
- Click item → navigate to command
- Empty state message if no history/favorites

---

### DECISION-04: State Persistence
**Decision:** Persist ALL working state

**What Persists:**
- Last active view
- Search query
- Active category filter
- Active runtime filters
- Command history (last 10)
- Favorites list

**Implementation:**
- Save state on every change (debounced if rapid)
- Restore state on page load (after DOM ready)
- Use `gsd:` prefix for all localStorage keys
- Wrap in try/catch for private browsing mode

---

### DECISION-05: History Tracking
**Decision:** Track on command expand

**Rationale:**
- Clear user intent to view
- Not triggered by accidental clicks
- Doesn't track passive actions (copy, navigate)

**Implementation:**
- Call `trackHistory(cmdId)` in cmd-row click handler
- Deduplicate (remove existing before adding)
- Add to front of array
- Limit to 10 items

---

### DECISION-06: Clear Data
**Decision:** Settings modal with confirmation

**Rationale:**
- Keeps sidebar clean
- Confirmation prevents accidents
- Centralized data management

**Implementation:**
- "Data Management" section in Settings
- "Clear History" button → confirmation → clear
- "Clear Favorites" button → confirmation → clear
- Toast notification on success

---

## Success Criteria (From ROADMAP.md)

Phase 2 is complete when ALL of the following are true:

1. ✅ User reloads page and returns to last active view automatically
2. ✅ User reloads page and search query + filters are restored
3. ✅ User can view list of recently viewed commands (last 10)
4. ✅ User can mark a command as favorite (star icon)
5. ✅ User can filter to show only favorited commands
6. ✅ User can clear history and clear favorites

---

## Files to Modify

**Single File:** `gsd.html`

**Sections to Add/Modify:**
1. **CSS:**
   - `.favorite-btn` styles (star button)
   - `.history-list`, `.favorites-list` styles
   - `.sidebar-section` updates for new sections

2. **HTML:**
   - Star button in `.cmd-meta` template
   - History section in sidebar
   - Favorites section in sidebar
   - Data management section in Settings modal

3. **JavaScript:**
   - `loadHistory()`, `saveHistory()` functions
   - `loadFavorites()`, `saveFavorites()` functions
   - `trackHistory(cmdId)` function
   - `toggleFavorite(cmdId)` function
   - `renderHistory()` function
   - `renderFavorites()` function
   - State persistence on load/save
   - Clear history/favorites handlers

---

## Out of Scope (Explicitly Deferred)

The following are **NOT** in Phase 2:

- ❌ Command usage analytics (most viewed, etc.)
- ❌ Export/import favorites as JSON
- ❌ Search within history
- ❌ History grouping by session
- ❌ Favorite folders/categories
- ❌ Sync across devices
- ❌ Command ratings (stars 1-5)
- ❌ Notes/annotations on commands

These may be added in Phase 3+ or as separate enhancements.

---

## Dependencies

**Phase 2 depends on:** Phase 1 ✅ COMPLETE

**Future phases depend on Phase 2:**
- Phase 3 (Export & Sharing) — May export favorites/history
- Phase 5 (Performance) — May optimize localStorage operations

---

## Risks & Mitigations

### Risk: localStorage quota exceeded
**Likelihood:** Low (5MB limit, text data is small)
**Impact:** Medium (state not saved)
**Mitigation:** Wrap all localStorage in try/catch, fail gracefully

### Risk: Private browsing mode
**Likelihood:** Medium (some users use private/incognito)
**Impact:** Low (state just doesn't persist)
**Mitigation:** Check if localStorage available, degrade gracefully

### Risk: Sidebar clutter
**Likelihood:** Medium (adding 2 new sections)
**Impact:** Low (can collapse/hide)
**Mitigation:** Keep sections compact, consider collapsible

---

## Next Steps

1. **Review this CONTEXT.md** — User confirms decisions are correct
2. **Run `/gsd-plan-phase 2`** — AI generates detailed implementation plan with tasks
3. **Plan verification** — User reviews plan for accuracy
4. **Execute Phase 2** — AI implements features task by task
5. **Verify success criteria** — User tests all 6 criteria manually

---

*2-CONTEXT.md created: 2026-04-19*
*Decisions captured via structured discussion*
*Ready for Phase 2 planning*
