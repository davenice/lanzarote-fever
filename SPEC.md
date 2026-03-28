# Lanzarote Fever — Game Specification

## Overview

**Lanzarote Fever** is a mobile-only browser-based puzzle game delivered as a Progressive Web App (PWA). It is a sliding tile puzzle — the player slides tiles around a grid to place each family member's name/photo tile adjacent to the emoji that represents something they love. The puzzle is the memory and logic challenge of knowing who loves what, plus the spatial challenge of routing tiles without blocking each other.

Suitable for ages 5 and up.

---

## Family Members

14 family members. Each has a unique `id` used to load their avatar image.

### Children
| Name | Emoji | Interest | Avatar file |
|------|-------|----------|-------------|
| Emily | 🎻 | Violin | avatars/emily.png |
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
| Rachel | 🧁 | Baking | avatars/rachel.png |
| Dave | 🎸 | Guitar | avatars/dave.png |
| Granny | 🧵 | Sewing | avatars/granny.png |
| Grandpa | 📐 | Maths | avatars/grandpa.png |
| Jerry | 🏰 | Disney | avatars/jerry.png |
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

| Setting | Easy | Medium | Hard | Advanced |
|---------|------|--------|------|----------|
| Pairs | 4 | 6 | 10 | 10 |
| Wildcards | 0 | 3 | 4 | 4 |
| Grid | 3 × 3 | 4 × 4 | 5 × 5 | 5 × 5 |
| Total tiles | 8 + 1 blank | 12 + 3W + 1 blank | 20 + 4W + 1 blank | 20 + 4W + 1 blank |
| Colour hints | ✅ Yes | ❌ No | ❌ No | ❌ No |
| Shuffle moves | 30 | 80 | 150 | 150 |
| Memory mode | ❌ | ❌ | ❌ | ✅ Yes |

**Colour hints (Easy only):** each name tile and its emoji share a coloured border — young players can use this to identify pairs before they know the family members.

**Wildcards:** random holiday/Lanzarote-themed emoji tiles (`🌋 ☀️ 🌊 🏖️ 🦎 🌴`) that fill out the grid. They are not part of any pair and cannot be matched.

---

## Game Mechanics

### Goal
Slide the tiles around so that each name/photo tile is **orthogonally adjacent** (touching horizontally or vertically) to its correct emoji tile. All pairs must be adjacent simultaneously to win.

### Matching rule
The player must **know** (or guess) which emoji belongs to which family member — there is no automatic colour coding on Medium/Hard/Advanced. Knowing the family is part of the puzzle.

### Sliding tiles
- Tap any tile that is directly adjacent (horizontal or vertical) to the single blank space → it slides smoothly into the blank.
- Only one tile can move per tap.
- No drag — tap only.

### Match feedback
- When a name tile and its emoji tile become adjacent, both tiles **glow** with their pair colour (pulsing box-shadow).
- The glow disappears if they are separated again.

### Undo
- The ↩ button reverses the most recent slide.
- Can be pressed repeatedly to undo multiple slides in reverse order.
- Disabled when no history exists.

### Win condition
All pairs simultaneously adjacent → win screen appears after a 350ms delay.

---

## Advanced Mode — Memory Mechanic

In Advanced mode, all tile faces are **hidden** at the start (transparent content, tile outlines only visible). The mechanic:

- **On each slide:** the moved tile becomes fully visible (age 4).
- **Each subsequent move:** all non-moving tiles fade by one step. After 4 more moves the tile is fully hidden again.
- **Matched pairs:** when a pair becomes adjacent and glows, both tiles are forced fully visible and stay visible for as long as they remain matched.
- **Age → opacity mapping:**

| Age | Opacity | Meaning |
|-----|---------|---------|
| 0 | 0% | Hidden (forgotten) |
| 1 | 25% | Fading fast |
| 2 | 50% | Fading |
| 3 | 75% | Recently moved |
| 4 | 100% | Just moved / matched |

- **Start guarantee:** no name+emoji pair starts adjacent — the puzzle would otherwise be trivially won by the glow appearing immediately.

---

## Level Generation

Levels are **randomly generated** on every game start.

### Algorithm
1. Shuffle the 14-member family pool, select `cfg.pairs` members.
2. Build a **solved state**: names placed in even rows (0, 2, 4…), emoji directly below in odd rows. This guarantees each pair is vertically adjacent in the solved state.
3. Fill remaining cells (scanning from the end): one blank cell, then wildcard tiles.
4. **Shuffle:** make K random valid slides from the solved state (never immediately reversing). Guaranteed solvable by construction.
5. **Safety check:** verify the result isn't accidentally solved; re-shuffle in small bursts if so.
6. **Advanced only:** verify no pair is adjacent at the start; re-shuffle if needed.

---

## Visual Design

### Theme
Volcanic Lanzarote. Dark lava-rock background, glowing coloured tile borders and pulse animations, fire-toned UI.

### Colour palette
| Pair index | Glow / border accent |
|------------|----------------------|
| 0 | #ff6600 orange |
| 1 | #ff3300 red-orange |
| 2 | #ffdd00 yellow |
| 3 | #00ccaa teal |
| 4 | #bb55ff purple |
| 5 | #88ff00 lime |
| 6 | #00aaff blue |
| 7 | #ff44aa pink |
| 8 | #ff9900 amber |
| 9 | #00ffaa mint |

App background: `#0d0400`. Tile background: `#180a02`. Tile border default: `#4a2010`.

---

## Screens

### 1 — Title
🌋 icon, "LANZAROTE FEVER" heading, "Family Slide" subtitle, brief game instructions, Play button.

### 2 — Difficulty
Four buttons: Easy / Medium / Hard / Advanced with short descriptions. Back button.

### 3 — Game
- **HUD** (top bar): ← back, difficulty · matched/total pairs, ↩ undo.
- **Grid**: fills available screen below HUD. Cell size computed to maximise grid within viewport (minimum 40px).
- **Grid padding:** 12px around the grid so match glow doesn't clip at the screen edge.
- Responsive to orientation change (220ms debounce, restarts current puzzle).

### 4 — Win
🎉 icon, "You got it!" heading, pair count subtitle, Next Puzzle (primary) and Change Difficulty (secondary) buttons.

---

## Responsive Layout

- **Mobile-only** design (no desktop breakpoints).
- Cell size = `min(availableWidth / cols, availableHeight / rows)`, minimum 40px.
- Accounts for 3px gap between cells and grid-wrap padding.

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

- Square photo avatars for all family members (currently partial coverage).
- Shared-seed multiplayer: same puzzle for multiple players via URL seed parameter (`?seed=abc123`), compare times asynchronously.
