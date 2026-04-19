# Testing Patterns

**Analysis Date:** 2026-04-19

## Current State

**Test Framework:** None detected

**Test Files:** None found

**Test Coverage:** Not measured

---

## Project Characteristics

This codebase is a **single-file static web application** (`gsd.html`) with:

- **1762 lines** of HTML, CSS, and JavaScript combined
- **No build step** — runs directly in browser
- **No dependencies** — vanilla JavaScript, no npm packages
- **No module system** — all code in global scope

---

## Test Framework (If Added)

### Recommended Setup

Given the single-file nature, the following approaches are recommended:

#### Option 1: Vitest + jsdom (Recommended)

```bash
npm install -D vitest jsdom
```

**vitest.config.js:**
```javascript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    include: ['**/*.test.js'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
    },
  },
});
```

**Why Vitest:**
- Fast execution
- Built-in coverage
- JSDOM for DOM testing
- Compatible with Vite if project grows

#### Option 2: Jest + jsdom

```bash
npm install -D jest jsdom @testing-library/dom @testing-library/jest-dom
```

**jest.config.js:**
```javascript
module.exports = {
  testEnvironment: 'jsdom',
  testMatch: ['**/*.test.js'],
  setupFilesAfterEnv: ['@testing-library/jest-dom'],
};
```

---

## Test File Organization

### Recommended Structure

```
gsd/
├── gsd.html
├── package.json
├── vitest.config.js
└── tests/
    ├── commands.test.js
    ├── wizard.test.js
    ├── navigation.test.js
    └── utils/
        └── escHtml.test.js
```

### Naming Convention

- **Pattern:** `[feature].test.js`
- **Location:** `tests/` directory (separate from source)

---

## Test Structure Patterns

### Unit Test Example

```javascript
import { describe, it, expect, beforeEach } from 'vitest';

describe('escHtml', () => {
  it('escapes ampersands', () => {
    expect(escHtml('A & B')).toBe('A &amp; B');
  });

  it('escapes less-than signs', () => {
    expect(escHtml('a < b')).toBe('a &lt; b');
  });

  it('escapes greater-than signs', () => {
    expect(escHtml('a > b')).toBe('a &gt; b');
  });

  it('escapes quotes', () => {
    expect(escHtml('say "hello"')).toBe('say &quot;hello&quot;');
  });

  it('handles null/undefined', () => {
    expect(escHtml(null)).toBe('null');
    expect(escHtml(undefined)).toBe('undefined');
  });
});
```

### Component Test Example

```javascript
import { describe, it, expect, beforeEach } from 'vitest';
import { JSDOM } from 'jsdom';

describe('Command Rendering', () => {
  let container;

  beforeEach(() => {
    const dom = new JSDOM('<!DOCTYPE html><body><div id="cmd-list"></div></body>');
    global.window = dom.window;
    global.document = dom.window.document;
    container = document.getElementById('cmd-list');
  });

  it('renders command list with correct structure', () => {
    renderCommands();
    
    const rows = container.querySelectorAll('.cmd-row');
    expect(rows.length).toBeGreaterThan(0);
    
    const firstRow = rows[0];
    expect(firstRow.querySelector('.cmd-name code')).toBeTruthy();
    expect(firstRow.querySelector('.cmd-short')).toBeTruthy();
    expect(firstRow.querySelector('.badge')).toBeTruthy();
  });

  it('toggles detail view on click', () => {
    renderCommands();
    
    const firstRow = container.querySelector('.cmd-row');
    firstRow.click();
    
    expect(firstRow.classList.contains('open')).toBe(true);
    expect(firstRow.getAttribute('aria-expanded')).toBe('true');
  });
});
```

### Integration Test Example

```javascript
describe('Search Functionality', () => {
  it('filters commands based on search input', () => {
    renderCommands();
    
    const searchInput = document.getElementById('search');
    searchInput.value = 'phase';
    searchInput.dispatchEvent(new Event('input'));
    
    const visibleRows = document.querySelectorAll('.cmd-row');
    visibleRows.forEach(row => {
      const cmdText = row.querySelector('code').textContent.toLowerCase();
      expect(cmdText).toContain('phase');
    });
  });

  it('shows empty state when no matches', () => {
    renderCommands();
    
    const searchInput = document.getElementById('search');
    searchInput.value = 'xyznonexistent123';
    searchInput.dispatchEvent(new Event('input'));
    
    const emptyState = document.getElementById('empty-state');
    expect(emptyState.style.display).not.toBe('none');
  });
});
```

---

## Mocking

### What to Mock

- **External API calls:** If any are added in future
- **Clipboard API:** For copy functionality

```javascript
// Mock clipboard
global.navigator = {
  clipboard: {
    writeText: vi.fn(() => Promise.resolve()),
  },
};
```

### What NOT to Mock

- **DOM manipulation:** Test actual DOM behavior
- **Event handlers:** Test real click/keyboard interactions
- **Data structures:** Use actual `CMDS` array

---

## Fixtures and Factories

### Command Fixture

```javascript
// tests/fixtures/commands.js
export const MOCK_CMDS = [
  {
    id: "test-command",
    cmd: "/gsd-test",
    cat: "core",
    short: "Test command",
    desc: "A test command for testing",
    flags: ["--test"],
    usage: "/gsd-test arg",
    when: "When testing",
    related: [],
    runtimes: ["Test Runtime"]
  }
];
```

### Location

- **Fixtures:** `tests/fixtures/`
- **Helpers:** `tests/helpers/`

---

## Coverage

### Recommended Target

- **Statements:** 80%
- **Branches:** 70%
- **Functions:** 80%
- **Lines:** 80%

### View Coverage

```bash
npm run test -- --coverage
# Opens coverage report in browser
```

---

## Test Types

### Unit Tests

**Scope:** Individual functions and utilities

**Examples:**
- `escHtml()` — HTML escaping
- `getFiltered()` — Command filtering logic
- `copyText()` — Clipboard operations

### Integration Tests

**Scope:** Component interactions

**Examples:**
- Search input → filtered command list
- Click row → expanded detail view
- Filter chip → category filtering
- Wizard flow → step navigation

### E2E Tests

**Framework:** Playwright or Cypress (if needed)

**When to add:**
- Multiple HTML files
- Backend integration
- Complex user flows

---

## Common Patterns

### Async Testing

```javascript
it('copies text to clipboard', async () => {
  const mockWriteText = vi.fn(() => Promise.resolve());
  global.navigator.clipboard = { writeText: mockWriteText };
  
  copyText('test', button);
  
  await vi.waitFor(() => {
    expect(mockWriteText).toHaveBeenCalledWith('test');
  });
});
```

### Error Testing

```javascript
it('handles missing command gracefully', () => {
  const result = CMDS.find(c => c.id === 'nonexistent');
  expect(result).toBeUndefined();
});
```

### DOM Query Testing

```javascript
it('renders badges with correct category class', () => {
  renderCommands();
  
  const coreBadge = document.querySelector('.b-core');
  expect(coreBadge).toBeTruthy();
  expect(coreBadge.textContent).toBe('Core');
});
```

---

## Test Commands (Recommended)

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

---

## Testing Checklist

### Before Adding Tests

- [ ] Extract JavaScript to separate `.js` file(s) for easier testing
- [ ] Set up package.json with test dependencies
- [ ] Configure test runner (Vitest/Jest)
- [ ] Create `tests/` directory

### Priority Test Areas

1. **Utility functions** (`escHtml`, `copyText`)
2. **Filter logic** (`getFiltered`)
3. **View switching** (`switchView`)
4. **Wizard navigation** (step progression)
5. **Search functionality** (input → filter)
6. **Keyboard shortcuts** (/, Escape, Enter)

### Future Considerations

If the project grows:

- Split `gsd.html` into separate `.html`, `.css`, `.js` files
- Add TypeScript for type safety
- Consider component framework (React, Vue, Svelte)
- Add visual regression testing (Playwright screenshots)

---

*Testing analysis: 2026-04-19*
