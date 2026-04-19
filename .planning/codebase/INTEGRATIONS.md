# External Integrations

**Analysis Date:** 2026-04-19

## APIs & External Services

**Font Delivery:**
- **Fontshare API** - Font delivery service by Indian Type Foundry
  - Endpoint: `https://api.fontshare.com/v2/css`
  - Font: Satoshi (variable weights: 300, 400, 500, 600, 700, 800, 900)
  - Usage: Primary UI typeface for all text rendering
  - CDN URL: `https://api.fontshare.com/v2/css?f[]=satoshi@300,400,500,600,700,800,900&display=swap`
  - Preconnect: `<link rel="preconnect" href="https://api.fontshare.com">`

**GitHub:**
- **GitHub Repository Link** - External link in top navigation
  - URL: `https://github.com/gsd-build/get-shit-done`
  - Purpose: Link to official GSD source repository
  - Attributes: `target="_blank" rel="noopener"` for security

## Data Storage

**Databases:**
- **None** - No database connections

**Local Storage:**
- **Browser localStorage** (implied) - For session state persistence
  - Used for: Command history, view state, filter preferences
  - No external sync

**File Storage:**
- **Local filesystem only** - Planning documents stored in `.planning/codebase/`
- No cloud storage integration

**Caching:**
- **Browser cache only** - No explicit caching layer
- Font CDN uses standard HTTP caching headers

## Authentication & Identity

**Auth Provider:**
- **None** - No authentication system
- No user accounts, no sessions, no tokens

## Monitoring & Observability

**Error Tracking:**
- **None** - No error tracking service (no Sentry, LogRocket, etc.)
- Errors surface via browser console only

**Logs:**
- **Console logging** - Development debugging via `console.log()` (if any)
- No production logging infrastructure

**Analytics:**
- **None** - No analytics (no Google Analytics, Plausible, etc.)
- No telemetry, no usage tracking

## CI/CD & Deployment

**Hosting:**
- **Static file hosting** - Any static host can serve `gsd.html`
- Options: GitHub Pages, Netlify, Vercel, AWS S3, Cloudflare Pages, local file://

**CI Pipeline:**
- **None** - No CI/CD configured for the UI file itself
- No automated tests, no build pipeline, no deployment scripts

**Version Control:**
- **Git** - Source code managed via Git
- Repository: `https://github.com/gsd-build/get-shit-done`

## Environment Configuration

**Required env vars:**
- **None** - No environment variables required

**Secrets location:**
- **None** - No secrets, API keys, or credentials in the codebase

**CDN Configuration:**
- Fontshare CDN is public and requires no API key
- No rate limiting or authentication for font requests

## Webhooks & Callbacks

**Incoming:**
- **None** - No webhook endpoints (no server to receive them)

**Outgoing:**
- **None** - No outgoing webhooks or HTTP callbacks

## Third-Party Scripts

**Analytics Scripts:**
- **None** - No third-party JavaScript loaded

**Tracking Pixels:**
- **None** - No tracking pixels or beacons

**Embedded Content:**
- **None** - No iframes, no embedded widgets

## Content Delivery

**CDN Resources:**
| Resource | URL | Purpose |
|----------|-----|---------|
| Satoshi Font | `api.fontshare.com` | UI typography |

**Self-Hosted:**
- All CSS - Inline in `<style>` block
- All JavaScript - Inline in `<script>` block
- All HTML - Single document structure
- SVG icons - Inline in HTML (search icon, expand arrows)

## Network Requirements

**Internet Required:**
- **Yes** - For Fontshare CDN on first load
- **After cache** - Fonts are cached by browser, subsequent loads work offline

**Offline Capability:**
- **Partial** - UI functions without internet after font cache
- Fallback fonts: `-apple-system, sans-serif` if Fontshare unavailable

**CORS:**
- **Not applicable** - No cross-origin XHR/fetch requests
- Font CDN serves CSS with appropriate CORS headers

## Integration Points (Future)

**Potential Integrations:**
- **GitHub API** - Could integrate for PR creation, branch management
- **AI APIs** - Could integrate with Claude, GPT, Gemini for command execution
- **File System Access API** - Could enable direct file writing in supported browsers

**Current Limitations:**
- No backend integration
- No real-time sync
- No multi-user collaboration features

---

*Integration audit: 2026-04-19*
