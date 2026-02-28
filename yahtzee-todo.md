# Yahtzee Design Review – Todo

## Items

- [x] **1. Remove dead `@keyframes pulse`**
  - Defined at ~line 263, never applied anywhere. Pure dead code. Delete the entire `@keyframes pulse { ... }` block.

- [x] **2. Fix no-op radial-gradient on body**
  - `background: radial-gradient(circle at top, #020617, #020617)` starts and ends at the same color. Replace with `background: var(--bg)`.

- [x] **3. Fix `padding-bottom: 80px` on `main`**
  - Hardcoded 80px is a leftover, creates empty zone at bottom especially on mobile. Replace with `padding-bottom: calc(16px + env(safe-area-inset-bottom))` to match hangman.html's pattern.

- [x] **4. Add `--danger` CSS variable for `#ef4444`**
  - `#ef4444` is hardcoded in `.scorecard td.score.used.below-par`. Add `--danger: #ef4444` to `:root` and replace the hardcoded value with `var(--danger)`.

- [x] **5. Replace three copy-pasted modal button group inline styles with `.modal-actions` class**
  - Three `<div style="display: flex; gap: 10px; justify-content: center; margin-top: 20px;">` blocks in winnerModal, statsModal, and statsLoadModal. Created a `.modal-actions` CSS class and replaced all three.

- [x] **6. Add focus indicator to `.player-name` input**
  - Added `border-bottom: 1px solid transparent` (invisible at rest) and `.player-name:focus { border-bottom-color: var(--accent) }` so players get a cyan underline when the name field is active.

- [x] **7. Rename `{ Roll }` button to "Roll Dice"**
  - Changed `{ Roll }` to `Roll Dice` on the roll button.

- [x] **8. Restrict `min-height: 44px` on `.scorecard td.score` to touch devices**
  - Moved `min-height: 44px` into `@media (hover: none)` so it only applies on touch screens.

- [x] **9. Add `max-height` + `overflow-y: auto` to `.modal-content`**
  - Added `max-height: 85vh; overflow-y: auto` to `.modal-content` so tall stats modals scroll instead of clipping.

- [x] **10. Add `viewport-fit=cover` to the `<meta name="viewport">` tag**
  - Updated to `content="width=device-width, initial-scale=1.0, viewport-fit=cover"` to support notched/island phones.
