# SCREEN BRIEF 01: LEARN
## The Lesson Theater

**Prerequisites:** Read and absorb `00-design-foundation.md` before building this screen. All tokens, motion rules, and anti-patterns from that document are mandatory here.

---

## CONCEPT

The Learn screen is a **cinematic classroom**. The user is watching a chess lesson unfold as if it were a film — the board is the stage, the pieces are actors, and the lesson narrator is an invisible director guiding the camera. This is NOT a video player with a sidebar. This is NOT a text tutorial with a board widget. This is an immersive, step-by-step experience where the board IS the lesson.

**Metaphor:** A private screening room. One screen. One chair. Total focus.

---

## LAYOUT STRUCTURE

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   [Lesson Title — top left, minimal]     [Step X of Y]   │
│                                                          │
│         ┌─────────────────────────┐                      │
│         │                         │                      │
│         │                         │                      │
│         │      THREE.JS BOARD     │     NARRATION        │
│         │     (hero, 60% width)   │     PANEL            │
│         │                         │     (right side,     │
│         │                         │      30% width,      │
│         │                         │      vertically      │
│         └─────────────────────────┘      centered)       │
│                                                          │
│   ┌─────────────────────────────────────────────────┐    │
│   │  ← PREV  ·····●···  STEP INDICATOR  ···  NEXT →│    │
│   └─────────────────────────────────────────────────┘    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## COMPONENTS

### 1. Lesson Header (top edge)
- Lesson title: Secondary display font, --chrome, uppercase, letter-spacing 0.12em. Small — 14px.
- Lesson category tag: Mono font, --steel, e.g., "OPENING · SICILIAN DEFENSE"
- Step counter: Mono font, --steel, right-aligned. "STEP 04 / 12"
- The header is nearly invisible — it's metadata, not a navigation bar. Opacity 0.6, fades to 0.4 after 3 seconds of inactivity, returns on mouse move.

### 2. The Board (center-left)
- Three.js board from the foundation spec.
- Camera starts at a standard isometric view (30° elevation, 45° azimuth).
- As steps progress, the camera may dolly/orbit to focus on the relevant area of the board.
  - Example: If the lesson discusses a kingside attack, camera slowly orbits to show the kingside from the attacker's perspective (800ms, ease-out-expo).
- Pieces involved in the current step get a subtle rim glow (--chrome-highlight at 0.3 opacity).
- Pieces NOT involved in the current step dim slightly (reduce light intensity on their material by 30%).
- When a step involves a move, the piece animates to its new position per the foundation motion rules.
- Between steps, the board has the ambient "breathing" idle — very slow light orbit.

### 3. Narration Panel (right side)
- Background: --obsidian with a very subtle left-edge gradient fading from transparent to obsidian (the panel bleeds into the board space).
- Content structure per step:
  - **Move notation**: Mono font, --signal or --chrome, large (24px). e.g., "1. e4"
  - **Explanation**: Body font (Satoshi), --silver, 16px, line-height 1.6. 2–4 sentences max per step.
  - **Key insight callout** (optional): A single highlighted sentence with a left border in --signal (2px). Body font, --chrome, italic.
- The panel content transitions per step: current text fades out (150ms), new text fades in from below (300ms, 12px translate-y, ease-out-expo).
- Scrollable if content overflows, but design for no-scroll — brevity is a rule.

### 4. Step Navigation (bottom)
- A horizontal track of dots — one per step.
- Current step dot: --signal, slightly larger (8px vs 6px), with glow-signal.
- Completed steps: --chrome dots.
- Upcoming steps: --ash dots.
- "PREV" and "NEXT" labels flanking the dots: Ghost button style, secondary display font, --steel, uppercase.
- Keyboard navigation: Left/Right arrows. Space bar = next step.
- The step indicator sits in the lower third of the screen, with generous space below (--space-7).
- On step change: the dot transition is smooth — the glow slides from one dot to the next (200ms).

### 5. Quick Controls (floating, bottom-right)
- A small cluster of ghost icons (20px, --steel):
  - Flip board (rotate 180°)
  - Reset to start
  - Auto-play steps (toggles — when active, icon glows --signal)
  - Fullscreen toggle
- These appear on hover near the bottom-right corner. Hidden otherwise. Opacity transition 300ms.

---

## MOTION CHOREOGRAPHY

### First Load (entering the Learn screen)
1. Screen starts pure black.
2. A single key light fades up on the board (600ms, ease-out-expo). The board materializes from darkness.
3. Pieces appear with a staggered drop-in: back rank first, then pawns. Each piece falls into position with a micro-bounce (--ease-spring). Stagger: 40ms per piece, total ~1.3s.
4. The narration panel text fades in after the board is populated (200ms delay, 400ms fade).
5. Step indicator dots appear last, sliding up from below (200ms, staggered 30ms each).
6. Total entrance: ~2.5 seconds. Feels deliberate, not slow.

### Step Progression
1. Current narration text fades out + slides down 8px (150ms).
2. Board piece moves (if applicable) — 600ms with appropriate easing per piece type.
3. Camera adjusts if the new step has a different focus area (800ms, ease-out-expo).
4. New narration text fades in + slides up from 12px below (300ms, ease-out-expo, 100ms delay after move completes).
5. Step indicator dot transitions (200ms).

### Auto-play Mode
- Steps advance automatically every 4 seconds.
- A thin progress line animates along the bottom of the narration panel (4s linear, --signal).
- Clicking anywhere pauses auto-play.

---

## STATES

- **Default:** Board at isometric view, first step loaded, narration visible.
- **Mid-lesson:** Board may be at a non-default camera angle. Completed steps have chrome dots.
- **Lesson complete:** Final step shows a summary card overlaying the narration panel — "Lesson Complete" in display font, with stats (time spent, moves reviewed). A "PRACTICE THIS" call-to-action button in --signal links to the Practice screen. The board camera slowly pulls back to a wide shot.
- **Error/fallback:** If Three.js fails to load, render the 2D board (same colors, CSS grid) with piece icons. Narration and step navigation work identically.

---

## CONTENT MODEL

```typescript
interface Lesson {
  id: string;
  title: string;                    // e.g., "The Sicilian Defense: Najdorf Variation"
  category: string;                 // e.g., "OPENING"
  steps: Step[];
}

interface Step {
  moveNotation: string | null;      // e.g., "1. e4" or null for intro/summary steps
  fen: string;                      // Board position after this step
  explanation: string;              // 2–4 sentences
  insight?: string;                 // Optional highlighted callout
  cameraPosition?: CameraConfig;    // Optional camera override for this step
  highlightSquares?: string[];      // e.g., ["e4", "d5"] — squares to emphasize
  arrowFrom?: string;               // Optional arrow: from square
  arrowTo?: string;                 // Optional arrow: to square
}

interface CameraConfig {
  azimuth: number;      // degrees
  elevation: number;    // degrees
  distance: number;     // camera distance from center
  lookAt: [number, number, number]; // focal point
}
```

---

## ACCESSIBILITY

- All step text is screen-reader accessible.
- Board state described via aria-live region: "White pawn moves to e4."
- Keyboard navigation for all controls.
- High-contrast mode: Increase --silver to --white-hot for all body text.
- Reduced motion mode: Disable camera animation, piece animation instant, fade transitions only.
