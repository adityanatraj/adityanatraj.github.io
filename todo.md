# HEIC Converter — Build Checklist

## Tasks

- [x] Explore repo structure and existing tool patterns
- [x] Research heic2any library API and CDN
- [x] Draft design and architecture plan (approved)
- [ ] Create tools/web/tools/heic_converter.html
  - [ ] HTML skeleton (nav, controls, drop zone, results, download-all, privacy note)
  - [ ] CSS (design tokens, layout, cards, spinner, responsive)
  - [ ] JS — library loading with error fallback
  - [ ] JS — drop zone (drag-and-drop + click-to-browse)
  - [ ] JS — format toggle + quality slider
  - [ ] JS — sequential queue + convertFile
  - [ ] JS — card rendering (queued / converting / done / error states)
  - [ ] JS — re-convert all (manual, settings-drift detection)
  - [ ] JS — individual download + download all (staggered)
  - [ ] JS — controls disabled during conversion
  - [ ] JS — cleanup (revokeAll on beforeunload)
  - [ ] JS — HTML escaping (XSS prevention)
  - [ ] Accessibility (aria, keyboard nav)
- [ ] Register tool in index.html tools array
- [ ] Code review (staff engineer perspective)
- [ ] Commit and push to feature branch

## Decisions Locked In
- Sequential conversion queue (one file at a time)
- Manual Re-convert All button (no auto re-convert on settings change)
- Individual file downloads — no ZIP for now
- No file size cap (browser-local, no cost to the server)
- No EXIF/orientation correction (out of scope)
- CDN: https://cdn.jsdelivr.net/npm/heic2any@0.0.4/dist/heic2any.min.js
