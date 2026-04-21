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

## Core Product Decisions

These are the **best conservative selections** for the concept.

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
- **Beginner:** 9 × 9, 10 mines
- **Intermediate:** 16 × 16, 40 mines
- **Expert:** 30 × 16, 99 mines

No custom board editor is required in the initial version.

### 3. Input Model
The game should be **desktop-first** to remain faithful to the original.
- **Left click:** reveal tile
- **Right click:** place/remove flag
- **Chord action:** supported on revealed numbered tiles when surrounding flags match the number

Mobile-first control schemes are out of scope for the initial version.

### 4. First Click Safety
Adopt a **safe first click** rule.

Rationale:
- It is a minimal deviation from some original versions.
- It significantly improves fairness.
- It is widely accepted in modern Minesweeper implementations.
- It does not materially change the core game loop.

Conservative interpretation:
- The first revealed tile must never be a mine.
- No stronger guarantee is required initially.
- The implementation does **not** need to guarantee a zero-valued first click unless that falls out naturally from implementation.

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

Changing difficulty must generate a new board using the selected preset.

### FR-3: Board Generation
The game must generate a minefield for the selected difficulty.
- Mine count must match the preset.
- Grid dimensions must match the preset.
- The first revealed tile must be safe.

### FR-4: Tile Reveal
When a hidden, non-flagged tile is revealed:
- If it contains a mine, the game ends in loss.
- If it has one or more adjacent mines, its number is displayed.
- If it has zero adjacent mines, neighboring empty region tiles are revealed automatically.

### FR-5: Flagging
The player must be able to toggle a flag on hidden tiles.
- Flags prevent accidental reveal through standard click behavior.
- Flag count should affect the mine counter display in the classic way.

### FR-6: Chording
The player must be able to chord on a revealed numbered tile.
- If the number of adjacent flagged tiles equals the tile number, unrevealed adjacent non-flagged tiles are revealed.
- If any such revealed tile contains a mine, the game ends in loss.

### FR-7: Game End States
The game must support:
- **Win:** all non-mine tiles revealed
- **Loss:** mine revealed

On loss:
- mines should be revealed clearly
- incorrectly flagged tiles may be shown distinctly if desired

On win:
- the board should be visually frozen
- the timer should stop
- remaining unflagged mines may optionally auto-flag, but this is not required

### FR-8: Timer
The UI must include a timer.
- Timer starts on the first reveal.
- Timer stops on win or loss.
- Display format should be simple and classic in spirit.

### FR-9: Mine Counter
The UI must include a mine counter.
- Initial value matches total mine count for the board.
- Counter updates based on placed flags.
- Exact legacy behavior for negative counts is optional, but acceptable if implemented.

### FR-10: Reset Control
The UI must include a reset control modeled after the classic face button.
- Clicking it starts a new game using the current difficulty.
- The button may visually change state for normal / pressed / win / loss if desired.

### FR-11: Prevent Browser Friction
The app must suppress browser behaviors that interfere with play where reasonable.
- Right click on the board should not open the browser context menu.
- Input handling should feel immediate and predictable.

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

## Non-Goals

The following are explicitly out of scope for the first version:
- mobile-optimized controls
- accessibility deep pass beyond basic semantic care
- custom map creation
- solvability guarantees beyond safe first click
- no-guess puzzle generation
- save/load system
- statistics dashboard
- online competition
- daily puzzle rotation
- alternate skins or game themes

---

## Product Definition Summary

The first version should be understood as:

> A small, one-page, desktop-first browser version of classic Minesweeper with classic board sizes, classic controls, classic UI structure, safe first click, and no extra mechanics.

This is the baseline to protect during implementation. Any new idea should be rejected unless it clearly strengthens authenticity, clarity, or usability **without** moving the game away from classic Minesweeper.