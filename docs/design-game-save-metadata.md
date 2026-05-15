# Design Document: Game Save Metadata — Start, Current, and End Times

**Author**: Staff Engineer, Web Design & Single-Page Games Team  
**Date**: 2026-05-15  
**Status**: Proposal

---

## What We Were Asked

When players save and load games, they want to know:

- **When did this game start?**
- **Where am I in it right now?** (last save time, if still playing)
- **When did it end?** (if the game is over)

The proposed semantics:

| Field | Meaning |
|---|---|
| `startedAt` | When the player began this game session |
| `endedAt` | When the game reached its final state (win/loss/complete) |
| `currentTime` | `endedAt` if game is over, otherwise `savedAt` (last save time) |

`currentTime` is a **derived display concept** — it is not an independent stored field. It answers "where in time is this game right now?"

---

## Current State Audit

There are three games in the repository. All use file-based save/load (JSON download + file picker upload). Yahtzee also persists aggregate statistics to `localStorage`.

### Checkers (`tools/web/checkers.html`)

| Field | Present? | Notes |
|---|---|---|
| `startedAt` | No | |
| `savedAt` | **Yes** | `new Date().toISOString()`, set on every save |
| `endedAt` | No | Game has `phase: 'gameover'` and `winner` fields but no timestamp |

The save envelope already wraps `savedAt` at the top level alongside `gameState`. It is the closest to the target of any of the three games.

### Hangman (`tools/web/hangman.html`)

| Field | Present? | Notes |
|---|---|---|
| `startedAt` | No | |
| `savedAt` | No | State is saved as raw `JSON.stringify(state)` with no metadata wrapper |
| `endedAt` | No | Game has `over: boolean` but no timestamp |

Hangman is the only game with **no timestamp tracking at all**. It is also structurally the simplest, which makes it the easiest to retrofit.

### Yahtzee (`tools/web/yahtzee.html`)

| Field | Present? | Notes |
|---|---|---|
| `startedAt` | No | |
| `savedAt` | **Yes** | `new Date().toISOString()`, top-level in the save envelope |
| `endedAt` | No | Game has `gameEnded: boolean` in `gameState` but no timestamp |

Yahtzee also stores per-player statistics in `localStorage` (wins, all scores, high score) but these have no per-game timestamps either, which means game duration cannot currently be computed even in aggregate.

### Summary Table

| Game | `startedAt` | `savedAt` | `endedAt` | Stats |
|---|---|---|---|---|
| Checkers | — | ✓ | — | None |
| Hangman | — | — | — | None |
| Yahtzee | — | ✓ | — | localStorage, no timestamps |

No game currently tracks when it **started** or when it **ended**.

---

## Should We Pursue This?

**Yes. Unambiguously.**

This is a small change with outsized payoff and essentially zero downside. The reasons:

### Value to Players

A save file without a start time is an orphan — when a player opens it weeks later, the file name tells them nothing about context. A save file that says "started 6 days ago, last saved 2 days ago" is immediately grounding. For a completed game, "finished in 47 minutes" is a satisfying data point.

### Value to the Team

`endedAt − startedAt` gives us game duration. For Yahtzee especially, where we already track stats like high score and win counts, game duration is a natural next metric. We cannot add it retroactively without this foundation.

### Implementation Risk Is Near Zero

All saves are file-based. There is no server, no database schema, no migration to coordinate. Old save files that lack these timestamps simply load with `null` values — we handle that case gracefully and the game works exactly as before. This is purely additive.

### The Work Is Small

Three games, three nearly identical changes. Each game already has a "game ends" moment and a "new game starts" moment. We are adding two `new Date().toISOString()` calls and threading them through the save envelope. Estimated implementation time: 2–3 hours across all three games, including testing.

---

## Proposed Schema

### Standard Save Envelope (all games)

```json
{
  "version": "1.1",
  "startedAt": "2026-05-10T14:22:00.000Z",
  "savedAt":   "2026-05-12T09:15:30.000Z",
  "endedAt":   null,
  "gameState": { ... }
}
```

- `startedAt`: set once when `initGame()` / new-game setup runs; carried through every save
- `savedAt`: set on every save (existing behavior in Checkers and Yahtzee)
- `endedAt`: set to `new Date().toISOString()` the moment the game reaches its terminal state; `null` until then

`currentTime` (for display) is always computed, never stored:

```javascript
const currentTime = gameState.endedAt ?? savedAt;
```

### Yahtzee Statistics Extension (optional, Phase 2)

When a game ends, record duration alongside the existing stats:

```json
{
  "gamesPlayed": 12,
  "playerWins": { "Alice": 7, "Bob": 5 },
  "playerScores": { "Alice": [243, 198, ...], "Bob": [210, 177, ...] },
  "highestScore": { "name": "Alice", "score": 312 },
  "gameDurations": [2820, 3140, ...]
}
```

`gameDurations` stores seconds per game. From this, average session length becomes trivially computable.

---

## Per-Game Implementation Plan

### Checkers

1. In `initGame()`: set `gameState.startedAt = new Date().toISOString()`; set `gameState.endedAt = null`
2. When winner is determined (where `phase` is set to `'gameover'`): set `gameState.endedAt = new Date().toISOString()`
3. Save function: carry `startedAt` and `endedAt` into the save envelope (alongside existing `savedAt`)
4. Load function: restore `startedAt` and `endedAt` from file; handle `undefined`/`null` gracefully for old saves

### Hangman

1. In new-game setup (when `state` is initialized): set `state.startedAt = new Date().toISOString()`; set `state.endedAt = null`
2. Where `state.over = true` is set: also set `state.endedAt = new Date().toISOString()`
3. Save function: wrap state in an envelope (add `version`, `startedAt`, `savedAt`, `endedAt`) instead of bare `JSON.stringify(state)` — bump filename logic, maintain backward-compatible load
4. Load function: detect old format (no `version` key) and handle both paths

### Yahtzee

1. In `initGame()`: set `startedAt` inside `gameState`; set `endedAt = null`
2. Where `gameEnded = true` is set: also set `gameState.endedAt = new Date().toISOString()`
3. Save function: carry `startedAt` and `endedAt` into the save envelope (alongside existing `savedAt`)
4. Load function: restore `startedAt` and `endedAt` from file; existing backward-compatibility logic already handles this pattern
5. `updateStats()` (end-of-game stats write): optionally append `(endedAt - startedAt)` in seconds to `gameDurations` array in localStorage

---

## Backward Compatibility

Old save files will not have `startedAt` or `endedAt`. The load path should treat these as `null` and proceed normally. No warning or error is appropriate — old saves simply won't display time metadata. New saves will always have all three fields.

No version bump is needed for Checkers or Yahtzee since both already have a `version` field; a minor version increment (`"1.1"` → `"1.2"`) is sufficient to signal the schema change if we ever need to distinguish old from new files programmatically.

Hangman currently has no envelope at all. The load function should check for the presence of a `version` key: if absent, treat the entire JSON as the raw game state (old format); if present, unwrap the `gameState` key (new format).

---

## Phased Rollout

### Phase 1 — Store the timestamps (recommended now)

Implement `startedAt`, `savedAt`, and `endedAt` across all three games. No UI change is required — the timestamps live in the save file and are available for Phase 2. This phase has no user-visible impact except enriched save files.

### Phase 2 — Surface the timestamps in the UI

Show meaningful time information at save and load moments:

- **Save dialog / filename**: `checkers-save.json` → `checkers-save-2026-05-12.json` (optional)
- **Load confirmation**: "Game started May 10 · last saved May 12 · in progress" or "Game started May 10 · finished May 10 (47 min)"
- **In-game header/footer**: subtle "started 3 days ago" or session timer

Phase 2 is where players actually feel the value. Phase 1 makes Phase 2 possible.

### Phase 3 — Analytics in Yahtzee (optional, later)

Once `gameDurations` accumulates data in localStorage, expose it in the stats modal: "Average game: 52 min · Fastest: 31 min." This is purely additive to existing stats infrastructure.

---

## What We Are Not Proposing

- **Server-side storage**: All timestamps stay in the save file and localStorage. No backend, no accounts, no sync.
- **Auto-save**: The current explicit save-file model is good. Timestamps do not require changing when saves happen.
- **Breaking existing saves**: Nothing in this proposal invalidates old save files.
- **A session timer visible during play**: That is a UX decision for Phase 2, not a prerequisite for Phase 1.

---

## Recommendation

**Proceed with Phase 1 now.** The implementation is small, the risk is zero, and it lays the groundwork for everything else on this list. Phase 2 (surfacing the times in the UI) should follow once we agree on how we want to present them — that is a design conversation worth having separately, and it is much easier to have with real data already in the save files.

Hangman needs the most structural change (adding the save envelope) but it is also the simplest game, so it is the right place to start and validate the pattern before applying it to Checkers and Yahtzee.
