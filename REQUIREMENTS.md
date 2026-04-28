# REQUIREMENTS.md

## Project

**Working title:** Minesweeper Clone  
**Format:** One-page web app  
**Scope:** Small, static-screen browser game  
**Design goal:** Stay as close as reasonably possible to classic Microsoft Minesweeper while keeping implementation simple and conservative.

---

## Product Direction

This project is a **faithful browser adaptation of classic Minesweeper**.

The guiding principle is:

> Reproduce the original game experience as closely as possible, while allowing only minimal concessions required for a clean modern web implementation.

This is **not** a reinterpretation, reskin, or feature expansion. It should feel immediately familiar to anyone who has played classic Minesweeper.

---

## Checkpoint: Player Goal

**The player must reveal all non-mine tiles on the board to win.**

- Flagging is a helper mechanism but not required to win.
- Winning condition: 100% of non-mine tiles revealed.
- Losing condition: any mine tile revealed.

---

## Checkpoint: Game Loop

1. **Initialization:** Player selects difficulty. Board renders as hidden. Timer shows "000". Mine counter shows total mines.
2. **First Click:** Player left-clicks. Mines are generated (clicked tile guaranteed safe). Board state becomes "active". Timer starts.
3. **Active Play:** Left-click reveals tiles. Right-click flags tiles. Board updates real-time. Timer increments.
4. **Win:** All non-mine tiles revealed → board freezes, timer stops, reset button shows "win" state.
5. **Loss:** Any mine revealed → all unrevealed mines shown, board freezes, timer stops, reset button shows "loss" state.
6. **Reset:** Click reset → new game starts with current difficulty.

---

## Core Product Decisions

These are the **best conservative selections** for the concept.

### 0. Reference Version
This implementation targets **Windows 95/98 Minesweeper** as the canonical reference.
- Board presets, control scheme, timer format, mine counter behavior, and flagging rules follow that version.
- When "classic" is referenced in this document, it means this version.
- Ambiguities should be resolved by testing against the reference version's behavior.

### 1. Gameplay Fidelity
The game must preserve the classic Minesweeper rule set:
- Hidden mines are distributed on the board.
- Revealing a mine causes immediate loss.
- Revealing a safe tile shows the number of adjacent mines.
- Revealing a tile with zero adjacent mines expands recursively / flood reveals neighboring empty tiles.
- The player wins by revealing all non-mine tiles.
- Flagging is available but not required for victory.

### 2. Board Presets
The game should include the classic difficulty presets:
- **Beginner:** 9 × 9 (width × height), 10 mines
- **Intermediate:** 16 × 16 (width × height), 40 mines
- **Expert:** 30 × 16 (width × height), 99 mines

*Convention: All dimensions are stated as width × height.*

No custom board editor is required in the initial version.

### 3. Input Model & Control Scheme (Checkpoint)

The game is **desktop-first**, mouse-driven, Windows 95/98 Minesweeper control scheme.

**Primary Controls:**
- **Left click on hidden tile:** Reveal tile. Mine = lose. Adjacent mines shown. Zero-adjacent triggers flood reveal.
- **Right click on hidden tile:** Toggle flag.
- **Left click on revealed numbered tile:** Chord action (if flagged-adjacent-count == number, reveal all non-flagged adjacent).
- **Left click on reset button:** New game, current difficulty.
- **Left click on difficulty selector:** Change difficulty (only before first click or after game end).

**Adjacency Definition (Critical):**
- **Mine counting on a tile:** **8 directions** (orthogonal + diagonal).
- **Flood reveal expansion:** **4 directions** (orthogonal only; no diagonals).
- **Chord action adjacency:** **8 directions**.

**Input Edge Cases:**
- Left-click on flagged tile: **no-op** (remains flagged).
- Right-click on revealed tile: **no-op** (cannot flag).
- Right-click outside board: browser context menu allowed.
- Double-click: not assigned (treat as two clicks).
- Keyboard: not supported in v1.

### 4. First Click Safety
Adopt a **safe first click** rule.

Rationale:
- It is a minimal deviation from some original versions.
- It significantly improves fairness.
- It is widely accepted in modern Minesweeper implementations.
- It does not materially change the core game loop.

Implementation detail:
- Mine placement is **deferred until after the first click**.
- The player clicks a hidden tile; the board is generated such that the clicked tile is safe (not a mine).
- This ensures no mine can ever occupy the first-clicked location, regardless of RNG.
- After the first click, the board is locked and no mines are relocated.

### 5. UI Faithfulness
The application should include the classic Minesweeper structural elements:
- mine counter
- timer
- reset button / face button
- bordered game frame
- board-centered layout

The UI should be **retro-inspired and recognizably classic**, but it does **not** need to be a pixel-perfect recreation of any specific Windows version.

### 6. Animation and Effects
Keep presentation minimal.
- Tile reveal should feel immediate.
- No elaborate animations.
- No particle effects.
- No audio required.

The original game feel depends on speed and clarity, not spectacle.

### 7. Feature Discipline
The initial version must **not** include:
- power-ups
- hint systems
- progression systems
- achievements
- leaderboards
- daily challenges
- story or narrative framing
- themes / skins
- multiple screens
- account systems
- ads / monetization mechanics

The app should remain a single focused puzzle experience.

---

## Functional Requirements

### FR-1: Single Page Layout
The game must run as a one-page web app.
- All gameplay occurs on a single screen.
- No navigation between views is required.
- Restarting and changing difficulty should happen within the same page.

### FR-2: Difficulty Selection
The user must be able to select one of the three classic difficulty presets:
- Beginner
- Intermediate
- Expert

Difficulty can be changed **only before first click or after game end** (win/loss). Clicking the reset button applies the current difficulty selection. Changing difficulty mid-game is not allowed; the player must complete or reset the current game first.

### FR-3: Board Generation
The game must generate a minefield for the selected difficulty.
- Mine count must match the preset.
- Grid dimensions must match the preset.
- The first revealed tile must be safe (not a mine); **this does NOT guarantee zero adjacent mines**.
- Before the first click, all tiles are rendered visually as hidden/unrevealed.

### FR-4: Tile Reveal
When a hidden, non-flagged tile is revealed:
- If it contains a mine, the game ends in loss.
- If it has one or more adjacent mines, its number is displayed.
- If it has zero adjacent mines, automatically reveal all orthogonally adjacent tiles (up, down, left, right) that are hidden and non-flagged. Repeat recursively for any newly-revealed zero-value tiles.
  - *Note: Orthogonal adjacency means 4 directions, not 8. Diagonals are NOT included in the flood reveal.*

### FR-5: Flagging
The player must be able to toggle a flag on hidden tiles only.
- Flagging is only allowed on tiles that are currently hidden (not yet revealed).
- Right-click on a hidden tile toggles the flag state: unflagged → flagged, flagged → unflagged.
- Right-click on a revealed tile is a **no-op** (cannot flag revealed tiles).
- Flags prevent accidental reveal through standard left-click behavior.
- Left-click on a flagged tile is a **no-op** (tile remains flagged; no reveal occurs).
- Flag count should affect the mine counter display (current mine count = total mines - flags placed).

### FR-6: Chording
The player must be able to chord on a revealed numbered tile.
- Chording is performed by clicking on a revealed tile that displays a number.
- If the count of **flagged** tiles orthogonally adjacent to the clicked tile equals the tile's number, then all unrevealed non-flagged tiles orthogonally adjacent to that tile are automatically revealed.
- If any of those revealed tiles contains a mine, the game ends in loss.
- *Note: Chord checks only the flag count, not flag correctness. If flags are placed on incorrect tiles but the count matches, the chord still fires.*

### FR-7: Game End States
The game must support:
- **Win:** all non-mine tiles have been revealed (flagged or unflagged state is irrelevant).
- **Loss:** a mine is revealed by any action (left-click or chord).

On loss:
- **All unrevealed mines must be revealed immediately.** Show them clearly so the player understands the board state.
- Optionally, incorrectly flagged tiles (flags placed on non-mine tiles) may be shown distinctly (e.g., crossed-out flag), but this is not required.
- The board becomes non-interactive: tiles cannot be clicked or flagged.

On win:
- The board becomes visually frozen / non-interactive.
- The timer stops immediately.
- All non-revealed tiles remain hidden (no auto-reveal of mines).
- The reset button should visually change to a "win" state if feasible (e.g., smiley face with sunglasses).

### FR-8: Timer
The UI must include a timer.
- **Before first click:** Timer displays "000".
- Timer starts incrementing on the **first left-click reveal** (not on flag placement or other interactions).
- Timer stops incrementing on win or loss.
- Timer counts in seconds, starting from 0.
- Timer caps at 999 seconds (displays "999" and does not increment further).
- Display format: simple numeric with leading zeros (e.g., "000", "123", "045"), no colon or "seconds" suffix needed.

### FR-9: Mine Counter
The UI must include a mine counter.
- Initial value matches the total mine count for the board.
- Counter formula: `total mines - flags placed`.
- Counter updates immediately when a flag is placed or removed.
- Counter must **never display a negative value**. If flags exceed mines (player places more flags than the mine count), the counter displays **0**.
- Display format: simple numeric (e.g., "010"), using leading zeros if needed to match classic display width.

### FR-10: Reset Control
The UI must include a reset control modeled after the classic face button.
- Clicking it immediately starts a new game using the current difficulty.
- Any in-progress game is discarded; the timer resets to "000".
- The button may visually change state for normal / pressed / win / loss if desired.

### FR-11: Prevent Browser Friction
The app must suppress browser behaviors that interfere with play where reasonable.
- Right-click on board tiles must not open the browser's context menu (use `preventDefault()` on `contextmenu` events within the game board).
- Left-click and right-click on tiles should be processed immediately without delay.
- Outside the board area (header, difficulty buttons, reset button), browser context menu is acceptable.

### FR-12: Keyboard Support
Keyboard support is **not required** in the initial version.
- The game must be fully playable using only mouse: left-click and right-click.
- Keyboard shortcuts (arrow keys, Enter, Space) are out of scope for v1 and may be added in future versions if desired.

---

## UX Requirements

### UX-1: Immediate Readability
The board must be easy to scan visually.
- Hidden, revealed, and flagged states must be clearly distinguishable.
- Number colors should follow classic conventions where practical.

### UX-2: Familiar Layout
The page should visually prioritize the game area.
- The board should be centered or otherwise prominently placed.
- The header area should contain classic status controls.
- The page should not contain distracting side panels or unrelated content.

### UX-3: Low Cognitive Overhead
The app should require no onboarding for players familiar with Minesweeper.
- No tutorial modal in the initial version.
- No settings drawer unless strictly necessary.

### UX-4: Responsive But Not Reimagined
The app may scale to fit common desktop viewport sizes, but should not redesign the game structure responsively in a way that changes its character.

---

## Visual Requirements

### VR-1: Classic-Inspired Styling
The UI should evoke classic Minesweeper.
Recommended cues:
- beveled panel edges
- flat gray or similarly restrained palette
- sharp square tiles
- digital-looking counter/timer style if practical

### VR-2: Conservative Modernization
Minor polish is acceptable, but must remain restrained.
Allowed:
- cleaner spacing
- slightly improved typography
- subtle hover feedback

Not allowed:
- flashy gradients
- oversized rounded cards
- playful mobile-game styling
- decorative thematic overlays

### VR-3: Number Presentation
Adjacent-mine numbers should use recognizable color coding aligned with player expectations.
Reference color scheme (Windows 95 Minesweeper):
- 1 = Blue
- 2 = Green
- 3 = Red
- 4 = Dark Blue
- 5 = Maroon
- 6 = Teal/Cyan
- 7 = Black
- 8 = Gray

*Modern web colors may be adapted for readability, but should maintain the recognizable intent of the classic scheme.*

### VR-4: Tile Visual States
Each tile must clearly display its state. Required states:
- **Hidden:** raised/beveled appearance, indicating an unrevealed tile (clickable)
- **Revealed (empty):** flat, pressed appearance; shows blank/empty (no number)
- **Revealed (numbered):** flat, pressed appearance; shows number 1–8 in the appropriate color
- **Flagged (hidden):** raised/beveled appearance with a flag icon or symbol overlaid
- **Mine (revealed on loss):** flat appearance with mine icon/symbol, clearly visible
- **Incorrectly flagged (on loss, optional):** flagged state with a visual distinction (e.g., crossed-out flag, different color)

Hover/active states are optional but recommended for better UX:
- Hidden tiles may appear slightly different on hover (e.g., brighter bevel) to indicate interactivity.
- No visual feedback is required when hovering over revealed or flagged tiles.

---

## Checkpoint: Browser Assumptions

**Supported Browsers:**
- Modern desktop browsers: Chrome, Firefox, Safari, Edge (ES6+ JavaScript).
- Desktop-only (mobile out of scope for v1).

**Required Capabilities:**
- DOM manipulation.
- CSS flexbox/grid for layout.
- JavaScript event handling (click, contextmenu).
- No canvas, WebGL, or advanced APIs required.
- LocalStorage not required (no persistence).

**Browser Behaviors Suppressed:**
- Right-click context menu on board tiles (preventDefault on contextmenu).
- Text selection on tiles (user-select: none in CSS).

**No Backend:**
- Static front-end only.
- No server, database, or authentication.
- All game state client-side and volatile.

---

## Technical Scope Requirements

### TS-1: Static Front-End App
The initial version should be front-end only.
- No backend required.
- No accounts.
- No cloud save.

### TS-2: Lightweight Implementation
The implementation should remain small and understandable.
- Prefer a simple architecture.
- Avoid unnecessary framework complexity.
- Keep game state logic explicit and testable.

### TS-3: Deterministic UI State
At any moment, the game should have a clear internal state:
- current difficulty
- board data
- revealed / hidden / flagged tile states
- timer state
- win/loss/active status

---

## Checkpoint: Acceptance Criteria

**Project Complete When:**

1. ✅ **Player Goal:** Player can win by revealing all non-mine tiles without hitting a mine.
2. ✅ **Game Loop:** Initialization → First Click → Active Play → Win/Loss → Reset is fully functional.
3. ✅ **All In-Scope Features:** FR-1 through FR-12 and all VR/UX requirements implemented and tested.
4. ✅ **Control Scheme:** Left-click/right-click work. Chord works. Reset/difficulty selector work. All no-ops behave correctly.
5. ✅ **Visual Requirements:** All 6 tile states visually distinct. Numbers use classic color scheme. UI is retro-inspired, board centered.
6. ✅ **Game State Correct:** Mine counter never negative. Timer displays correctly (000 before first click, increments, caps at 999). Win/loss detection correct.
7. ✅ **Browser Behavior:** Context menu suppressed on board. Game runs in modern desktop browsers. No backend.
8. ✅ **No Out-of-Scope:** Keyboard, mobile controls, save/load, audio, achievements absent.
9. ✅ **Code Quality:** Game state explicit and testable. Lightweight, simple architecture.
10. ✅ **QA Tested:** All three difficulties tested. Win/loss/timer/counter/flood/chord/reset all verified.

---

## Checkpoint: In-Scope Features (Committed)

**Core Gameplay:**
- Tile reveal with mine detection and flood reveal (FR-4)
- Flagging system (FR-5)
- Chord action (FR-6)
- Three difficulty presets: Beginner 9×9-10, Intermediate 16×16-40, Expert 30×16-99 (FR-2)
- Safe first click (FR-3)
- Win/loss detection and end-game states (FR-7)
- Timer: starts on first click, caps at 999s, displays with leading zeros (FR-8)
- Mine counter: total mines - flags, clamps to 0 (FR-9)
- Reset button (FR-10)
- Difficulty selector (before first click or after game end) (FR-2)
- Single-page layout (FR-1)
- Context menu suppression on board (FR-11)

**UI/UX:**
- Tile visual states: hidden, revealed-empty, revealed-numbered, flagged, mine, incorrectly-flagged (VR-4)
- Classic color scheme for numbers 1–8 (VR-3)
- Retro-inspired UI with beveled edges (VR-1, VR-2)
- Board centered and immediately readable (UX-1, UX-2)

## Checkpoint: Out-of-Scope Features (Explicitly Rejected)

- Keyboard shortcuts or navigation
- Mobile-optimized controls
- Custom map creation
- Save/load or persistence
- Statistics or leaderboards
- Accessibility deep pass
- Audio or elaborate animations
- Power-ups, hints, achievements
- Multiple screens, accounts, monetization
- Themes or skins

---

## Product Definition Summary

The first version should be understood as:

> A small, one-page, desktop-first browser version of classic Minesweeper with classic board sizes, classic controls, classic UI structure, safe first click, and no extra mechanics.

This is the baseline to protect during implementation. Any new idea should be rejected unless it clearly strengthens authenticity, clarity, or usability **without** moving the game away from classic Minesweeper.

## Project Setup Requirements

- The project shall include a `package.json` file in the repository root.
- The `package.json` file shall define the project's runnable commands in a consistent way.
- The `package.json` file shall include at least a `test` script so the same test command can be run every time.
- If the project uses ES module imports in JavaScript, `package.json` shall set `"type": "module"`.
- The project shall remain compatible with a plain JavaScript, static-site workflow.
