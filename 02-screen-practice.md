# SCREEN BRIEF 02: PRACTICE
## The Training Chamber

**Prerequisites:** Read and absorb `00-design-foundation.md` before building this screen. All tokens, motion rules, and anti-patterns from that document are mandatory here.

---

## CONCEPT

The Practice screen is a **focused puzzle environment**. The user is given a board position and must find the best move (or sequence of moves). The UI strips down to near-nothing — just you, the board, and the puzzle. It feels like a training montage: intense, rhythmic, and rewarding.

**Metaphor:** A boxer hitting the heavy bag in a dark gym. Spot-lit. Alone. Focused.

The key UX difference from Learn: here the user is ACTIVE, not observing. The board is interactive. The narration is replaced by minimal prompts and immediate feedback.

---

## LAYOUT STRUCTURE

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│   [PUZZLE 14 / 50]                     [streak: 7] [timer]│
│                                                            │
│                                                            │
│              ┌─────────────────────────┐                   │
│              │                         │                   │
│              │                         │                   │
│              │      THREE.JS BOARD     │                   │
│              │     (centered, large)   │                   │
│              │                         │                   │
│              │                         │                   │
│              └─────────────────────────┘                   │
│                                                            │
│              ┌─────────────────────────┐                   │
│              │   PROMPT / FEEDBACK     │                   │
│              │   "White to move.       │                   │
│              │    Find the best move." │                   │
│              └─────────────────────────┘                   │
│                                                            │
│         [HINT]                              [SKIP →]       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## COMPONENTS

### 1. Status Bar (top edge)
- Puzzle counter: Mono font, --steel, left-aligned. "PUZZLE 14 / 50"
- Streak indicator: A small counter showing consecutive correct puzzles. Mono font.
  - When streak = 0: --steel, invisible/0.3 opacity.
  - When streak > 0: --chrome, visible. At streak ≥ 5: --signal with subtle glow.
  - Streak number animates on change: scale pop (1.0 → 1.3 → 1.0, 300ms, ease-out-back).
- Timer (optional, only in timed mode): Mono font, large numbers (20px), --chrome. Counts up by default, countdown in challenge practice. When < 10 seconds remaining: --signal, pulsing glow.
- The status bar is ultra-minimal. Same fade behavior as Learn: reduces opacity after inactivity.

### 2. The Board (centered, dominant)
- Three.js board, centered both horizontally and vertically in the viewport.
- Larger than in Learn mode — the board IS the screen. Target: 65–70% of viewport height.
- Camera: Straight-on view from the player's side (white at bottom by default). Slight 15° elevation for depth. No auto-orbit — static and focused.
- Board orientation: Auto-flips based on whose turn it is (white to move = white at bottom).
- **Interaction:** Full piece dragging or click-to-select, click-to-place.
  - On hover over a valid piece: Piece glows subtly (chrome highlight intensity +20%).
  - On piece selection (click): Piece lifts up 0.4 units. Valid destination squares pulse with a soft --chrome glow (breathing animation, 2s cycle).
  - On move: Piece glides to destination (600ms, ease-spring for the landing).
  - Wrong move: Board briefly flashes (a single frame of --signal at 5% opacity over the whole scene), piece returns to its original square (400ms, ease-out-expo). Prompt text updates.
  - Correct move: A sharp, clean burst — a thin ring of light expands from the destination square outward and fades (400ms). Captured piece (if any) dissolves. The "correct" indicator appears.
  - If the puzzle is a multi-move sequence: After the user's correct move, the opponent's response animates automatically (600ms), then prompt updates for the next move.

### 3. Prompt / Feedback Zone (below board)
- Centered below the board, max-width same as board.
- **Default state — Prompt:**
  - "WHITE TO MOVE" or "BLACK TO MOVE": Secondary display font, --chrome, uppercase, 14px, letter-spacing 0.12em.
  - Below: "Find the best move." in body font, --silver, 16px.
- **After wrong move — Feedback:**
  - Text transitions: prompt slides out (150ms), feedback slides in (300ms).
  - "Not quite. Try again." in body font, --steel, 16px. No drama — just a quiet nudge.
  - Optional: If hint is available, "HINT" button glows gently to draw attention.
- **After correct move — Success:**
  - "CORRECT" in display font, --signal (or --success for variation), 18px, letter-spacing 0.15em.
  - Below: The correct move notation in mono font, --chrome. e.g., "Nf7+ was the move."
  - Brief explanation (1 sentence, body font, --silver) if available. e.g., "The knight fork wins the queen."
  - After 1.5s: Auto-advance to next puzzle OR show "NEXT →" button.
- **After puzzle complete — Summary micro-card:**
  - Appears in the prompt zone: Time taken (mono, --chrome), difficulty rating (e.g., "1400 ELO"), correct/incorrect indicator.
  - Fades in (300ms), stays for 2s or until user clicks next.

### 4. Hint System
- "HINT" button: Ghost style, --steel, bottom-left. Uppercase, small.
- On click:
  - Level 1 hint: The correct piece to move gets a soft glow (--signal-glow pulse). No text. Just a visual nudge.
  - Level 2 hint (second click): The destination square also pulses. An arrow draws on the board from piece to square (SVG overlay, --signal at 40% opacity, 600ms draw animation).
  - Level 3 hint (third click): The narration zone shows the answer + explanation. Puzzle is marked as "solved with hints" (different from clean solve in the Report).
- Each hint level has a cost — the streak resets to 0 on any hint use. This should be communicated subtly (streak counter dims when hint is used).

### 5. Skip Button
- "SKIP →" ghost button, bottom-right, --steel.
- Skips to the next puzzle. Counts as incomplete in the report.
- No confirmation dialog — skipping should be frictionless.

### 6. Difficulty Indicator (subtle)
- A small badge near the puzzle counter or inline: "1200 ELO" or "INTERMEDIATE" in mono font, --steel, 12px.
- Not prominent — the user shouldn't feel judged by the number during practice.

---

## MOTION CHOREOGRAPHY

### Puzzle Load (entering a new puzzle)
1. Board is already present (it persists across puzzles — never re-renders the full board).
2. Pieces rearrange: All pieces simultaneously glide to their new positions (400ms, ease-out-expo). Pieces leaving the board scale down + fade (200ms). New pieces scale up from 0 (300ms, ease-out-back).
3. Prompt text fades in (200ms delay, 300ms fade).
4. Total transition between puzzles: ~700ms. Fast enough to maintain rhythm.

### Correct Answer Sequence
1. Piece lands at destination (600ms, ease-spring).
2. Light burst from destination square (400ms).
3. Prompt zone transitions to "CORRECT" (300ms).
4. Streak counter increments with scale pop (300ms, ease-out-back).
5. Hold for 1.5s.
6. Auto-advance: next puzzle loads (piece rearrangement, 400ms).

### Wrong Answer Sequence
1. Board flash — single frame red tint (80ms).
2. Piece returns to origin (400ms, ease-out-expo). No bounce — the return is chastened, not playful.
3. Prompt updates to "Not quite" (200ms fade swap).
4. Board resets to interactive state immediately.

### Streak Milestone (every 5 correct)
- At 5, 10, 15, etc.: The streak counter does a larger scale pop + brief --signal glow on the counter. A thin horizontal line sweeps across the bottom of the screen left-to-right (400ms, --signal at 30% opacity). Subtle. Not gamified.

---

## PRACTICE MODES

The Practice screen supports multiple modes, selectable before entering. The layout stays the same; the behavior and rules change:

1. **Free Practice** — Unlimited puzzles, no timer, no pressure. Default mode.
2. **Timed Blitz** — 60 seconds total, solve as many as possible. Timer counts down prominently. At 10s: timer turns --signal and pulses.
3. **Precision Mode** — 10 puzzles, no timer, but only ONE attempt per puzzle. Forces careful calculation. No hints available.
4. **Themed Set** — Puzzles filtered by tactic type (fork, pin, skewer, discovered attack, etc.). The category label appears in the status bar.

Mode selection happens BEFORE entering this screen (on a selection overlay or the parent navigation). This screen always opens directly into the active mode.

---

## STATES

- **Awaiting input:** Board interactive, prompt showing, hint/skip visible.
- **Move animating:** Board non-interactive for 600ms during piece animation.
- **Feedback showing:** Prompt zone shows result, board non-interactive briefly.
- **Puzzle transition:** Pieces rearranging, prompt fading, brief non-interactive window.
- **Session complete:** After final puzzle, the prompt zone expands into a session summary card: total puzzles, accuracy %, average time, streak record. "VIEW REPORT" CTA in --signal links to the Report screen. "PRACTICE MORE" secondary button.

---

## CONTENT MODEL

```typescript
interface Puzzle {
  id: string;
  fen: string;                    // Starting position
  sideToMove: 'white' | 'black';
  solution: SolutionMove[];       // The correct move sequence
  difficulty: number;             // ELO rating
  theme: string[];                // e.g., ["fork", "knight"]
  explanation?: string;           // Brief explanation of the tactic
  hints: Hint[];
}

interface SolutionMove {
  from: string;                   // e.g., "g5"
  to: string;                     // e.g., "f7"
  notation: string;               // e.g., "Nf7+"
  isPlayerMove: boolean;          // true = user must make this, false = auto-play response
}

interface Hint {
  level: number;                  // 1, 2, or 3
  type: 'highlight_piece' | 'show_destination' | 'show_answer';
  data: string;                   // square or move notation
}
```

---

## ACCESSIBILITY

- Board is keyboard-navigable: arrow keys move focus between squares, Enter to select/place.
- Screen reader: "Puzzle 14 of 50. White to move. Your streak is 7."
- Wrong move: aria-live announces "Incorrect. Try again."
- Correct move: aria-live announces "Correct. Knight to f7, check. Forking the king and queen."
- Reduced motion: Piece moves are instant, flash effects disabled, fades only.
