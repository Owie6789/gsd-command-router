# GSD Command Router

<div align="center">
  <img src="https://github.com/gsd-build/get-shit-done/blob/0171f7055371f345338226688eb394439ffd79f5/assets/gsd-logo-2000-transparent.png" alt="GSD Command Router" width="80" height="80" style="margin-bottom: 20px;">
</div>

## why this exists

GSD has 50+ commands across 8 categories. Remembering all of them is impossible. Finding the right one for your situation shouldn't require scrolling through a massive README or guessing which command does what.

this fixes that.

---

## What It Does

**GSD Command Router** is an interactive documentation site that helps you discover, filter, and understand GSD (Get Shit Done) CLI commands:

- **Smart Search** — Weighted scoring ranks results by relevance (command name, description, flags)
- **Runtime Filtering** — Filter commands by AI tool (Claude Code, Cursor, Gemini, etc.)
- **Category + Runtime** — Combine filters with AND/OR logic
- **Command History** — Tracks your last 10 viewed commands
- **Favorites System** — Star frequently used commands for quick access
- **Theme Colors** — Switch between 5 accent colors (Yellow, Red, Blue, Green, Purple)
- **Font Preferences** — Choose UI and code fonts (Satoshi, Google Sans, JetBrains Mono, etc.)
- **Full Keyboard Nav** — Navigate entirely without mouse (Tab, Arrow keys, Enter, Escape)
- **State Persistence** — All preferences persist across reloads

### Export Options

- **Copy Usage** — Copy command usage with one click
- **Export Commands** — Export filtered commands as Markdown/PDF (coming in v2.0)
- **Shareable URLs** — Share filtered views with query params (coming in v2.0)

---

## Quick Start

### Online

Open `gsd.html` in any modern browser. No server, no build step, no dependencies.

### Offline

1. Download the `gsd.html` file from this repository
2. Open it in any browser
3. Works completely offline — no internet required

---

## Features

✨ **Zero Dependencies** — Pure HTML, CSS, and JavaScript (vanilla, no frameworks)
⚡ **Instant Load** — Single file, ~5500 lines, loads instantly
🎨 **Minimal Design** — Clean, dark theme with Satoshi font
📱 **Mobile Friendly** — Responsive design (860px, 700px breakpoints)
🔒 **Privacy First** — Everything happens in your browser, nothing sent anywhere
♿ **Accessible** — WCAG AA compliant with ARIA labels, skip links, keyboard navigation
🌙 **Theme Colors** — 5 accent colors (Yellow, Red, Blue, Green, Purple)
🔧 **Font Preferences** — 4 UI fonts, 5 code fonts with persistence
📊 **Command History** — Last 10 viewed commands tracked
⭐ **Favorites** — Star and filter your most-used commands

---

## Supported AI Runtimes

GSD Command Router supports 15+ AI coding tools:

- Claude Code
- Cursor
- Windsurf
- Copilot
- Codex
- Gemini CLI
- OpenCode
- Kilo Code
- Cline
- Amp
- Trae
- Qwen Code
- CodeBuddy
- Augment
- Antigravity

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `/` | Focus search |
| `Tab` | Navigate forward |
| `Shift+Tab` | Navigate backward |
| `Arrow Down` | Next command |
| `Arrow Up` | Previous command |
| `Home` | First command |
| `End` | Last command |
| `Enter` | Expand/collapse command |
| `Escape` | Close command → Clear search → Blur (sequential) |

---

## Technical Details

- **Browser Support** — All modern browsers (Chrome, Firefox, Safari, Edge)
- **File Size** — Single HTML file, ~5500 lines
- **Performance** — Optimized with debounced search (300ms), virtual scrolling ready
- **Accessibility** — WCAG AA compliant with proper ARIA labels, skip links, focus management
- **Storage** — localStorage for state persistence (gsd:lastView, gsd:searchQuery, etc.)

---

## Project Stats

| Metric | Value |
|--------|-------|
| **UAT Tests** | 32/32 PASS (100%) |
| **Development Time** | ~8h across 4 sessions |
| **Lines of Code** | ~5500 (gsd.html + planning docs) |
| **Test Coverage** | 0% (manual UAT only) |
| **Version** | v1.0 (2026-04-19) |

---

## Roadmap (v2.0)

- [ ] Export commands as Markdown/PDF
- [ ] Shareable URLs with query params
- [ ] Print-optimized stylesheet
- [ ] Automated testing (Vitest)
- [ ] ESLint/Prettier configuration
- [ ] Light theme toggle
- [ ] Command comparison feature
- [ ] Analytics dashboard (opt-in)

---

## Contributing

Found a bug? Have an idea? Open an issue or submit a PR. Please keep it simple.

---

## License

MIT — Do whatever you want with it

---

<div align="center">

**v1.0** — Command Router Core Features


Made with ❤️ by [Owie](https://github.com/Owie6789)

</div>
