# Phase 5 Context: Performance & Polish

**Phase Number:** 5
**Created:** 2026-04-19
**Status:** Ready for Planning

---

## Phase Goal

Application feels responsive and accessible. Visual polish to prevent flat appearance.

---

## Prior Context (From Previous Phases)

These decisions from previous phases are LOCKED and must be respected:

### From Phase 1:
- ✅ Search debouncing at 300ms — DO NOT MODIFY (working correctly)
- ✅ Keyboard navigation (arrow keys, Enter, Escape) — VERIFY & FIX if broken
- ✅ Focus rings with accent color — VERIFY & FIX if broken

### From Phase 1.1:
- ✅ Theme color system (dynamic accent colors) — DO NOT ADD LIGHT THEME
- ✅ Font preferences (UI + code fonts) — Already implemented

### From Phase 2:
- ✅ State persistence (localStorage) — Already implemented
- ✅ History & favorites — Already implemented

### From Phase 4:
- ✅ AI Maintenance Guide — All changes must follow documented conventions

---

## Implementation Decisions (Locked for Phase 5)

### DECISION-01: Virtual Scrolling Infrastructure

**Decision:** Add virtual scrolling preparation/pattern, but don't fully implement yet.

**Rationale:**
- Current command count (~50) doesn't need virtual scrolling
- When commands grow to 100+, performance will degrade
- Add the infrastructure now so it's easy to activate later
- Don't over-engineer for a problem we don't have yet

**Implementation Details:**
- Add comment in `renderCommands()` explaining when to activate
- Add `VIRTUAL_SCROLL_THRESHOLD = 100` constant
- Add placeholder function `renderVirtualizedCommands()` with TODO
- Document in AI Maintenance Guide how to activate

**Files to Modify:**
- `gsd.html` — Add constants, placeholder function, comments

**Out of Scope:**
- Full virtual scrolling implementation
- Windowing libraries
- Complex scroll position tracking

---

### DECISION-02: Accessibility Audit & Fixes

**Decision:** Conduct thorough accessibility audit with ACTUAL TESTING, not assumptions. Fix all verified issues.

**Rationale:**
- User reported "accessibility for different pages seems kinda broken"
- Need to verify with facts, not what "looks good"
- Test actual keyboard navigation across ALL views
- Test screen reader compatibility
- Test focus management

**Audit Areas:**
1. **Keyboard Navigation** — Test every view (Commands, Wizard, Flow, Cheatsheet, Concepts, FAQ, Compare, Quick Start)
   - Tab order logical?
   - All interactive elements focusable?
   - Focus visible on all elements?
   - Enter/Space activate buttons?
   - Escape closes modals/clears search?

2. **Screen Reader** — Test with NVDA (Windows) or VoiceOver (Mac)
   - All buttons have aria-labels?
   - All images have alt text?
   - All form inputs have labels?
   - Landmark regions defined?

3. **Focus Management**
   - Focus trapped in modals?
   - Focus restored after modal closes?
   - Focus visible on all interactive elements?
   - Skip-to-content link?

4. **Color Contrast** — Test with WebAIM Contrast Checker
   - All text meets WCAG AA (4.5:1 for normal, 3:1 for large)?
   - All interactive elements have sufficient contrast?

**Implementation Approach:**
1. Manual audit of each view (checklist below)
2. Document all issues found
3. Fix issues systematically
4. Re-test to verify fixes

**Audit Checklist:**
```markdown
## Commands View
- [ ] Search input has label
- [ ] Filter chips are keyboard navigable
- [ ] Command rows are focusable
- [ ] Expand icon visible on focus
- [ ] Favorite button has aria-label
- [ ] Copy button has aria-label
- [ ] Related commands are keyboard navigable

## Wizard View
- [ ] Wizard options are keyboard navigable
- [ ] Back button focusable
- [ ] Progress indicators have aria-labels

## Settings Modal
- [ ] Modal has role="dialog"
- [ ] Modal has aria-modal="true"
- [ ] Modal has aria-labelledby
- [ ] Focus trapped inside modal
- [ ] Focus restored when modal closes
- [ ] All form inputs have labels
- [ ] Color swatches have aria-labels

## Sidebar
- [ ] Navigation items are keyboard navigable
- [ ] Active state announced by screen reader
- [ ] Section headers have proper heading level

## All Views
- [ ] Skip-to-content link
- [ ] Landmark regions (<main>, <nav>, <header>)
- [ ] Page has <h1>
- [ ] Heading hierarchy is logical (h1 → h2 → h3)
```

**Files to Modify:**
- `gsd.html` — Fix all verified issues

**Out of Scope:**
- Full WCAG AAA compliance
- Automated testing infrastructure (Phase 3 skipped)
- Screen reader support documentation

---

### DECISION-03: Background Grid Pattern

**Decision:** Add subtle grid/white pattern to website background edges so it doesn't look too flat.

**Rationale:**
- Current solid dark background (#080808) looks flat
- Subtle pattern adds visual depth without distraction
- Grid pattern matches technical/documentation aesthetic
- Edges only — center content area stays clean

**Implementation Details:**
- CSS gradient or SVG pattern
- Very subtle (low opacity, ~5-10%)
- Grid lines or dot pattern
- Applied to edges/corners, not center
- Respects dark theme (white/light lines on dark)

**CSS Approach:**
```css
/* Option 1: CSS Grid Gradient */
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
  background-size: 20px 20px;
}

/* Option 2: Edge-only vignette with subtle pattern */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: radial-gradient(
    ellipse at center,
    transparent 0%,
    transparent 60%,
    rgba(255,255,255,0.02) 100%
  );
  pointer-events: none;
}
```

**Files to Modify:**
- `gsd.html` — Add CSS pattern to body or root styles

**Out of Scope:**
- Animated backgrounds
- Complex illustrations
- Changing existing color scheme

---

## Success Criteria (From ROADMAP.md — Updated)

Phase 5 is complete when ALL of the following are true:

1. ✅ Virtual scrolling infrastructure added (constants, placeholder, docs)
2. ✅ Accessibility audit completed with documented findings
3. ✅ All verified accessibility issues fixed
4. ✅ Keyboard navigation works on ALL views (tested manually)
5. ✅ Background grid pattern visible but subtle
6. ✅ No breaking changes to existing features

**Removed Criteria (already done in Phase 1):**
- ~~Search input has debouncing~~ — DONE in Phase 1
- ~~Keyboard navigation exists~~ — DONE in Phase 1

**Removed Criteria (skipped by user request):**
- ~~Light/dark theme toggle~~ — User wants dark-only

---

## Dependencies

**Phase 5 depends on:**
- ✅ Phase 1 (search/filter/keyboard nav) — COMPLETE
- ✅ Phase 1.1 (UI polish) — COMPLETE
- ✅ Phase 2 (state persistence) — COMPLETE
- ✅ Phase 4 (AI Maintenance Guide) — COMPLETE

**Future phases depend on Phase 5:**
- None identified — Phase 5 is final polish phase

---

## Files to Modify

**Single File:** `gsd.html`

**Sections to Modify:**
1. **CSS** — Add background grid pattern
2. **JavaScript** — Add virtual scrolling constants/placeholders
3. **HTML** — Add missing ARIA attributes, fix focus management
4. **AI Maintenance Guide** — Update with accessibility patterns

---

## Risks & Mitigations

### Risk: Accessibility fixes break existing features
**Likelihood:** Medium  
**Impact:** Medium  
**Mitigation:** Test each fix individually, keep git history clean for easy rollback

### Risk: Over-engineering virtual scrolling
**Likelihood:** Low  
**Impact:** Low  
**Mitigation:** Add constants and placeholder only, document "activate when 100+ commands"

### Risk: Background pattern too distracting
**Likelihood:** Low  
**Impact:** Low  
**Mitigation:** Start with very low opacity (3-5%), increase only if barely visible

### Risk: Accessibility audit scope creep
**Likelihood:** Medium  
**Impact:** Medium  
**Mitigation:** Stick to verified issues only, document "nice-to-haves" for future phases

---

## Out of Scope (Explicitly Deferred)

The following are **NOT** in Phase 5:

- ❌ Light theme (user explicitly said no)
- ❌ Full virtual scrolling implementation (just prep)
- ❌ WCAG AAA compliance (AA is sufficient)
- ❌ Automated accessibility testing (no test infrastructure)
- ❌ Performance profiling/optimization (debouncing already done)
- ❌ Animation/transition polish
- ❌ Mobile-specific accessibility (focus on desktop first)

---

## Testing Protocol

**Manual Testing Required:**

### Keyboard Navigation Test
1. Unplug mouse (or ignore it)
2. Navigate entire site using only Tab, Enter, Escape, Arrow keys
3. Verify every interactive element is reachable and usable
4. Document any dead-ends or broken navigation

### Screen Reader Test
1. Enable NVDA (Windows) or VoiceOver (Mac)
2. Navigate site with screen reader only
3. Verify all content is announced correctly
4. Document any missing labels or confusing announcements

### Visual Test
1. View site on large monitor (1920px+)
2. Check background pattern is visible but subtle
3. Check pattern doesn't interfere with content
4. Verify pattern works at different zoom levels (100%, 125%, 150%)

---

## Next Steps

1. **Review this CONTEXT.md** — User confirms decisions are correct
2. **Run `/gsd-plan-phase 5`** — AI generates detailed implementation plan with tasks
3. **Plan verification** — User reviews plan for accuracy
4. **Execute Phase 5** — AI implements features task by task
5. **Verify success criteria** — User tests all criteria manually

---

*5-CONTEXT.md created: 2026-04-19*
*Decisions captured via structured discussion*
*Ready for Phase 5 planning*
