# Phase 2 Plan: State & Persistence

**Phase Number:** 2
**Created:** 2026-04-19
**Status:** Ready for Execution

---

## Overview

**Goal:** User preferences and activity persist across browser sessions.

**Success Criteria:** 6 items (see ROADMAP.md)

**Locked Decisions:** 6 items (see 2-CONTEXT.md)

**Files to Modify:** `gsd.html`

**Estimated Effort:** 8 tasks × ~1.5 hours = 12 hours

---

## Tasks

### TASK-01: State Persistence Infrastructure

**Dependencies:** None

**Acceptance Criteria:**
- [ ] `loadState()` function reads all persisted state from localStorage
- [ ] `saveState()` function persists all state to localStorage
- [ ] All localStorage operations wrapped in try/catch
- [ ] Graceful degradation if localStorage unavailable (private browsing)
- [ ] State restored on page load (after DOM ready)

**Implementation Notes:**
```javascript
// State object structure
const appState = {
  lastView: 'commands',
  searchQuery: '',
  activeFilter: 'all',
  runtimeFilter: [],
  history: [],
  favorites: []
};

// Load state on init
function loadState() {
  try {
    return {
      lastView: localStorage.getItem('gsd:lastView') || 'commands',
      searchQuery: localStorage.getItem('gsd:searchQuery') || '',
      activeFilter: localStorage.getItem('gsd:activeFilter') || 'all',
      runtimeFilter: JSON.parse(localStorage.getItem('gsd:runtimeFilter') || '[]'),
      history: JSON.parse(localStorage.getItem('gsd:history') || '[]'),
      favorites: JSON.parse(localStorage.getItem('gsd:favorites') || '[]')
    };
  } catch (e) {
    console.warn('localStorage unavailable, using defaults');
    return { ...defaultState };
  }
}

// Save state (debounced)
function saveState() {
  try {
    localStorage.setItem('gsd:lastView', appState.lastView);
    localStorage.setItem('gsd:searchQuery', appState.searchQuery);
    // ... etc
  } catch (e) {
    console.warn('Failed to save state:', e);
  }
}
```

**Verification:**
- Open page → make changes → reload → state restored
- Open private browsing → no errors, state just doesn't persist

---

### TASK-02: Last View Persistence

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] Switching view (Commands/Wizard/Flow/etc.) saves to `gsd:lastView`
- [ ] On page load, restore last active view
- [ ] Sidebar active state matches restored view
- [ ] View content rendered for restored view

**Implementation Notes:**
```javascript
// In switchView() function
function switchView(id) {
  // ... existing code ...
  appState.lastView = id;
  saveState();
}

// In INIT section
document.addEventListener('DOMContentLoaded', () => {
  const state = loadState();
  if (state.lastView && state.lastView !== 'commands') {
    switchView(state.lastView);
  }
});
```

**Verification:**
- Click "Wizard" tab → reload page → Wizard tab active
- Click "Cheatsheet" tab → reload → Cheatsheet active

---

### TASK-03: Search & Filter Persistence

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] Search query saves to `gsd:searchQuery`
- [ ] Active category filter saves to `gsd:activeFilter`
- [ ] Active runtime filters save to `gsd:runtimeFilter`
- [ ] On page load, restore all search/filter state
- [ ] Search input shows restored query
- [ ] Filter chips show restored active state

**Implementation Notes:**
```javascript
// In search input handler
document.getElementById('search').addEventListener('input', (e) => {
  clearTimeout(searchDebounceTimer);
  searchDebounceTimer = setTimeout(() => {
    searchVal = e.target.value;
    appState.searchQuery = searchVal;
    saveState();
    renderCommands();
  }, 300);
});

// In filter chip handler
document.getElementById('filter-row').addEventListener('click', e => {
  // ... existing code ...
  appState.activeFilter = activeFilter;
  saveState();
});

// Restore on load
const state = loadState();
if (state.searchQuery) {
  document.getElementById('search').value = state.searchQuery;
  searchVal = state.searchQuery;
}
if (state.activeFilter && state.activeFilter !== 'all') {
  // Activate correct filter chip
}
```

**Verification:**
- Type "plan" in search → reload → "plan" still in search box
- Click "Core" filter → reload → Core filter active
- Select runtime filters → reload → same runtimes selected

---

### TASK-04: Command History Tracking

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] `trackHistory(cmdId)` adds command to history array
- [ ] History limited to last 10 commands (newest first)
- [ ] Deduplication (remove existing before adding to front)
- [ ] History tracked on command expand (click)
- [ ] History sidebar section displays last 10 commands
- [ ] Click history item → navigate to command
- [ ] Empty state message if no history

**Implementation Notes:**
```javascript
// History functions
function trackHistory(cmdId) {
  // Remove if exists (deduplicate)
  appState.history = appState.history.filter(id => id !== cmdId);
  // Add to front
  appState.history.unshift(cmdId);
  // Limit to 10
  appState.history = appState.history.slice(0, 10);
  saveState();
}

function loadHistory() {
  return appState.history;
}

function renderHistory() {
  const container = document.getElementById('history-list');
  if (appState.history.length === 0) {
    container.innerHTML = '<p style="color:var(--t2);font-size:12px">No recent commands</p>';
    return;
  }
  container.innerHTML = appState.history.map(cmdId => {
    const cmd = CMDS.find(c => c.id === cmdId);
    return cmd ? `
      <div class="sidebar-item history-item" data-cmd="${cmdId}">
        <span class="sidebar-dot"></span>
        <code>${escHtml(cmd.cmd)}</code>
      </div>
    ` : '';
  }).join('');
  
  // Add click handlers
  container.querySelectorAll('.history-item').forEach(item => {
    item.addEventListener('click', () => {
      const cmdId = item.dataset.cmd;
      openCmd = cmdId;
      switchView('commands');
      renderCommands();
      // Scroll to command
    });
  });
}

// In cmd-row click handler
cmdRow.addEventListener('click', () => {
  trackHistory(cmd.id);
  // ... expand logic
});
```

**Verification:**
- Expand 5 different commands → reload → all 5 in history sidebar
- Click 11th command → oldest removed from history
- Click history item → navigates to that command

---

### TASK-05: Favorites System

**Dependencies:** TASK-01

**Acceptance Criteria:**
- [ ] Star button (★/☆) in `.cmd-meta` section of each command
- [ ] Click star → toggle favorite status
- [ ] Filled star (★) for favorited, outline (☆) for not favorited
- [ ] Favorites saved to `gsd:favorites`
- [ ] Favorites sidebar section displays all favorited commands
- [ ] Click favorite item → navigate to command
- [ ] Empty state message if no favorites

**Implementation Notes:**
```javascript
// Add star button to cmd-row template
const starBtn = document.createElement('button');
starBtn.className = 'favorite-btn';
starBtn.innerHTML = isFavorite ? '★' : '☆';
starBtn.title = isFavorite ? 'Remove from favorites' : 'Add to favorites';
starBtn.addEventListener('click', (e) => {
  e.stopPropagation(); // Don't expand/collapse
  toggleFavorite(cmd.id);
});
cmdMeta.appendChild(starBtn);

// Favorite functions
function toggleFavorite(cmdId) {
  const index = appState.favorites.indexOf(cmdId);
  if (index > -1) {
    // Remove
    appState.favorites.splice(index, 1);
  } else {
    // Add
    appState.favorites.push(cmdId);
  }
  saveState();
  renderCommands(); // Re-render to update star icons
  renderFavorites();
}

function renderFavorites() {
  const container = document.getElementById('favorites-list');
  if (appState.favorites.length === 0) {
    container.innerHTML = '<p style="color:var(--t2);font-size:12px">No favorites yet</p>';
    return;
  }
  // Similar to renderHistory()
}

// In renderCommands(), check if command is favorited
const isFavorite = appState.favorites.includes(cmd.id);
starBtn.innerHTML = isFavorite ? '★' : '☆';
```

**Verification:**
- Click star on command → star becomes filled ★
- Reload page → star still filled
- Click filled star → becomes outline ☆
- Favorited commands appear in Favorites sidebar

---

### TASK-06: Favorites Filter

**Dependencies:** TASK-05

**Acceptance Criteria:**
- [ ] "Favorites" filter chip in filter row
- [ ] Click Favorites chip → show only favorited commands
- [ ] Can combine with search query
- [ ] Active state visible (like other filter chips)
- [ ] Exit favorites filter → return to normal view

**Implementation Notes:**
```javascript
// Add Favorites chip to filter row
<button class="chip" data-cat="favorites">
  ★ Favorites
</button>

// In getFiltered()
function getFiltered() {
  return CMDS.filter(cmd => {
    // Category filter
    if (activeFilter === 'favorites') {
      if (!appState.favorites.includes(cmd.id)) return false;
    } else if (activeFilter !== 'all' && cmd.cat !== activeFilter) {
      return false;
    }
    
    // Runtime filter
    // ... existing logic ...
    
    // Search scoring
    // ... existing logic ...
    
    return true;
  });
}
```

**Verification:**
- Click "★ Favorites" chip → only favorited commands shown
- Type in search while Favorites active → filters within favorites
- Click "All" → all commands shown again

---

### TASK-07: Clear Data Controls

**Dependencies:** TASK-04, TASK-05

**Acceptance Criteria:**
- [ ] "Data Management" section in Settings modal
- [ ] "Clear History" button with confirmation
- [ ] "Clear Favorites" button with confirmation
- [ ] Confirmation dialog prevents accidents
- [ ] Clear success shows toast notification
- [ ] History/favorites lists update immediately after clear

**Implementation Notes:**
```html
<!-- In Settings modal -->
<div class="settings-section" style="margin-top:24px;padding-top:24px;border-top:1px solid var(--line)">
  <h3 style="font-size:14px;font-weight:700;margin-bottom:16px">Data Management</h3>
  <div style="display:flex;gap:8px">
    <button class="modal-btn" id="clear-history">Clear History</button>
    <button class="modal-btn" id="clear-favorites">Clear Favorites</button>
  </div>
</div>
```

```javascript
// Clear handlers
document.getElementById('clear-history')?.addEventListener('click', () => {
  if (confirm('Clear all command history? This cannot be undone.')) {
    appState.history = [];
    localStorage.removeItem('gsd:history');
    renderHistory();
    showToast('History cleared');
  }
});

document.getElementById('clear-favorites')?.addEventListener('click', () => {
  if (confirm('Clear all favorites? This cannot be undone.')) {
    appState.favorites = [];
    localStorage.removeItem('gsd:favorites');
    renderFavorites();
    renderCommands(); // Update star icons
    showToast('Favorites cleared');
  }
});
```

**Verification:**
- Settings → Clear History → Confirm → history empty, toast shown
- Settings → Clear Favorites → Confirm → favorites empty, stars reset

---

### TASK-08: Integration Testing & Polish

**Dependencies:** TASK-02 through TASK-07

**Acceptance Criteria:**
- [ ] All 6 success criteria pass manual testing
- [ ] No console errors in browser dev tools
- [ ] State persists correctly across reloads
- [ ] Private browsing mode degrades gracefully
- [ ] All UI elements properly styled
- [ ] Keyboard navigation works with new elements

**Implementation Notes:**
- Test in Chrome, Firefox, Safari if possible
- Test in private/incognito mode
- Check localStorage in DevTools → Application → Local Storage
- Verify no memory leaks (history doesn't grow unbounded)

**Verification Protocol:**
See "Verification Protocol" section below.

---

## Verification Protocol

**Execute after all tasks complete. All 6 criteria MUST pass.**

### Criterion 1: Restore Last View
1. Click "Wizard" tab
2. Reload page
3. **Expected:** Wizard tab still active

### Criterion 2: Restore Search & Filters
1. Type "plan" in search
2. Click "Core" category filter
3. Select "Claude Code" runtime filter
4. Reload page
5. **Expected:** Search shows "plan", Core filter active, Claude Code selected

### Criterion 3: Command History
1. Expand 5 different commands (click to view details)
2. Reload page
3. **Expected:** All 5 commands visible in History sidebar section
4. Click 11th command
5. **Expected:** Oldest command removed from history (only 10 kept)

### Criterion 4: Favorites System
1. Click star on 3 commands
2. Stars become filled (★)
3. Reload page
4. **Expected:** Same 3 stars still filled
5. Click filled star
6. **Expected:** Star becomes outline (☆), removed from favorites

### Criterion 5: Favorites Filter
1. Click "★ Favorites" filter chip
2. **Expected:** Only favorited commands shown
3. Type in search box
4. **Expected:** Filters within favorites only
5. Click "All" chip
6. **Expected:** All commands shown again

### Criterion 6: Clear Data
1. Settings → Clear History → Confirm
2. **Expected:** History empty, toast notification
3. Settings → Clear Favorites → Confirm
4. **Expected:** Favorites empty, all stars outline, toast notification

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| localStorage quota exceeded | Wrap all operations in try/catch, 5MB limit is plenty for text data |
| Private browsing mode | Graceful degradation - state just doesn't persist, no errors |
| History grows unbounded | Slice to 10 after every addition |
| State corruption | JSON.parse in try/catch, reset to defaults on error |
| UI clutter in sidebar | Keep sections compact, use 12px font, truncate long command names |

---

## Out of Scope (Explicitly Deferred)

- ❌ Command usage analytics (most viewed, etc.)
- ❌ Export/import favorites as JSON
- ❌ Search within history
- ❌ History grouping by session
- ❌ Favorite folders/categories
- ❌ Sync across devices
- ❌ Command ratings (stars 1-5)
- ❌ Notes/annotations on commands

---

*2-PLAN.md created: 2026-04-19*
*Ready for execution via /gsd-execute-phase 2*
