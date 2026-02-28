# CLAUDE.md — adityanatraj.github.io

Personal GitHub Pages site: a minimal portfolio + collection of self-contained browser tools.
Served statically via GitHub Pages from `main`. No build step, no framework, no dependencies.

---

## Spirit of the Site

- **Simple tools that solve specific problems** — lightweight, offline-first, browser-native
- **No tracking, no signup, no data leaving the device** — stated explicitly in about.html
- **"Slop-coded but reviewed"** — author's own words; AI-assisted code is fine, but it gets a pass before shipping
- **MIT licensed** — all tools are open source, provided as-is
- The site is meant to be low-maintenance, not a showcase of engineering craft

---

## File Structure

```
/
├── index.html              # Main tools landing page (light theme)
├── about.html              # Philosophy/about page (dark theme, respects prefers-color-scheme)
├── CLAUDE.md               # This file
├── README.md               # Minimal (one line)
├── .gitignore              # Only .DS_Store
└── tools/
    └── web/
        ├── qr_coder.html
        ├── yahtzee.html
        ├── hangman.html
        └── language/
            ├── vietnamese_conversations.html
            └── vietnamese_hotel.html
```

Tools live in `tools/web/`. Thematically grouped tools go in subdirectories (e.g., `language/`).
No shared CSS, JS, or template files — every HTML page is fully self-contained.

---

## Design Language

### Index (index.html) — Light Theme
- Background: `#fafafa` / `#ffffff`
- Text: `#1a1a1a` primary, `#666666` secondary, `#999999` tertiary
- Accent: `#3b82f6` (blue)
- Border: `#e5e5e5`
- Shadows: very subtle (`0 1px 3px rgba(0,0,0,0.05)`)
- Transition: `all 0.2s cubic-bezier(0.4, 0, 0.2, 1)`
- Font: system stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI', ...`)
- Responsive breakpoints: 768px and 480px

### About (about.html) — Light Theme (matches index)
- Same CSS variables as index
- Gradient text on h1 (dark→gray via `-webkit-background-clip`) adapted for light background
- Max-width container: 800px (narrower than index's 1200px — intentional for prose readability)

### Game Tools (yahtzee, hangman) — Dark Theme
- Very dark backgrounds (`#020617`, `#0f1220`) — intentional, reduces eye strain for gaming
- Accent colors differ: Yahtzee uses cyan `#38bdf8`, Hangman uses blue `#7aa2ff`
- No nav bar — standalone tool experience, not meant to navigate away mid-game

### Language Tools (vietnamese_*) — Light Theme
- Light beige/white, minimalist, inspired by magazine/poster layout
- No nav bar
- Uses Web Speech API for text-to-speech

### Shared Aesthetic Principles
- Typography-first: hierarchy through size/weight/color, not decoration
- Rounded corners: 8–16px depending on element size
- Hover effects: color shift + subtle translateY(-2px) or translateX(4px)
- Emojis as tool icons (functional, not semantic)
- No external CSS frameworks — all styles are inline in `<style>` tags

---

## The Index Page (`index.html`)

The tool list is **data-driven**. All tools are defined as objects in a `const tools = [...]` array near the bottom of the file, rendered via `innerHTML`. The `#toolsList` div in HTML is empty — tools are injected at runtime.

**To add a tool**, add one object to the `tools` array:
```js
{ href: 'tools/web/mytool.html', icon: '🔧', title: 'My Tool', tag: 'Utility', description: 'One sentence.', tags: 'searchable keywords here' }
```

**To remove a tool**, delete its line from the `tools` array.

Fields: `href`, `icon` (emoji), `title`, `tag` (category label), `description` (one sentence), `tags` (space-separated search keywords).

---

## Known Issues

None outstanding as of the initial cleanup pass (Feb 2026).

---

## Conventions to Follow

- **Self-contained files**: Do not create shared CSS/JS files. Keep everything inside the single HTML file.
- **No build tools**: No npm, no bundlers, no preprocessors. Plain HTML/CSS/JS only.
- **CDN dependencies only when necessary**: qr_coder.html uses a CDN-hosted QRCode.js. Prefer native browser APIs. If a CDN is needed, use a pinned version.
- **CSS variables at `:root`**: All color/shadow/transition values go in `:root` CSS variables for easy theming.
- **No comments in HTML/CSS** unless logic is non-obvious.
- **Tabs for indentation** (matches existing code style).
- **Inline onclick handlers** are acceptable in tool pages (existing pattern in hangman.html).
- **State management**: Single flat state object (e.g., `gameState = {...}`) — no classes, no frameworks.
- **localStorage** for persistent stats only (not game state between sessions unless explicitly a feature).
- **Save/load via JSON file download/upload** is the established pattern for game state persistence.

---

## Tool Page Template Notes

When creating a new game/tool page:
- Match the dark game theme if it's a game (`#020617` background, cyan/blue accent)
- Match the light theme for utility/reference tools
- Include mobile-safe padding: `padding-bottom: env(safe-area-inset-bottom)` for games
- Standard font stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`
- After creating the file, **remember to add a row to index.html**

---

## What This Site Is Not

- Not a blog
- Not a portfolio of professional work
- Not a showcase of React/TypeScript/modern web dev
- Not a production app — no error monitoring, no analytics, no CI/CD beyond GitHub Pages auto-deploy
