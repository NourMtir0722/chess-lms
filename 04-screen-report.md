# SCREEN BRIEF 04: REPORT
## The Mirror

**Prerequisites:** Read and absorb `00-design-foundation.md` before building this screen. All tokens, motion rules, and anti-patterns from that document are mandatory here.

---

## CONCEPT

The Report screen is a **performance mirror** — it shows the user who they are as a chess player with unflinching clarity and cinematic presentation. This is NOT a generic analytics dashboard with widgets. This is a curated editorial page that tells the story of the player's journey through data, like a luxury magazine spread about an athlete's season.

**Metaphor:** The post-fight analysis room. Calm. Honest. The adrenaline has passed. Now you study the tape.

---

## LAYOUT STRUCTURE

The Report is a single-column, vertically scrolling editorial page. No sidebars. No widget grids. Content is organized in "chapters" — each chapter is a full-width section separated by dramatic spacing (--space-8, 96px).

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│                                                            │
│         YOUR REPORT                                        │
│         May 2026                                           │
│                                                            │
│                                                            │
│═══════════════════════════════════════════════════════════  │
│                                                            │
│                                                            │
│   CHAPTER 1: THE NUMBER                                    │
│   ┌────────────────────────────┐                           │
│   │         1,487              │                           │
│   │    Current Rating          │                           │
│   │    ▲ +87 this month        │                           │
│   └────────────────────────────┘                           │
│                                                            │
│                                                            │
│═══════════════════════════════════════════════════════════  │
│                                                            │
│   CHAPTER 2: THE ARC                                       │
│   [Rating over time — line chart, full width]              │
│                                                            │
│═══════════════════════════════════════════════════════════  │
│                                                            │
│   CHAPTER 3: THE BREAKDOWN                                 │
│   [Category performance tiles]                             │
│                                                            │
│═══════════════════════════════════════════════════════════  │
│                                                            │
│   CHAPTER 4: THE REPLAY                                    │
│   [Key moments from recent games — board snapshots]        │
│                                                            │
│═══════════════════════════════════════════════════════════  │
│                                                            │
│   CHAPTER 5: THE PATH                                      │
│   [What to work on next — recommendations]                 │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## CHAPTERS (COMPONENTS)

### Chapter 1: THE NUMBER
The hero metric. One giant number dominates the screen.

- Rating number: Display font (PP Neue Machina), --white-hot, MASSIVE (96–120px on desktop, 64px on mobile). Centered or left-aligned.
- Label below: "CURRENT RATING" in secondary display font, --steel, 12px, uppercase, letter-spacing 0.15em.
- Change indicator: "▲ +87" or "▼ -12" in mono font, --success (if positive) or --signal (if negative), 18px. The triangle is a custom SVG, not a text character.
- Subtext: "THIS MONTH" in mono font, --ash, 11px.
- The number animates on load: counts up from 0 to the actual value over 1200ms (ease-out-expo). The counter should feel mechanical — incrementing by irregular jumps (not linear), simulating a precision instrument settling.

### Chapter 2: THE ARC
A single line chart showing rating over time. This is the emotional centerpiece.

- Full content width. Height: 300px.
- X-axis: Time (last 30 days, last 3 months, or all time — selectable via ghost tabs above the chart).
- Y-axis: Rating. Minimal grid lines — only 2–3 horizontal reference lines, --smoke at 0.2 opacity.
- The line: 2px, gradient from --steel (left/old) to --chrome (right/recent). The rightmost point (current) is a dot with --signal glow.
- Area fill below the line: Gradient from --signal-glow at the bottom to transparent. Very subtle — the chart should feel like a glowing horizon line.
- Hover interaction: A vertical crosshair line (--smoke, 1px) follows the cursor. At the intersection, a tooltip shows the date and exact rating in a small card (--graphite background, --chrome text, mono font).
- Animation on load: The line draws from left to right (800ms, ease-out-expo). The glow area fills in behind it.
- Key moments can be marked on the line: small dots at positions where the user played a Challenge game. Hover to see the result: "vs AI Master — Win +12".

### Chapter 3: THE BREAKDOWN
Performance by category. Clean tiles, not bar charts.

- A row of 3–4 tiles, each representing a skill area: TACTICS, OPENINGS, ENDGAME, STRATEGY.
- Each tile:
  - Background: --obsidian. No border.
  - Category label: Secondary display font, --steel, uppercase, 12px. Top of tile.
  - Score/percentage: Display font, --chrome, 36px. e.g., "78%"
  - Subtle ring chart behind the number — an SVG ring (60px diameter, 2px stroke). Filled portion: --chrome. Unfilled: --smoke at 0.3.
  - Trend arrow: Small "▲" or "▼" next to the score, --success or --signal.
  - Below: "42 puzzles solved" in body font, --steel, 13px.
- Tiles stagger in on scroll-enter: 80ms per tile, slide up from 16px + fade (400ms, ease-out-expo).
- Hover: Tile background shifts to --graphite. The ring chart stroke thickens to 3px.

### Chapter 4: THE REPLAY
Key moments from recent games — the "highlight reel."

- A vertical stack of 2–3 "moment cards." Each card shows:
  - A 2D mini-board (not Three.js — a CSS grid rendering of the position, using the dark/chrome square colors from the design tokens). Board size: ~200px, left-aligned within the card.
  - To the right of the board:
    - Game context: "vs THE STRATEGIST — Game 7" in secondary display font, --steel, 12px.
    - Move: "23. Nf7+!" in mono font, --signal (if brilliant) or --chrome (if solid), 20px. The "!" or "!!" is part of the annotation.
    - Annotation: "Knight fork wins the queen — a tactic from your Lesson 4 training." Body font, --silver, 14px. 1–2 sentences.
    - Type badge: "BRILLIANT" or "BLUNDER" or "TURNING POINT" in mono font, 11px, uppercase. --signal background at 10% with --signal text for brilliant. --caution for blunders.
  - Card background: --obsidian. Subtle left-border accent (2px, --signal for brilliant, --caution for blunder, --chrome for neutral).
  - Cards stagger in on scroll: 120ms apart, slide up 20px + fade (500ms).

### Chapter 5: THE PATH
Forward-looking recommendations. What to study/practice next.

- Header: "WHAT TO WORK ON" in secondary display font, --chrome, 16px, uppercase.
- 2–3 recommendation items, each a minimal row:
  - Left: An icon or piece symbol (e.g., ♞ for knight tactics) in mono font, --signal, 24px.
  - Center: Recommendation text in body font, --chrome, 16px. e.g., "Your knight tactics accuracy dropped 12% this week. Practice knight forks."
  - Right: A ghost CTA — "PRACTICE →" or "LEARN →" in secondary display font, --steel, 12px. Links to the relevant Practice or Learn screen.
- These recommendations are algorithmically generated based on performance data.
- Subtle connecting line (vertical, 1px, --smoke) between items, like a timeline.
- The last item has a special treatment: "NEXT UNLOCK" with a lock icon (--signal), showing how close they are to the next achievement. e.g., "3 more perfect puzzle sets to unlock Master Tactician."

---

## SCROLL BEHAVIOR

- The page scrolls smoothly. No snap-scrolling — it should feel like browsing a magazine.
- Each chapter fades in on scroll-enter (IntersectionObserver, threshold: 0.2). Content slides up from 24px + fades over 500ms.
- Between chapters: A thin horizontal divider (1px, --smoke, 50% width, centered) with --space-8 above and below.
- Parallax: The hero rating number has a slight parallax — it scrolls at 0.8x speed relative to the rest. Subtle, not nauseating.

---

## TIME PERIOD SELECTION

- A set of ghost tabs at the very top (below the header): "7D" "30D" "3M" "ALL" — mono font, --steel, 13px.
- Selected tab: --chrome with a bottom underline (2px, --signal).
- Switching periods: All data on the page transitions — numbers count to new values (400ms), chart redraws (600ms), tiles update.
- The whole page reacts to the period change cohesively, not component-by-component.

---

## MOTION CHOREOGRAPHY

### First Load
1. "YOUR REPORT" title fades in (400ms).
2. The hero rating counts up from 0 (1200ms, ease-out-expo).
3. Change indicator slides in from right (200ms delay, 300ms, ease-out-expo).
4. As the user scrolls, each subsequent chapter animates on enter.

### Period Switch
1. All visible numbers "blur" briefly (a 100ms text blur filter, then resolve to new values).
2. Chart line retracts to left (200ms), redraws with new data (600ms).
3. Tiles crossfade their content (200ms).

---

## STATES

- **Data available:** Full report renders with all chapters.
- **Insufficient data:** If the user hasn't played enough (e.g., first day), show a minimal version — the rating number + "Play 5 more games to unlock your full report" in body font, --steel. The empty state should still be beautiful — use the negative space.
- **Loading:** A skeleton version — dark rectangles where content will be, pulsing gently (--smoke to --graphite, 2s cycle). No spinner.

---

## CONTENT MODEL

```typescript
interface Report {
  currentRating: number;
  ratingChange: number;           // +87 or -12 for the selected period
  ratingHistory: RatingPoint[];   // Time series data for the chart
  categories: CategoryScore[];
  keyMoments: KeyMoment[];
  recommendations: Recommendation[];
  period: '7d' | '30d' | '3m' | 'all';
}

interface RatingPoint {
  date: string;                   // ISO date
  rating: number;
  gameId?: string;                // If this point corresponds to a game
  result?: 'win' | 'loss' | 'draw';
}

interface CategoryScore {
  category: 'tactics' | 'openings' | 'endgame' | 'strategy';
  score: number;                  // 0–100 percentage
  trend: number;                  // +5 or -12 for the period
  puzzlesSolved: number;
}

interface KeyMoment {
  gameId: string;
  opponentName: string;
  moveNumber: number;
  notation: string;
  fen: string;                    // Position at this moment
  annotation: string;
  type: 'brilliant' | 'blunder' | 'turning_point';
}

interface Recommendation {
  icon: string;                   // Chess piece unicode or category icon
  text: string;
  actionType: 'practice' | 'learn';
  actionTarget: string;           // Lesson or puzzle set ID
}
```

---

## ACCESSIBILITY

- All chart data available as a table for screen readers (visually hidden).
- Rating change announced: "Your rating is 1,487, up 87 points this month."
- Scroll-triggered animations respect prefers-reduced-motion — all content visible immediately, no transforms.
- Charts have aria-labels describing the trend in plain language.
