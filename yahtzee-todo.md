# Yahtzee – Second Pass Todo

## Items

- [x] **1. Dice pip faces instead of digit characters**
  - In `renderDice()`, added `const PIPS = ['', '⚀', '⚁', '⚂', '⚃', '⚄', '⚅']` and changed `el.textContent = d.value` → `el.textContent = PIPS[d.value]`.
  - Updated `.die` font-size from `30px` to `44px` and removed `font-weight: 800` (redundant for Unicode glyphs).

- [x] **3. Modal responsive width**
  - Changed `.modal-content` from `max-width: 500px` to `width: min(500px, calc(100vw - 32px))` for proper 16px margins on narrow viewports.

- [x] **4. Remove console.log statements**
  - Removed all four `console.log` calls in production paths (legacy save, replace/merge/skip stats actions).
  - `console.warn` and `console.error` in catch blocks left intact.

- [x] **5. Revoke Object URL after save download**
  - Added `setTimeout(() => URL.revokeObjectURL(a.href), 0)` immediately after `a.click()` in `saveGame()`.

- [x] **6. Replace remaining hardcoded color values with CSS variables**
  - Added `--die-face: #f8fafc` to `:root`.
  - Replaced: `header { background: #020617 }` → `var(--bg)`
  - Replaced: `button { color: #020617 }` → `var(--bg)`
  - Replaced: `button.secondary { background: #334155 }` → `var(--line)`
  - Replaced: `.scorecard tr.section td { background-color: #020617 }` → `var(--bg)`
  - Replaced: `.die { background: #f8fafc; color: #020617 }` → `var(--die-face)` / `var(--bg)`

- [x] **7. Spacebar / Enter triggers Roll Dice**
  - Added `document.addEventListener('keydown', ...)` at bottom of script.
  - Fires `rollDice()` on Space or Enter when no modal is open and rollBtn is not disabled.
  - Calls `e.preventDefault()` to suppress browser scroll on Space.

- [x] **8. Rename "Small" / "Large" straight labels to "Sm. Str." / "Lg. Str."**
  - Updated CATEGORIES array entries for `smallStraight` and `largeStraight`.

- [x] **9. Warn before discarding an in-progress game**
  - Added guard at top of `newGame()`: checks `!gameState.gameEnded && any scores filled`.
  - Shows `confirm()` only when game is actually in progress; skips on fresh start or after game ends.
