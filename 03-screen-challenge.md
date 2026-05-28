# SCREEN BRIEF 03: CHALLENGE
## The Arena

**Prerequisites:** Read and absorb `00-design-foundation.md` before building this screen. All tokens, motion rules, and anti-patterns from that document are mandatory here.

---

## CONCEPT

The Challenge screen is where the user plays a real game — against AI or another player. This is the **performance stage**. Everything built in Learn and Practice leads here. The atmosphere should shift: the lighting tightens, the UI recedes even further, and the board becomes everything. This is tension, clock pressure, and consequence.

**Metaphor:** A boxing ring. The overhead lights. The crowd goes silent. It's just you and the opponent across 64 squares.

---

## LAYOUT STRUCTURE

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│   ┌─ OPPONENT INFO ──────────┐    ┌─ CLOCK ─────────┐     │
│   │ ●  AI MASTER (1800)      │    │     12:45        │     │
│   │    ████████░░  8 captured │    │                  │     │
│   └──────────────────────────┘    └──────────────────┘     │
│                                                            │
│              ┌─────────────────────────┐                   │
│              │                         │                   │
│              │      THREE.JS BOARD     │   MOVE            │
│              │     (centered, hero)    │   HISTORY         │
│              │                         │   (scrolling      │
│              │                         │    column,        │
│              │                         │    right side)    │
│              └─────────────────────────┘                   │
│                                                            │
│   ┌─ PLAYER INFO ────────────┐    ┌─ CLOCK ─────────┐     │
│   │ ●  YOU (1450)            │    │     14:22        │     │
│   │    ██████░░░░  4 captured │    │                  │     │
│   └──────────────────────────┘    └──────────────────┘     │
│                                                            │
│   [RESIGN]  [OFFER DRAW]                   [SETTINGS ⚙]   │
│                                            (sound, board)  │
└────────────────────────────────────────────────────────────┘
```

---

## COMPONENTS

### 1. Player Info Bars (top = opponent, bottom = you)
- Each bar is a horizontal strip, left-aligned, minimal.
- **Avatar/indicator:** A small circle (12px). Opponent: --steel. You: --signal when it's your turn, --chrome otherwise.
- **Name:** Secondary display font, --chrome, uppercase, 14px. e.g., "AI MASTER" or player name.
- **Rating:** Mono font, --steel, in parentheses. e.g., "(1800)"
- **Captured pieces:** Tiny piece silhouettes (12px, Unicode or SVG) showing pieces this player has captured. Displayed in a row, --steel. When a new capture happens, the piece icon appears with a scale-in animation (200ms, ease-out-back).
- **Material advantage:** If one side has material advantage, show a "+3" or similar in mono font, --chrome, next to captured pieces.
- The opponent bar is at the top (far side of the board), the player bar at the bottom (near side). This mirrors the over-the-board perspective.

### 2. Clocks (flanking the player bars)
- Right-aligned, vertically aligned with their respective player bar.
- Time display: Display font (PP Neue Machina), large (28px), --chrome.
- **Active clock** (whose turn it is): --white-hot, slightly brighter, with a very subtle breathing glow (2s cycle).
- **Inactive clock:** --steel, dimmer.
- **Time pressure (< 1 minute):** Clock turns --signal. Numbers pulse gently with each second. At < 10 seconds: faster pulse, glow-signal active.
- **Time increment** (if applicable): Small "+5s" label below the clock in mono font, --ash.
- Clock transition on move: The active clock instantly deactivates (color shifts in 100ms), the other activates.

### 3. The Board (center)
- Three.js board, same spec as foundation.
- Camera: Fixed at player's perspective (white at bottom if playing white). 20° elevation — slightly higher than practice for a more commanding view.
- **No camera movement during the game.** The fixed camera creates stability and focus. Camera movement is reserved for end-of-game moments.
- Interaction: Same as Practice — hover glow, click-to-select, click-to-place, drag supported.
- **Last move highlight:** The two squares involved in the most recent move have a subtle --chrome outline (1px, 0.3 opacity). Not distracting, but traceable.
- **Pre-move:** If the user clicks during opponent's turn, the move is queued. Pre-move squares highlight with a dashed --steel outline. Pre-move executes instantly when it becomes the user's turn.
- **Check indicator:** When a king is in check, the king's square gets a --signal glow pulse (slow, 3s cycle). The square itself has a --signal tint at 10% opacity.

### 4. Move History (right side panel)
- A scrolling column, right of the board. Width: ~200px.
- Background: --obsidian, no border. Left edge has a 1px --smoke divider line.
- Moves displayed in algebraic notation, two columns (white | black):
  ```
  1.  e4      e5
  2.  Nf3     Nc6
  3.  Bb5     a6
  ```
- Font: Mono (JetBrains Mono), --silver, 14px.
- The most recent move is highlighted: --chrome text with a subtle left border --signal (2px).
- Auto-scrolls to the latest move.
- Clicking a previous move: Temporarily shows that position on the board (board dims to 50% opacity to indicate "review mode"). Click "LIVE" button or make a move to return to current position.
- On mobile: Move history is hidden behind a pull-up drawer from the bottom.

### 5. Game Actions (bottom edge)
- **RESIGN:** Ghost button, --steel, left side. On click: Confirmation overlay — "Resign this game?" with "YES" (--signal) and "NO" (ghost) buttons. The overlay is a centered card, --obsidian background, shadow-elevated.
- **OFFER DRAW:** Ghost button, --steel, next to resign. Sends draw offer to opponent. Button state changes to "DRAW OFFERED" (--steel, disabled look) while pending. If declined: button returns to default, a brief text "Draw declined" appears and fades (2s).
- **SETTINGS:** Ghost icon (gear), bottom-right. Opens a small popover: toggle sound, toggle board coordinates (a-h, 1-8), toggle pre-move, toggle move animation speed.

---

## MOTION CHOREOGRAPHY

### Game Start
1. The screen loads with the board already visible (no dramatic entrance — the player chose to enter this game, so the entry should feel like sitting down at the table).
2. Player bars slide in from left/right (300ms, ease-out-expo).
3. Clocks appear with a fade (200ms).
4. If playing white, the board is immediately interactive. If black, the opponent's first move plays automatically after a brief pause (1–2s for AI thinking simulation).

### During Play
- **Player's move:** Piece glides to destination (500ms, ease-spring). Clock switch (100ms). Move appears in history (fade-in, 200ms). If capture: captured piece dissolves (200ms) and appears in the capturer's piece row (scale-in, 200ms).
- **Opponent's move:** Brief pause (500ms–2s for AI "thinking"), then piece moves identically. The opponent's clock was running during the pause.
- **Check:** King's square pulses --signal. A single subtle "ping" in the lighting — the key light briefly intensifies by 10% and returns (300ms). No dramatic effects — check is normal in chess.
- **Castling:** Both king and rook move simultaneously (king first, rook follows 100ms later). Same duration.
- **En passant:** Capturing pawn moves diagonally, captured pawn dissolves from its actual square (not the destination).
- **Promotion:** When a pawn reaches the last rank, a promotion selector appears — 4 piece options (Queen, Rook, Bishop, Knight) displayed as chrome 3D pieces in a horizontal row above the board. Hover highlights with glow. Click selects. The pawn transforms (scale down, scale up as new piece, 400ms). Selector appears with a fast scale-in (200ms, ease-out-back).

### Game End
- **Checkmate:**
  1. The decisive move plays normally.
  2. A 500ms pause — the "realization."
  3. All lights dim except a spotlight on the mated king position. (Lighting transition: 800ms, ease-in-out-quart.)
  4. Camera slowly dollies in toward the mated king (1200ms, ease-out-expo).
  5. "CHECKMATE" appears centered over the board: Display font, --white-hot, 32px, letter-spacing 0.2em. Fades in with a subtle scale (0.95 → 1.0, 600ms).
  6. Below: "You win" or "You lose" in body font, --chrome or --steel respectively, 16px. Rating change: "+12" in --success or "-8" in --signal. Mono font.
  7. Action buttons appear (800ms delay): "REVIEW GAME" (secondary) and "PLAY AGAIN" (primary, --signal).

- **Resignation / Draw / Timeout:**
  1. Same spotlight/dim effect as checkmate but less dramatic — general dim, no camera move.
  2. Result text appears: "DRAW BY AGREEMENT", "RESIGNED", "TIME FORFEIT" in display font, --chrome, 24px.
  3. Same action buttons.

- **Stalemate / Insufficient Material:**
  1. Board goes to a neutral state — all pieces at normal light, no spotlights.
  2. "DRAW" text appears, --steel, display font, with the reason below in body font.

---

## AI OPPONENT PERSONALITY

When playing against AI, the opponent can have different "personalities" that affect thinking time and play style. These are cosmetic but add character:

- **Novice:** Quick moves (0.5–1s think time), occasional blunders. Name: "THE STUDENT"
- **Tactician:** Medium think time (1–3s), plays sharp tactical lines. Name: "THE TACTICIAN"
- **Positional Master:** Longer think time (2–5s), grinds positional advantages. Name: "THE STRATEGIST"
- **Grandmaster:** Variable think time (1–8s), strong moves. Name: "THE GRANDMASTER"

The opponent name appears in the opponent info bar. The AI thinking is simulated — the actual computation is instant, but the delay creates the illusion of thought.

---

## STATES

- **Pre-game:** Opponent info loading, clocks ready, board set up. Brief "GAME STARTING" text that fades.
- **Active play:** Full interactive state.
- **Opponent thinking:** Board non-interactive (it's not your turn), opponent clock running, subtle "thinking" indicator — three small dots pulsing next to opponent name.
- **Review mode:** Triggered by clicking a past move in history. Board shows historical position, dimmed. "LIVE" button appears above board.
- **Game over:** Result overlay, action buttons, board frozen at final position.

---

## CONTENT MODEL

```typescript
interface Game {
  id: string;
  playerWhite: Player;
  playerBlack: Player;
  timeControl: TimeControl;
  moves: GameMove[];
  result?: GameResult;
  pgn: string;
}

interface Player {
  name: string;
  rating: number;
  isHuman: boolean;
  aiPersonality?: 'novice' | 'tactician' | 'strategist' | 'grandmaster';
}

interface TimeControl {
  baseMinutes: number;          // e.g., 15
  incrementSeconds: number;     // e.g., 5
}

interface GameMove {
  moveNumber: number;
  notation: string;             // e.g., "Nf3"
  fen: string;                  // Position after this move
  timeRemainingMs: number;      // Clock time after this move
  isCheck: boolean;
  isCapture: boolean;
  capturedPiece?: string;       // e.g., "queen"
}

interface GameResult {
  winner: 'white' | 'black' | 'draw';
  method: 'checkmate' | 'resignation' | 'timeout' | 'stalemate' | 'draw_agreement' | 'insufficient_material';
  ratingChange: number;         // +12 or -8
}
```

---

## ACCESSIBILITY

- Active clock announced: "Your clock: 14 minutes 22 seconds."
- Opponent move announced: "Opponent plays Knight to f3."
- Check announced: "Check. Your king is in check."
- Game result announced with rating change.
- All game actions keyboard-accessible.
- Reduced motion: No camera dolly on checkmate, instant piece moves, fades only.
