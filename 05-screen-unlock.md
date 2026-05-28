# SCREEN BRIEF 05: UNLOCK
## The Vault

**Prerequisites:** Read and absorb `00-design-foundation.md` before building this screen. All tokens, motion rules, and anti-patterns from that document are mandatory here.

---

## CONCEPT

The Unlock screen is the **reward ceremony**. When a user masters a concept, completes a milestone, or earns an achievement, they don't get a pop-up notification with confetti. They get a SCENE. A cinematic reveal. A moment that says: this mattered. The piece they unlocked materializes from darkness like an artifact being unveiled in a museum.

This screen has two modes: the **Unlock Moment** (the reveal animation when an achievement is freshly earned) and the **Collection View** (browsing all unlocked/locked achievements).

**Metaphor:** A private vault. Each achievement is an object in a display case. When you earn a new one, the case opens, the lights come on, and the object rotates under the spotlight.

---

## MODE 1: THE UNLOCK MOMENT (Full-screen reveal)

This triggers automatically when the user earns a new achievement. It takes over the screen.

### Sequence (total duration: ~4 seconds)

**Frame 1 — The Darkness (0ms–500ms)**
- Screen goes to pure --void black. All UI disappears.
- A beat of silence. Nothing happens. This is intentional — the pause creates anticipation.

**Frame 2 — The Light (500ms–1200ms)**
- A single point of light appears at the center of the screen — like a star igniting.
- The light is warm white, starting as a 4px dot and expanding to a soft circular gradient (200px radius, --chrome-highlight center, fading to transparent edges). Duration: 700ms, ease-out-expo.
- The light reveals a 3D object beneath it.

**Frame 3 — The Object (1200ms–2500ms)**
- A Three.js 3D chess piece (or abstract trophy shape) materializes.
- The piece starts at scale 0, rotates, and scales up to full size (ease-out-back, 800ms). It's chrome, high-polish, catching the light from above.
- The piece continues to slowly rotate on a turntable (one full rotation every 12 seconds).
- The lighting is the cinematic 3-point rig from the foundation, but with the key light directly above (creating top-down museum spot lighting).
- A subtle ambient particle effect: 5–8 tiny specular dots floating slowly upward around the piece, like dust in a spotlight beam. (Simple Three.js particle system — white dots, 1–2px, random slow drift upward, fade at edges.)

**Frame 4 — The Title (2500ms–3500ms)**
- Below the 3D piece, text fades in:
  - Achievement title: Display font, --white-hot, 28px, uppercase, letter-spacing 0.15em. e.g., "MASTER TACTICIAN"
  - Earned by: Body font, --silver, 15px. e.g., "10 perfect puzzle sets completed."
  - Date earned: Mono font, --steel, 12px. e.g., "MAY 28, 2026"
- Text entrance: Fade in + slide up from 16px, staggered 100ms per line, 400ms each.

**Frame 5 — The Actions (3500ms–4000ms)**
- Two buttons fade in at the bottom:
  - "CONTINUE" — Primary button, --signal. Returns to the previous screen.
  - "VIEW COLLECTION" — Ghost button, --steel. Goes to Mode 2.
- Buttons enter with fade + slide up (300ms).

### The 3D Object per Achievement Type

Each achievement category gets a distinct 3D object:

| Achievement Category | 3D Object | Material |
|---------------------|-----------|----------|
| Tactics Mastery | Knight piece (rearing) | Chrome, --signal emissive rim |
| Opening Knowledge | Book/codex shape | Brushed dark metal |
| Endgame Proficiency | King piece (tall, commanding) | Polished chrome |
| Rating Milestone | Abstract trophy/obelisk | Mirror chrome with gold tint |
| Streak Achievement | Chain links (interlocked) | Chrome with slight iridescence |
| Challenge Victory | Crown shape | Chrome with --signal inlay |
| Completion | Checkmark/shield | Matte silver |

If custom 3D models aren't available, use the standard chess piece that best represents the category, with a material variation to distinguish it from in-game pieces.

---

## MODE 2: THE COLLECTION (Gallery view)

### Layout

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│   YOUR COLLECTION                               12 / 30   │
│                                                            │
│   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐      │
│   │      │  │      │  │      │  │ LOCK │  │ LOCK │       │
│   │  ◆   │  │  ◆   │  │  ◆   │  │  🔒  │  │  🔒  │       │
│   │      │  │      │  │      │  │      │  │      │       │
│   │Title │  │Title │  │Title │  │ ???  │  │ ???  │       │
│   └──────┘  └──────┘  └──────┘  └──────┘  └──────┘       │
│                                                            │
│   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐      │
│   │ LOCK │  │ LOCK │  │ LOCK │  │ LOCK │  │ LOCK │       │
│   │  🔒  │  │  🔒  │  │  🔒  │  │  🔒  │  │  🔒  │       │
│   │      │  │      │  │      │  │      │  │      │       │
│   │ ???  │  │ ???  │  │ ???  │  │ ???  │  │ ???  │       │
│   └──────┘  └──────┘  └──────┘  └──────┘  └──────┘       │
│                                                            │
│                                                            │
│   ═══════════════════════════════════════════════          │
│   NEXT UNLOCK                                              │
│   "Win 3 more games to earn CHALLENGER"                    │
│   [████████░░░░] 7/10                                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Components

**Header:**
- "YOUR COLLECTION" — Secondary display font, --chrome, uppercase, 16px, letter-spacing 0.12em.
- Counter: "12 / 30" — Mono font, --steel, right-aligned. Shows earned vs total.

**Achievement Grid:**
- Grid of cards: 5 columns on desktop, 3 on tablet, 2 on mobile. Gap: --space-5 (32px).
- Each card: Aspect ratio 3:4 (portrait). Background: --obsidian. Border-radius: 4px.

**Unlocked Card:**
- A small 3D object preview (Three.js, simplified — lower poly, single light, no particles) OR a 2D icon silhouette rendered in --chrome.
- The object has a very slow idle rotation (one rotation per 20 seconds).
- Below the object: Achievement title in secondary display font, --chrome, 12px, uppercase.
- Date earned: Mono font, --ash, 10px.
- Hover: Card background shifts to --graphite, object rotation speeds up slightly, a subtle --glow-chrome appears.
- Click: Opens a detail overlay (see below).

**Locked Card:**
- Background: --obsidian, slightly darker than unlocked.
- Center: A lock icon — custom SVG, --ash, 24px. Minimal. A simple padlock silhouette with clean lines.
- Below: "???" in secondary display font, --ash, 12px. (Or the achievement name revealed but dimmed, depending on game design choice — revealing names creates aspiration.)
- Below that (optional): A one-line condition in body font, --steel, 10px. e.g., "Complete 5 endgame lessons"
- Hover: Lock icon subtly pulses (0.3 → 0.5 opacity, 2s cycle). No other change — locked is locked.
- No click action — or click could show the requirement in a tooltip.

**Achievement Detail Overlay (when clicking an unlocked card):**
- A centered modal card, max-width 480px. Background: --obsidian. Shadow: --shadow-floating.
- Contains:
  - The 3D object, larger, with full lighting rig and slow rotation. Height: 240px.
  - Title: Display font, --white-hot, 24px.
  - Description: Body font, --silver, 15px. 1–2 sentences. e.g., "Awarded for solving 10 tactical puzzle sets without error."
  - Date: Mono font, --steel, 12px.
  - Rarity (optional): "12% of players have earned this" in mono font, --steel, 11px.
  - Close: "×" in top-right corner, ghost style, or click outside to dismiss.
- Entrance: Fade in + scale from 0.95 to 1.0 (300ms, ease-out-expo). Background overlay: --void at 80% opacity.
- Exit: Fade out (200ms).

**Next Unlock Section (bottom of collection):**
- Separated by a thin divider (1px, --smoke, 50% width, centered).
- "NEXT UNLOCK" in secondary display font, --chrome, 14px, uppercase.
- The achievement being worked toward: Name in display font, --steel, 18px.
- Progress: A linear bar (2px height, --smoke track, --chrome fill). Glows --signal when > 80% complete.
- Counter: "7 / 10" in mono font, --chrome, right of the bar.
- Condition text: Body font, --steel, 13px. e.g., "Win 3 more challenge games."
- CTA: "GO →" ghost button, --steel. Links to the relevant activity (Practice, Challenge, Learn).

---

## MOTION CHOREOGRAPHY

### Collection Load
1. "YOUR COLLECTION" title fades in (300ms).
2. Counter counts up from 0 to total earned (400ms, mono font tick effect).
3. Cards stagger in: Row by row, 60ms per card, slide up from 16px + fade (400ms, ease-out-expo).
4. Unlocked cards start their idle rotation after entering.
5. Next Unlock section fades in last (200ms delay after final card).

### Achievement Earned (transition from Unlock Moment to Collection)
- If the user clicks "VIEW COLLECTION" after an Unlock Moment:
  1. The 3D object from the reveal shrinks and flies to its position in the grid (800ms, ease-in-out-quart). The camera follows.
  2. The rest of the collection fades in around it (400ms, staggered).
  3. The newly unlocked card has a --signal border glow that fades over 3 seconds (marking it as "new").

---

## ACHIEVEMENT TIERS

Achievements can have tiers (Bronze → Silver → Gold → Platinum). Each tier has a visual distinction:

| Tier | Material Treatment | Ring/Border |
|------|-------------------|-------------|
| Bronze | Warm dark chrome (slight amber tint) | None |
| Silver | Standard chrome | Thin --smoke ring |
| Gold | Chrome with warm highlight (subtle gold in specular) | Thin --caution ring |
| Platinum | Mirror chrome with slight iridescence | --signal glow ring |

Tiers are subtle material differences, NOT colored badges or cartoon medals. The luxury is in the surface quality, not the decoration.

---

## STATES

- **Fresh unlock:** Unlock Moment plays (Mode 1), then option to view Collection.
- **Collection browsing:** Grid view with all achievements.
- **Detail viewing:** Overlay open on a specific achievement.
- **Empty collection:** First-time user. "YOUR JOURNEY BEGINS" in display font, --steel. Show the first locked achievements as aspirational targets. No sad empty state — frame it as potential.
- **All unlocked:** A special treatment when 100% complete — all cards have a subtle ambient glow, and a "COLLECTION COMPLETE" banner appears at the top in --signal.

---

## CONTENT MODEL

```typescript
interface Achievement {
  id: string;
  title: string;                    // e.g., "MASTER TACTICIAN"
  description: string;              // Full description
  category: 'tactics' | 'openings' | 'endgame' | 'rating' | 'streak' | 'challenge' | 'completion';
  tier: 'bronze' | 'silver' | 'gold' | 'platinum';
  condition: string;                // Human-readable requirement
  progress: number;                 // 0–1 (percentage complete)
  target: number;                   // e.g., 10 (total needed)
  current: number;                  // e.g., 7 (current count)
  unlocked: boolean;
  unlockedDate?: string;            // ISO date
  rarity?: number;                  // Percentage of players who have this
  model3d: string;                  // Reference to 3D object type
}

interface UnlockEvent {
  achievementId: string;
  triggeredFrom: 'practice' | 'challenge' | 'learn';   // Which screen triggered the unlock
  timestamp: string;
}
```

---

## ACCESSIBILITY

- Unlock Moment: Achievement title and description announced via aria-live immediately (don't wait for animation to complete).
- Grid: Keyboard-navigable with arrow keys. Enter opens detail overlay.
- Locked items: "Locked. Requirement: complete 5 endgame lessons."
- Reduced motion: Unlock Moment simplified — 3D object fades in at full size (no scaling), no particles, title appears simultaneously. Collection cards appear without stagger.
