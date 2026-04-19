# Technology Stack

**Analysis Date:** 2026-04-19

## Languages

**Primary:**
- **HTML5** - Single-file document structure with semantic elements (`<header>`, `<nav>`, `<main>`, `<section>`)
- **CSS3** - All styling via `<style>` block in head. Uses custom properties (CSS variables), CSS Grid, Flexbox, and animations
- **JavaScript (ES6+)** - Vanilla JavaScript, no transpilation. Uses `const`/`let`, arrow functions, template literals, destructuring, modules pattern

**Secondary:**
- **Markdown** - Used for planning documents in `.planning/codebase/` directory

## Runtime

**Environment:**
- **Browser** - Modern web browser with ES6+ support
- **No server-side runtime** - Purely client-side application

**Package Manager:**
- **None** - No package management, no `package.json`, no dependencies to install

**Build Tools:**
- **None** - No bundlers (Webpack, Vite, Rollup), no transpilers (Babel, TypeScript), no minifiers
- **Zero build step** - The file runs directly in the browser

## Frameworks

**Core:**
- **None** - Pure vanilla JavaScript implementation

**UI/Styling:**
- **None** - No CSS frameworks (Tailwind, Bootstrap), no component libraries
- Custom CSS architecture with `:root` custom properties for theming

**Testing:**
- **None** - No test framework configured

**Build/Dev:**
- **None** - No development server, no hot reload, no watch mode

## Key Dependencies

**External Fonts:**
- **Satoshi** (Fontshare) - Primary typeface for UI. Loaded via CDN with weights 300-900
- **Fontshare CSS API** - Font delivery mechanism

**CDN Resources:**
- `api.fontshare.com` - Font stylesheet endpoint
- Preconnect hint for performance optimization

**No npm packages:**
- Zero npm dependencies
- No runtime libraries (no React, Vue, jQuery, lodash, etc.)

## Configuration

**Environment:**
- **No environment variables** - No `.env` files, no runtime configuration
- **No build configuration** - No `tsconfig.json`, `vite.config.js`, `webpack.config.js`
- **No linter/formatter config** - No ESLint, Prettier, or Biome configuration files

**File Structure:**
- Single-file architecture: `gsd.html` (1762 lines)
- Planning documents: `.planning/codebase/` (ARCHITECTURE.md, CONVENTIONS.md, STRUCTURE.md, TESTING.md)

## Platform Requirements

**Development:**
- **Minimum:** Modern browser with ES6+ support (Chrome 61+, Firefox 54+, Safari 11+, Edge 79+)
- **Recommended:** Latest Chrome/Firefox for best developer tools
- **Editor:** Any text editor (VS Code, Sublime, etc.) - no IDE requirements
- **No Node.js required** - Can develop without Node installed

**Production:**
- **Deployment:** Static file hosting (GitHub Pages, Netlify, Vercel, S3, etc.)
- **No server required** - Serves as static HTML
- **No backend** - No API server, no database, no runtime processes
- **CDN dependency:** Requires internet access for Fontshare CDN (or self-host fonts)

**Browser APIs Used:**
- `localStorage` - For state persistence (implied by session management)
- DOM API - Full usage for UI manipulation
- Fetch API - For loading external resources (if any)
- CSS Custom Properties - For theming system

## Architecture Pattern

**Single-File Application (SFA):**
- All HTML, CSS, and JavaScript in one `.html` file
- ~1762 lines total
- Self-contained, portable, no build pipeline
- Data embedded as JavaScript constants (`CMDS` array, `RUNTIMES` array)

**CSS Architecture:**
- **Custom Properties:** 30+ CSS variables in `:root` for colors, spacing, radii
- **Color Palette:** Dark theme with semantic color tokens (`--bg`, `--t0`, `--accent`, etc.)
- **Responsive:** Mobile breakpoints at 860px and 700px
- **Grid Layout:** CSS Grid for main app structure (sidebar + main content)

**JavaScript Architecture:**
- **IIFE pattern** - Immediately invoked functions for encapsulation
- **Data-driven UI** - Command data (`CMDS` array) drives view rendering
- **Event delegation** - Single event listeners on parent elements
- **State management** - Simple object-based state with reactive updates

---

*Stack analysis: 2026-04-19*
