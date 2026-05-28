# DESIGN FOUNDATION BRIEF
## Chess LMS — Art Direction & Source of Truth
### Codename: BLACK SQUARE

---

## 1. VISION STATEMENT

This is not a learning management system. This is a cinematic chess experience that teaches through atmosphere, tension, and reward. Every screen should feel like a scene — lit, composed, and scored. The product sits at the intersection of a luxury product showcase, a film title sequence, and a precision instrument.

Think: If Porsche Design built a chess academy inside a David Fincher film.

---

## 2. VISUAL IDENTITY

### 2.1 Color System

```
/* === CORE PALETTE === */

--void:            #000000;    /* True black — the canvas, the negative space */
--obsidian:        #0A0A0A;    /* Near-black for surfaces, cards, containers */
--graphite:        #141414;    /* Elevated surfaces, hover states */
--smoke:           #1E1E1E;    /* Subtle dividers, borders when needed */
--ash:             #2A2A2A;    /* Inactive elements, disabled states */
--steel:           #6B6B6B;    /* Secondary text, metadata, timestamps */
--silver:          #A0A0A0;    /* Body text on dark backgrounds */
--chrome:          #D4D4D4;    /* Primary text, high-emphasis labels */
--white-hot:       #F5F5F5;    /* Headlines, critical numbers, active states */

/* === ACCENT — SIGNAL RED === */
/* Used ONLY for: focus states, critical moves, danger, mastery moments, unlocks */

--signal:          #E63946;    /* Primary accent — the red rook */
--signal-glow:     #E6394640;  /* 25% opacity — for glows, halos */
--signal-deep:     #B71C1C;    /* Pressed/active state of accent */

/* === METALLIC TINTS (for gradients on 3D elements only) === */

--chrome-highlight: #FFFFFF;   /* Specular peak */
--chrome-body:      #C0C0C0;   /* Mid-tone metal */
--chrome-shadow:    #404040;   /* Metal in shadow */

/* === SEMANTIC === */

--success:         #4ADE80;    /* Correct move, completed lesson — used sparingly */
--caution:         #FBBF24;    /* Warning, time pressure */
--info:            #60A5FA;    /* Hints, annotations — very rare */
```

**COLOR RULES:**
- The background is ALWAYS --void or --obsidian. Never gray. Never blue-tinted dark. Pure darkness.
- Signal Red is surgical — never decorative. If you're using it on more than 2 elements per screen, you're overusing it.
- Text hierarchy is built from white-hot → chrome → silver → steel. Four levels, no more.
- Never use colored backgrounds for cards or containers. Elevation = subtle luminance shift (obsidian → graphite → smoke).
- Gradients are reserved for 3D metallic surfaces only. Flat UI elements stay flat.

### 2.2 Typography

```
/* === TYPE SYSTEM === */

Primary Display:    "PP Neue Machina"   /* Headlines, screen titles, numbers */
                    fallback: "Orbitron", "Rajdhani", sans-serif

Secondary Display:  "Monument Extended"  /* Labels, category names, navigation */
                    fallback: "Bebas Neue", "Oswald", sans-serif

Body:               "Satoshi"            /* Body text, descriptions, long-form */
                    fallback: "General Sans", "DM Sans", sans-serif

Mono:               "JetBrains Mono"     /* Chess notation, coordinates, data, code */
                    fallback: "Fira Code", "Source Code Pro", monospace
```

**TYPOGRAPHY RULES:**
- Headlines are UPPERCASE, letter-spacing: 0.08em–0.12em. Think engraved steel.
- Body text is sentence case, letter-spacing: 0.01em–0.02em. Comfortable reading.
- Chess notation (e4, Nf3, O-O) always uses the mono font at a slightly smaller size.
- Numbers in data/stats use the display font at large sizes — they are design elements.
- Maximum 2 font weights per typeface on a single screen. Restraint is luxury.
- Line height for body text: 1.6. For headlines: 1.0–1.1. Tight headlines, breathing body.

### 2.3 Spacing & Layout

```
/* === SPACING SCALE (8px base) === */

--space-1:   4px;     /* Micro — icon to label */
--space-2:   8px;     /* Tight — within components */
--space-3:   16px;    /* Default — between related elements */
--space-4:   24px;    /* Comfortable — between sections within a card */
--space-5:   32px;    /* Generous — between cards/components */
--space-6:   48px;    /* Dramatic — section breaks */
--space-7:   64px;    /* Cinematic — major section separation */
--space-8:   96px;    /* Stage — top/bottom page margins, hero spacing */

/* === LAYOUT === */

--content-max-width:  1440px;
--content-padding:    clamp(24px, 5vw, 80px);
--board-max-size:     min(70vh, 600px);  /* Chess board never overwhelms */
```

**LAYOUT RULES:**
- Asymmetric composition is preferred over centered. Push content left or right, let the board breathe.
- The chess board is always the gravitational center of any screen that contains one — everything else orbits it.
- Generous negative space on the right or bottom = cinematic letterboxing effect.
- No more than 3 columns at any breakpoint. Density kills luxury.
- Cards have no visible borders. Separation comes from spacing and subtle background shifts.

### 2.4 Elevation & Depth

```
/* === SHADOW SYSTEM === */
/* Shadows are warm/amber-tinted — simulating stage lighting from above */

--shadow-subtle:    0 1px 2px rgba(0,0,0,0.6);
--shadow-card:      0 4px 24px rgba(0,0,0,0.5), 0 1px 4px rgba(0,0,0,0.4);
--shadow-elevated:  0 8px 40px rgba(0,0,0,0.6), 0 2px 8px rgba(0,0,0,0.5);
--shadow-floating:  0 16px 64px rgba(0,0,0,0.7), 0 4px 16px rgba(0,0,0,0.5);

/* === GLOW EFFECTS (accent-driven) === */

--glow-signal:      0 0 20px var(--signal-glow), 0 0 60px rgba(230,57,70,0.1);
--glow-chrome:      0 0 15px rgba(255,255,255,0.08);
--glow-soft:        0 0 40px rgba(255,255,255,0.03);
```

**DEPTH RULES:**
- Depth is communicated through LIGHT, not shadow boxes. Objects closer to the viewer catch more light.
- The page is a dark stage. Elements are actors lit from above and slightly camera-left.
- Glow effects are reserved for interactive/active states and accent moments.
- Never stack more than 3 elevation levels on one screen.

---

## 3. MOTION LANGUAGE

### 3.1 Timing & Easing

```
/* === DURATION === */

--duration-instant:    80ms;    /* Micro-feedback: button press, toggle */
--duration-fast:       200ms;   /* State change: hover, focus */
--duration-normal:     400ms;   /* Component animation: card reveal, panel slide */
--duration-dramatic:   800ms;   /* Hero animation: piece move, screen transition */
--duration-cinematic:  1200ms;  /* Full cinematic moment: unlock reveal, scene change */
--duration-epic:       2000ms;  /* Rare: first load, major achievement, level-up */

/* === EASING === */

--ease-out-expo:       cubic-bezier(0.16, 1, 0.3, 1);      /* Primary — smooth deceleration */
--ease-in-out-quart:   cubic-bezier(0.76, 0, 0.24, 1);     /* Symmetric — for looping/breathing */
--ease-out-back:       cubic-bezier(0.34, 1.56, 0.64, 1);   /* Slight overshoot — for reveals */
--ease-spring:         cubic-bezier(0.22, 1.5, 0.36, 1);    /* Bouncy — chess pieces landing */
```

### 3.2 Motion Principles

**RULE 1 — OBJECTS HAVE WEIGHT**
Chess pieces don't teleport. They lift, glide, and land. A knight's move should have a slight arc. A rook slides linearly. Movement reflects the piece's character.

**RULE 2 — LIGHT MOVES, OBJECTS WAIT**
When transitioning between screens, the light changes first (fade/shift), then content appears. Think of stage lighting cues — the spot hits the mark before the actor enters.

**RULE 3 — ENTRANCE = CONFIDENT, EXIT = SWIFT**
Elements enter with deliberate, eased-out animation (400–800ms). Elements exit quickly with ease-in (150–200ms). Arrivals are celebrated. Departures are efficient.

**RULE 4 — STAGGER, DON'T SWARM**
When multiple elements appear, stagger by 60–80ms per item, from focal point outward. Never reveal everything at once. Never stagger more than 8 items — group after that.

**RULE 5 — THE BOARD BREATHES**
The 3D chess board should have an ambient idle animation: very slow, subtle rotation (0.5° over 10s) or a soft light orbit. The board is alive, waiting for input.

**RULE 6 — CINEMATIC CAMERA**
In 3D scenes (Three.js), transitions between states use camera movement, not cuts. Dolly in for detail, crane up for overview, orbit for perspective shifts. Duration: 800–1200ms.

### 3.3 Screen Transitions

Between the five core screens (Learn → Practice → Challenge → Report → Unlock):
- The transition is NOT a page navigation. It's a SCENE CHANGE.
- Current content fades to black (200ms).
- A brief hold on darkness (100ms) — the "beat."
- New content materializes from darkness (400ms, staggered).
- If a 3D board is present on both screens, it persists and the camera moves to a new position rather than fading out/in.

---

## 4. THREE.JS — 3D PIPELINE

### 4.1 Board & Pieces Specification

**BOARD:**
- 8×8 grid, each square is a separate mesh for individual interaction/highlight
- Light squares: Matte silver/aluminum material (roughness: 0.7, metalness: 0.9)
- Dark squares: Matte black/dark chrome (roughness: 0.6, metalness: 0.95)
- Board edge/frame: Polished chrome with chamfered edges
- Board sits on a subtle pedestal or floats in the void
- Total board should fill approx 60–70% of its container

**PIECES:**
- Material: High-polish chrome (roughness: 0.1, metalness: 1.0) for default pieces
- Highlighted/selected piece: Emissive rim glow (signal red or white)
- Opponent's pieces: Darker chrome or gunmetal finish (roughness: 0.3, metalness: 0.95)
- Piece silhouettes should be clean and readable at small sizes — favor simplified geometry over ornate detail
- Each piece type has a distinct mass/height ratio: king is tallest, pawn is shortest, but all share the same material language

**ENVIRONMENT:**
- Background: Pure black (no environment map on background)
- Lighting rig: 3-point cinematic setup
  - Key light: Warm white (slightly amber), 45° above, camera-left
  - Fill light: Cool white, 30° right, half intensity of key
  - Rim light: Pure white, behind and slightly above, creates edge highlights on chrome
- Optional: Soft area light from directly above for ambient fill
- Shadows: Soft shadow map, shadow-map-size 2048. Shadows ground the board.
- Reflection: Use environment map (dark studio HDRI) for chrome reflections, NOT scene reflections

### 4.2 Interaction Model

- Hovering a square: Subtle luminance increase on the square + piece glow
- Selecting a piece: Piece lifts 0.5 units upward (spring ease), possible valid squares highlight with a soft pulse
- Moving a piece: Piece glides along a path (arc for knight, line for others), duration 600ms, lands with a micro-bounce
- Capturing: Captured piece dissolves (scale down + fade, 300ms) as capturing piece arrives
- Check: Brief red pulse on the king's square, king gets a red rim glow
- Checkmate: Camera slowly dollies in toward the king, lights dim except a spot on the decisive position

### 4.3 Performance Targets

- 60fps on mid-range devices (target: GTX 1060 / M1 equivalent)
- Use instanced meshes for pawns where possible
- LOD not required (piece count is max 32, low poly is fine)
- Shadow map: Only from key light
- Post-processing: Tone mapping (ACES filmic), subtle bloom on specular highlights, optional film grain

---

## 5. COMPONENT DESIGN RULES

### 5.1 Buttons

- Primary: Background --signal, text --white-hot. No border radius > 4px. Hover: slight scale(1.02) + glow.
- Secondary: Background transparent, border 1px --smoke, text --chrome. Hover: background --graphite.
- Ghost: No background, no border. Text --silver. Hover: text --white-hot. Used for navigation.
- All buttons: height 48px minimum. Uppercase label, letter-spacing 0.1em, font: secondary display.
- Press: scale(0.98) + immediate color shift. Feel the click.

### 5.2 Cards / Containers

- Background: --obsidian or --graphite. NEVER use borders.
- Border-radius: 8px max. Prefer 4px or 0px. Sharp = precise = luxury.
- Inner padding: --space-4 (24px).
- Separation between cards: --space-5 (32px).
- Hover: Shift background one level lighter + glow-soft. No transform.

### 5.3 Progress Indicators

- Linear bars: 2px height, background --smoke, fill gradient from --steel to --chrome. At 100%: fill becomes --signal.
- Circular: SVG ring, 2px stroke, same color logic.
- Numbers > bars when possible. "14 / 20" is more luxurious than a progress bar.

### 5.4 Icons

- Style: Outlined, 1.5px stroke, no fill. Consistent with the wireframe/chrome aesthetic.
- Size: 20px default, 24px in navigation.
- Color: --steel default, --chrome on hover/active, --signal for alerts.
- Chess piece icons (when used as flat UI icons): Use Unicode chess symbols (♔♕♖♗♘♙♚♛♜♝♞♟) in the mono font, or custom SVG silhouettes.

### 5.5 Data Visualization

- Chart backgrounds: Transparent (inherit parent).
- Lines/bars: --chrome as default. --signal for the focal data point.
- Grid lines: --smoke at 0.3 opacity. Minimal. 3–4 lines max.
- Labels: --steel, mono font, small.
- Animate chart drawing on entrance: line draws left-to-right (800ms), bars grow upward (400ms staggered).

---

## 6. SOUND DESIGN DIRECTION (optional, for future reference)

If sound is implemented:
- Piece placement: Short, crisp tap — ceramic on stone.
- Piece capture: Same tap + a subtle low-frequency resonance.
- Check: A single, clear chime — high-pitched, metallic.
- Unlock/achievement: A rising tone sequence — think notification from a luxury car.
- Ambient: None in gameplay. Optional dark ambient drone in menus (very subtle).
- All sounds: dry, close-mic'd, minimal reverb. Not orchestral. Not 8-bit. Think ASMR meets precision engineering.

---

## 7. ANTI-PATTERNS — WHAT THIS IS NOT

Do NOT produce any of the following:

- ❌ Rounded, bubbly card UI with colored backgrounds (Duolingo aesthetic)
- ❌ Gamification with cartoon badges, streaks, or confetti
- ❌ Bright colored themes, gradient backgrounds, or pastel palettes
- ❌ Standard LMS layouts with sidebars, breadcrumbs, and hamburger menus
- ❌ Wood-textured chess boards or skeuomorphic pieces
- ❌ Green/brown chess.com or lichess visual language
- ❌ Drop shadows that feel like Material Design
- ❌ Rounded sans-serif everywhere (Inter, Poppins, Nunito)
- ❌ Emojis or playful icons as status indicators
- ❌ Cookie-cutter dashboards with widget grids
- ❌ Any element that could belong to "just another web app"

---

## 8. RESPONSIVE STRATEGY

- Desktop (1440px+): Full cinematic layout with 3D board and side panels.
- Laptop (1024–1439px): Board slightly smaller, panels below or overlayed.
- Tablet (768–1023px): Board takes full width, controls overlay from bottom.
- Mobile (< 768px): 2D board fallback (styled to match aesthetic), stacked layout, bottom navigation.
- The 3D experience degrades gracefully: if WebGL isn't available or performance is poor, render a 2D board using the same color/material tokens (dark squares, chrome highlights) via CSS.

---

## 9. FILE & COMPONENT NAMING

```
components/
  Board/             — Three.js chess board (3D)
  Board2D/           — CSS fallback board
  Piece/             — Individual piece component (3D)
  SceneLight/        — Lighting rig for Three.js
  
screens/
  LearnScreen/
  PracticeScreen/
  ChallengeScreen/
  ReportScreen/
  UnlockScreen/

shared/
  tokens.css         — All CSS variables from this brief
  motion.ts          — Easing functions, duration constants
  typography.css     — Font imports and type scale
```

---

## 10. BRIEF USAGE INSTRUCTIONS

This document is the SOURCE OF TRUTH. Every screen brief that follows will reference this foundation. When building any screen:

1. Import and apply the color tokens, type system, and spacing scale EXACTLY as defined here.
2. Follow the motion principles — test every animation against the 6 rules.
3. Maintain the cinematic lighting metaphor: dark stage, lit objects, controlled reveals.
4. When in doubt, remove elements. Negative space is a feature, not a problem.
5. Every screen should pass the "screenshot test" — if you took a still frame, would it look like a frame from a film, or a screenshot of a web app? Aim for the former.
