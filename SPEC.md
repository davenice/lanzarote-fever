# Lanzarote Fever — Game Specification

## Overview

**Lanzarote Fever** is a mobile-only browser-based puzzle game delivered as a Progressive Web App (PWA). It is a family-personalised flow/connect puzzle — players drag lava-flow paths between family member name tiles and their matching interest emoji tiles. The puzzle is the routing: paths must not cross, and on harder difficulties the board is densely packed so finding valid routes is genuinely challenging.

Suitable for ages 5 and up.

---

## Family Members

14 family members split into children and adults. Each has a unique `id` used to load their avatar image.

### Children
| Name | Emoji | Interest | Avatar file |
|------|-------|----------|-------------|
| Emily | 🎵 | Music | avatars/emily.png |
| Amy | 🤸 | Gymnastics | avatars/amy.png |
| Bethany | 🐭 | Mouse toys | avatars/bethany.png |
| Micah | 🏏 | Cricket | avatars/micah.png |
| Clara | 🍬 | Sweets | avatars/clara.png |
| Esther | 🐯 | Tigers | avatars/esther.png |

### Adults
| Name | Emoji | Interest | Avatar file |
|------|-------|----------|-------------|
| Jon | 🏃 | Running | avatars/jon.png |
| Joanna | 🎨 | Drawing | avatars/joanna.png |
| Rachel | 🎺 | Clarinet / Teaching | avatars/rachel.png |
| Dave | 🎸 | Guitar | avatars/dave.png |
| Granny | 🧵 | Sewing | avatars/granny.png |
| Grandpa | 📐 | Maths | avatars/grandpa.png |
| Jerry | ⛷️ | Skiing | avatars/jerry.png |
| Paul | 🐶 | Dogs | avatars/paul.png |

---

## Avatar System

- On startup, each member's `avatars/{id}.png` is preloaded silently.
- If the image loads successfully, the **name tile** shows the photo instead of text.
- If no image exists, the name tile shows the member's name as text.
- The emoji tile always shows the emoji regardless of avatar availability.
- Avatar images should be square. They are displayed with `object-fit: cover`.
- To add an avatar: drop `{id}.png` into the `avatars/` folder — no code changes needed.

---

## Difficulty Levels

Each puzzle randomly selects the required number of family members from the full pool of 14.

| Setting | Easy | Medium | Hard |
|---------|------|--------|------|
| Pairs | 5 | 8 | 10 |
| Grid (portrait) | 5 × 6 | 6 × 9 | 7 × 12 |
| Grid (landscape) | 6 × 5 | 9 × 6 | 12 × 7 |
| Colour hints | ✅ Yes | ❌ No | ❌ No |
| Min path length | 3 cells | 4 cells | 6 cells |
| Min endpoint distance | 2 (Manhattan) | 3 (Manhattan) | 5 (Manhattan) |
| Min board fill | 80% | 80% | 90% |
| Max gen attempts | 300 | 500 | 1000 |
| Path stop probability | 0.15 | 0.09 | 0.03 |

**Colour hints (Easy only):** each endpoint tile has a faint coloured border matching its pair — the name tile and its emoji share the same tint so young players can see what connects to what.

---

## Game Mechanics

### Goal
Connect each name tile to its correct emoji tile by drawing a lava-flow path. All pairs must be matched to win.

### Matching rule
The player must **guess** which emoji belongs to which name — there is no automatic colour-coding on Medium/Hard. Knowing the family is part of the puzzle.

### Drawing paths
- Tap/press on a name or emoji endpoint to begin drawing.
- Drag through adjacent cells (orthogonal only, no diagonals) to route the path.
- Dragging back over your own path **backtracks** it.
- Paths cannot cross locked paths from other pairs.
- Paths cannot pass through other pairs' endpoint tiles.

### Completing a connection
- Release on the **correct** opposite endpoint → path locks in, glows with its pair colour.
- Release on the **wrong** endpoint → red flash animation, path clears immediately.
- Release anywhere else → path silently clears.

### Undo
- The ↩ button in the HUD undoes the most recently locked pair, restoring those cells to empty.
- Can be pressed repeatedly to undo multiple pairs in reverse order.
- Has no effect on the currently active (in-progress) drag.

### Win condition
All pairs correctly connected → win screen appears after a 350ms delay.

---

## Level Generation

Levels are **randomly generated** on every game start — there are no fixed pre-made levels.

### Algorithm
1. Shuffle the 14-member family pool, select `cfg.pairs` members.
2. For each pair, grow a random path through empty cells:
   - Only start from cells with ≥ `minPathLen` reachable empty cells (flood-fill check).
   - Walk to adjacent empty cells, preferring moves that don't isolate the remaining empty region.
   - Stop randomly once length ≥ `minPathLen` (probability `stopProb` per step).
   - Reject if path is shorter than `minPathLen`.
   - Reject if Manhattan distance between the two endpoints is less than `minEndDist`.
3. After all pairs are placed, reject the board if filled cells ÷ total cells < `minFill`.
4. Retry up to `maxAttempts` times. If all attempts fail, fall back one difficulty level.
5. Return only the **endpoint positions** — the generated paths are discarded. The player must find their own valid routes.

### Why endpoint distance matters
The generated path may wind through 6+ cells, but if the two endpoints happen to be adjacent the player can draw a trivial 2-cell shortcut. Enforcing `minEndDist` prevents this regardless of the board density.

---

## Visual Design

### Theme
Volcanic Lanzarote. Dark lava-rock grid, glowing orange/coloured path lines, fire-toned UI.

### Colour palette
| Pair | Path background | Accent / border | Emoji |
|------|-----------------|-----------------|-------|
| 0 | #7a3000 | #ff6600 | 🟠 orange |
| 1 | #7a0e00 | #ff3300 | 🔴 red-orange |
| 2 | #6a6000 | #ffdd00 | 🟡 yellow |
| 3 | #005a50 | #00ccaa | 🟢 teal |
| 4 | #4a1580 | #bb55ff | 🟣 purple |
| 5 | #3a6000 | #88ff00 | 🟢 lime |
| 6 | #004a70 | #00aaff | 🔵 blue |
| 7 | #6a1545 | #ff44aa | 🩷 pink |
| 8 | #6a4000 | #ff9900 | 🟠 amber |
| 9 | #005535 | #00ffaa | 🟢 mint |

App background: `#0d0400`. Grid cell: `#221008`. Cell border: `#3a1a08`.

---

## Screens

### 1 — Title
🌋 icon, "LANZAROTE FEVER" heading, "Family Connect" subtitle, Play button.

### 2 — Difficulty
Three buttons: Easy / Medium / Hard with short descriptions. Back button.

### 3 — Game
- **HUD** (top bar): ← back, difficulty · solved/total pairs, ↩ undo.
- **Grid**: fills available screen below HUD. Cell size computed to maximise grid within viewport.
- Responsive to orientation change (220ms debounce, restarts current puzzle).

### 4 — Win
🎉 icon, "You got it!" heading, pair count subtitle, Next Puzzle (primary) and Change Difficulty (secondary) buttons.

---

## Responsive Layout

- **Mobile-only** design (no desktop breakpoints).
- Cell size = `min(availableWidth / cols, availableHeight / rows)`, minimum 32px, no hard maximum.
- Accounts for 3px gap between cells and 8px edge padding.
- Portrait orientation uses `pCols × pRows`; landscape swaps to `pRows × pCols`.

---

## PWA

- **Manifest:** `manifest.json`
- **Service worker:** `sw.js` (registered on load; failures caught silently)
- **Install to home screen:** supported on iOS and Android
- **Offline play:** yes (service worker caches assets)
- **Theme colour:** `#1a0800`
- **Status bar:** black-translucent (iOS)
- **App name:** Lanzarote Fever
- **Favicon:** inline SVG (volcano icon, no external file)
- **Hosting:** GitHub Pages (HTTPS required for service worker)

---

## Planned / Not Yet Implemented

- Square photo avatars for the remaining 6 family members (Jon, Joanna, Rachel, Dave, Jerry, Paul).
- Blocked "lava rock" cells as an additional difficulty mechanism.
- Shared-seed multiplayer: same puzzle for multiple players via URL seed parameter (`?seed=abc123`), compare times asynchronously.
