# BRIEF 00B: THREE.JS CHESS KIT
## The Foundry — 3D Component Library

**Prerequisites:** Read and absorb `00-design-foundation.md` before building. This brief builds the shared 3D component library that all screens consume. Nothing in this brief renders a full screen — it produces reusable, composable parts.

---

## PURPOSE

Build a self-contained Three.js chess kit that exports:

1. **ChessScene** — The master scene wrapper (renderer, camera, lighting rig, post-processing)
2. **ChessBoard** — The 8×8 board with individually addressable squares
3. **ChessPiece** — Each of the 6 piece types × 2 color variants
4. **ChessArrow** — Board annotation arrows
5. **ChessParticles** — Ambient particle effects (for Unlock screen)
6. **Presets** — Camera presets, lighting moods, and interaction modes

The kit must be framework-agnostic at its core (pure Three.js), with a React wrapper layer (using @react-three/fiber + @react-three/drei) for integration into the screen components.

---

## 1. CHESS SCENE (Master Wrapper)

### Renderer Configuration

```typescript
// Core renderer settings
{
  antialias: true,
  alpha: true,                        // Transparent background — page CSS handles the void
  toneMapping: THREE.ACESFilmicToneMapping,
  toneMappingExposure: 1.0,
  outputColorSpace: THREE.SRGBColorSpace,
  shadowMap: {
    enabled: true,
    type: THREE.PCFSoftShadowMap,
  },
  pixelRatio: Math.min(window.devicePixelRatio, 2),  // Cap at 2x for performance
}
```

### Post-Processing Stack

Apply in this order:
1. **Bloom** — Subtle, on specular highlights only. Threshold: 0.85, strength: 0.3, radius: 0.5. This makes chrome edges pop without washing out the scene.
2. **Vignette** — Very subtle darkening at screen edges. Offset: 0.3, darkness: 0.6. Creates the cinematic letterbox feel.
3. **Film Grain** (optional) — Extremely subtle. Noise intensity: 0.015. Only visible on close inspection. Adds analog texture to the digital chrome. Can be toggled off for performance.

Do NOT add: motion blur, chromatic aberration, depth of field (these fight the clean precision aesthetic).

### Camera

```typescript
// Default camera — adjusted per screen via presets
{
  type: THREE.PerspectiveCamera,
  fov: 35,                           // Narrow FOV = less distortion, more telephoto feel
  near: 0.1,
  far: 100,
  position: [0, 8, 10],              // Default isometric-ish view
  lookAt: [0, 0, 0],                 // Center of board
}
```

Camera transitions between presets use smooth interpolation:
- Position: Lerp with ease-out-expo, duration from preset config.
- LookAt: Slerp (spherical interpolation) for smooth focal point shifts.
- FOV: Lerp (for dramatic dolly-zoom effects, rarely used).

### Camera Presets

```typescript
const CAMERA_PRESETS = {

  // Standard isometric — used in Learn screen
  isometric: {
    position: [6, 8, 6],
    lookAt: [0, 0, 0],
    fov: 35,
    transitionDuration: 800,
  },

  // Player's view — used in Practice and Challenge
  player_white: {
    position: [0, 6, 8],
    lookAt: [0, 0, 0],
    fov: 35,
    transitionDuration: 800,
  },

  player_black: {
    position: [0, 6, -8],
    lookAt: [0, 0, 0],
    fov: 35,
    transitionDuration: 800,
  },

  // Dramatic close-up — used for checkmate, key moments
  closeup_kingside: {
    position: [4, 3, 4],
    lookAt: [2, 0, 0],
    fov: 40,
    transitionDuration: 1200,
  },

  closeup_center: {
    position: [0, 3, 3],
    lookAt: [0, 0, 0],
    fov: 40,
    transitionDuration: 1200,
  },

  // Overhead — bird's eye, used for Report mini-boards
  overhead: {
    position: [0, 12, 0.01],         // Slight z offset to avoid gimbal lock
    lookAt: [0, 0, 0],
    fov: 30,
    transitionDuration: 600,
  },

  // Trophy/unlock view — centered, slightly below eye level
  showcase: {
    position: [0, 2, 5],
    lookAt: [0, 1.5, 0],
    fov: 35,
    transitionDuration: 1000,
  },

  // Wide pull-back — used for lesson complete, end states
  wide: {
    position: [8, 10, 8],
    lookAt: [0, 0, 0],
    fov: 30,
    transitionDuration: 1200,
  },
};
```

---

## 2. LIGHTING RIG

The lighting rig is the single most important element for achieving the moodboard aesthetic. Every light has a purpose.

### Rig: "STUDIO" (Default for gameplay)

```typescript
const LIGHTING_STUDIO = {

  // KEY LIGHT — primary illumination, camera-left, slightly warm
  key: {
    type: THREE.DirectionalLight,
    color: 0xFFF5E6,                   // Warm white (subtle amber)
    intensity: 2.5,
    position: [-5, 10, 5],
    castShadow: true,
    shadow: {
      mapSize: { width: 2048, height: 2048 },
      camera: { left: -6, right: 6, top: 6, bottom: -6, near: 1, far: 30 },
      bias: -0.001,
      normalBias: 0.02,
      radius: 4,                       // Soft shadow edges
    },
  },

  // FILL LIGHT — opposite side, cooler, softer
  fill: {
    type: THREE.DirectionalLight,
    color: 0xE6F0FF,                   // Cool white (subtle blue)
    intensity: 0.8,
    position: [4, 6, -3],
    castShadow: false,
  },

  // RIM LIGHT — behind and above, pure white, creates edge definition on chrome
  rim: {
    type: THREE.DirectionalLight,
    color: 0xFFFFFF,
    intensity: 1.8,
    position: [0, 8, -8],
    castShadow: false,
  },

  // AMBIENT — very low, prevents pure black shadows
  ambient: {
    type: THREE.AmbientLight,
    color: 0x404040,
    intensity: 0.3,
  },

  // ENVIRONMENT MAP — for chrome reflections
  environment: {
    type: 'hdri',
    source: 'studio_dark',             // A dark studio HDRI — mostly black with a few soft area lights
    intensity: 0.5,                    // Subtle reflections, not mirror-bright
    // If HDRI not available, use a programmatic environment:
    fallback: {
      type: THREE.PMREMGenerator,       // Generate from a simple scene with dark walls and 2 area lights
    },
  },
};
```

### Rig: "SPOTLIGHT" (For Unlock reveals)

```typescript
const LIGHTING_SPOTLIGHT = {

  // Single top-down spot — museum exhibit lighting
  spot: {
    type: THREE.SpotLight,
    color: 0xFFFFFF,
    intensity: 4.0,
    position: [0, 10, 0],
    angle: Math.PI / 6,               // 30° cone
    penumbra: 0.6,                     // Soft edge falloff
    decay: 1.5,
    castShadow: true,
    shadow: {
      mapSize: { width: 2048, height: 2048 },
      bias: -0.001,
    },
    target: [0, 0, 0],                // Points straight down at the object
  },

  // Very faint fill to see the object's silhouette
  ambient: {
    type: THREE.AmbientLight,
    color: 0x202020,
    intensity: 0.15,
  },

  // Same environment map but much dimmer
  environment: {
    type: 'hdri',
    source: 'studio_dark',
    intensity: 0.2,
  },
};
```

### Rig: "DRAMATIC" (For checkmate, tension moments)

```typescript
const LIGHTING_DRAMATIC = {
  // Same as STUDIO but:
  // - Key light intensity reduced to 1.5
  // - Rim light intensity boosted to 2.5
  // - Ambient reduced to 0.1
  // - A new red-tinted point light appears near the focal area

  accent: {
    type: THREE.PointLight,
    color: 0xE63946,                   // --signal red
    intensity: 1.0,
    position: [0, 3, 0],              // Will be repositioned to the focal square
    distance: 6,
    decay: 2,
  },
};
```

### Lighting Transitions

When switching between rigs (or moods within a rig):
- Fade light intensities using lerp over 800ms (ease-in-out-quart).
- Color shifts use color lerp (THREE.Color.lerp).
- Never cut between lighting states — always crossfade.
- Shadow-casting lights should not be added/removed mid-scene (performance). Instead, set intensity to 0.

---

## 3. CHESS BOARD

### Geometry

- Board is 8×8 grid, centered at world origin (0, 0, 0).
- Each square is 1×1 unit. Total board: 8×8 units, from (-4, 0, -4) to (4, 0, 4).
- Each square is a separate mesh (THREE.PlaneGeometry rotated -90° on X, or BoxGeometry with very shallow height: 0.05 units).
- Using BoxGeometry with 0.05 height is preferred — gives squares a physical edge and catches rim light better.
- Squares are indexed: a1 = bottom-left when white is at bottom (position [-3.5, 0.025, 3.5]).

### Board Frame

- A surrounding frame mesh: BoxGeometry, 0.3 units wide, 0.1 units tall, running around the board perimeter.
- Material: Polished chrome (see materials below). The frame catches light beautifully and grounds the board.
- Optional: Board coordinates (a–h, 1–8) embossed or as floating text geometry on the frame edges. Mono font, --steel color, very small (0.15 unit height). Only visible when camera is close.

### Board Materials

```typescript
const MATERIALS = {

  // Light squares
  squareLight: {
    type: THREE.MeshStandardMaterial,
    color: 0xB0B0B0,                   // Matte silver
    roughness: 0.7,
    metalness: 0.9,
    envMapIntensity: 0.4,
  },

  // Dark squares
  squareDark: {
    type: THREE.MeshStandardMaterial,
    color: 0x1A1A1A,                   // Near-black
    roughness: 0.6,
    metalness: 0.95,
    envMapIntensity: 0.5,
  },

  // Board frame
  frame: {
    type: THREE.MeshStandardMaterial,
    color: 0xC0C0C0,                   // Chrome
    roughness: 0.15,
    metalness: 1.0,
    envMapIntensity: 0.8,
  },
};
```

### Square States

Each square can be in one of these visual states (applied via material property overrides, NOT by swapping materials):

| State | Visual Change |
|-------|--------------|
| `default` | Base material, no modification |
| `hovered` | Emissive: 0x222222 (subtle luminance increase) |
| `selected` | Emissive: 0x333333, border glow (see below) |
| `valid_move` | Emissive pulse: 0x1A1A1A ↔ 0x333333, breathing animation (2s cycle, ease-in-out-quart) |
| `last_move` | Emissive: 0x111111 (very subtle trace) |
| `check` | Emissive: pulse 0x000000 ↔ 0x3A0A0A (red tint, 3s cycle) |
| `highlight` | Emissive: 0x1A1A1A with --signal tint at low opacity |
| `premove` | Dashed outline effect (shader-based or overlay plane with dashed texture) |

**Square border glow:** For selected/check states, add a thin plane (0.02 units tall) around the square edge with an emissive material. This creates a "lit from within" border effect without using outlines.

### Board API

```typescript
interface ChessBoardAPI {
  // Set position from FEN string — animates pieces to new positions
  setPosition(fen: string, animate?: boolean): void;

  // Highlight specific squares
  setSquareState(square: string, state: SquareState): void;
  clearSquareStates(): void;

  // Draw an arrow between squares
  drawArrow(from: string, to: string, color?: string): void;
  clearArrows(): void;

  // Board orientation
  flipBoard(animated?: boolean): void;   // 180° rotation, 600ms

  // Get square world position (for camera targeting)
  getSquarePosition(square: string): THREE.Vector3;

  // Enable/disable interaction
  setInteractive(enabled: boolean): void;
}
```

---

## 4. CHESS PIECES

### Geometry Options (choose one approach)

**OPTION A — Procedural Geometry (Recommended for consistency)**
Build each piece from primitive Three.js geometries (cylinders, spheres, tori, cones, lathe geometry). This keeps the kit self-contained with no external model dependencies.

Piece construction guide:

| Piece | Construction |
|-------|-------------|
| **Pawn** | Sphere (head) + CylinderGeometry (neck) + Lathe (body curve) + CylinderGeometry (base). Total height: 0.6 units. |
| **Rook** | BoxGeometry (battlements, 3 notches) + CylinderGeometry (tower body) + wider CylinderGeometry (base). Height: 0.75 units. |
| **Knight** | LatheGeometry (body) + custom ExtrudeGeometry (horse head silhouette from SVG path). Height: 0.8 units. The knight is the most complex — a simplified angular head reads well at small sizes. |
| **Bishop** | Sphere (tip) + ConeGeometry (mitre) + CylinderGeometry (body) + Lathe (base). Height: 0.85 units. A slit in the mitre (boolean subtract or texture) distinguishes it. |
| **Queen** | TorusGeometry (crown ring with points) + CylinderGeometry (body) + Lathe (base). Height: 0.9 units. The crown should have 5–8 points — can be done with ConeGeometry instances arranged radially. |
| **King** | Cross on top (two intersecting BoxGeometry) + CylinderGeometry (body) + Lathe (base). Height: 1.0 units (tallest piece). |

All pieces share:
- A common base profile (wide base, narrow waist). The base diameter is 0.7 units (fills most of a square with margin).
- Smooth geometry — use sufficient segments (24+ for cylinders/lathes).
- A center origin at the base bottom — so piece.position.y = 0 places them flush on the board.

**OPTION B — External GLB/GLTF Models**
Load pre-modeled pieces from .glb files. Higher fidelity but requires asset management. If using this approach:
- Models should be low-poly (500–1500 triangles per piece).
- Models must have clean UVs but textures are not needed (material is applied programmatically).
- All models should share the same origin convention (center-bottom).
- Use GLTFLoader with DRACOLoader for compression.

### Piece Materials

```typescript
// Player's pieces (default: white side)
const MATERIAL_PLAYER = {
  type: THREE.MeshStandardMaterial,
  color: 0xE0E0E0,                     // Bright chrome
  roughness: 0.1,                       // High polish
  metalness: 1.0,
  envMapIntensity: 1.0,                 // Strong reflections
};

// Opponent's pieces (default: black side)
const MATERIAL_OPPONENT = {
  type: THREE.MeshStandardMaterial,
  color: 0x2A2A2A,                     // Dark gunmetal
  roughness: 0.3,                       // Slightly rougher
  metalness: 0.95,
  envMapIntensity: 0.7,
};

// Highlighted/selected piece — clone base material and add emissive
const MATERIAL_SELECTED = {
  // Clone from player or opponent material, then:
  emissive: 0xFFFFFF,
  emissiveIntensity: 0.15,             // Subtle glow
};

// Piece under check (king)
const MATERIAL_CHECK = {
  // Clone from base material, then:
  emissive: 0xE63946,                  // --signal red
  emissiveIntensity: 0.3,              // Animates: 0.1 ↔ 0.3 over 3s
};

// Piece in focus during lesson (Learn screen)
const MATERIAL_FOCUS = {
  // Clone from base material, then:
  emissive: 0xFFFFFF,
  emissiveIntensity: 0.08,             // Very subtle "this piece matters" glow
};

// Dimmed pieces (not relevant to current lesson step)
const MATERIAL_DIMMED = {
  // Clone from base material, then reduce:
  envMapIntensity: 0.3,                // Less reflective
  // Also reduce key light contribution via layers or custom shader
};
```

### Piece Animation System

```typescript
interface PieceAnimator {

  // Move a piece from one square to another
  movePiece(
    piece: ChessPiece,
    from: string,                      // e.g., "e2"
    to: string,                        // e.g., "e4"
    options?: {
      duration?: number,               // Default: 600ms
      easing?: EasingFunction,         // Default: ease-spring
      arc?: boolean,                   // True for knight moves (parabolic arc)
      arcHeight?: number,              // Default: 1.5 units for knights
      onComplete?: () => void,
    }
  ): void;

  // Lift a piece (selection feedback)
  liftPiece(
    piece: ChessPiece,
    height?: number,                   // Default: 0.4 units
    duration?: number,                 // Default: 200ms
  ): void;

  // Return a lifted piece to the board
  dropPiece(
    piece: ChessPiece,
    duration?: number,                 // Default: 200ms, ease-spring
  ): void;

  // Dissolve a captured piece
  dissolvePiece(
    piece: ChessPiece,
    duration?: number,                 // Default: 300ms
    // Animation: scale 1.0 → 0.0 + opacity 1.0 → 0.0 simultaneously
  ): void;

  // Materialize a piece (for setup, unlock, promotion)
  materializePiece(
    piece: ChessPiece,
    duration?: number,                 // Default: 400ms
    // Animation: scale 0.0 → 1.0 (ease-out-back) + opacity 0.0 → 1.0
  ): void;

  // Staggered entrance for all pieces (lesson/game start)
  enterAllPieces(
    pieces: ChessPiece[],
    options?: {
      staggerMs?: number,              // Default: 40ms per piece
      order?: 'rank' | 'file' | 'random',  // Default: 'rank' (back rank first)
      dropHeight?: number,             // Default: 2 units (pieces fall from above)
      easing?: EasingFunction,         // Default: ease-spring (bounce on land)
    }
  ): void;

  // Rearrange pieces for a new position (practice puzzle transition)
  rearrangePieces(
    currentFen: string,
    targetFen: string,
    duration?: number,                 // Default: 400ms
    // All pieces move simultaneously to new positions
    // Pieces leaving: dissolve
    // Pieces appearing: materialize
    // Pieces moving: glide
  ): void;
}
```

### Knight Arc Path

The knight is special — its move is an L-shape, but animating along an L looks robotic. Instead:
- Calculate a parabolic arc from start to end.
- Peak height: 1.5 units above the board (enough to visually "jump" over pieces).
- Duration: 700ms (slightly longer than linear moves).
- Easing: ease-out-expo for the horizontal, separate ease for vertical (rise fast, fall with gravity + micro-bounce).

---

## 5. CHESS ARROWS (Annotation Overlays)

Arrows are used in Learn mode to show suggested moves, and in Hint mode.

### Construction
- An arrow is a 3D object sitting 0.1 units above the board surface.
- Shaft: A rounded BoxGeometry or extruded shape, width 0.1 units.
- Head: A ConeGeometry or triangular prism, width 0.3 units.
- The arrow follows the straight line from center of `from` square to center of `to` square.
- For knight moves: The arrow bends at the L-junction (two segments joined).

### Material
```typescript
const MATERIAL_ARROW = {
  type: THREE.MeshStandardMaterial,
  color: 0xE63946,                     // --signal red
  roughness: 0.5,
  metalness: 0.3,
  transparent: true,
  opacity: 0.6,
};
```

### Animation
- Draw-on: Arrow scales along its length from 0 to full (from tail to head). Duration: 400ms, ease-out-expo.
- Idle: Subtle opacity breathing (0.5 ↔ 0.7, 3s cycle).
- Removal: Fade out (200ms).

---

## 6. CHESS PARTICLES (Unlock Effects)

A lightweight particle system for the Unlock screen's reveal moment.

### Setup
- 6–10 particles. Each is a tiny sphere (0.02 units) or a point sprite.
- Material: Additive blending, white, emissive.
- Particles spawn in a cylinder around the trophy object (radius: 1.5 units).
- Motion: Slow upward drift (0.2 units/second) with slight horizontal wobble (sine wave, 0.1 amplitude).
- Lifecycle: Each particle fades in at spawn height, reaches full opacity at mid-height, fades out at top. Total life: 4–6 seconds. Particles are recycled (loop).
- On first appearance: Stagger particle spawns over 2 seconds so they don't all appear at once.

### Performance Note
- Use THREE.Points (point cloud) if more than 10 particles. For 6–10, individual meshes are fine.
- Particles are ONLY active on the Unlock screen. Dispose when leaving.

---

## 7. UNLOCK TROPHY OBJECTS

Pre-built 3D objects for each achievement category. Same procedural approach as chess pieces.

| Category | Object | Construction |
|----------|--------|-------------|
| Tactics | Rearing Knight | Knight piece geometry but 2× scale, tilted 15° backward |
| Openings | Open Book | Two BoxGeometry "pages" angled at 30° from each other, spine is a CylinderGeometry |
| Endgame | Tall King | King piece geometry at 2× scale |
| Rating | Obelisk | Tapered BoxGeometry, narrow at top, with chamfered edges |
| Streak | Chain Links | Two TorusGeometry interlocked at 90° |
| Challenge | Crown | TorusGeometry base with 5 ConeGeometry points, no body below |
| Completion | Shield | ExtrudeGeometry from a shield SVG path, slight convex curve |

All trophies:
- Scale: ~2 units tall (fills the showcase camera frame nicely).
- Material: Same chrome as player pieces, but with slightly boosted envMapIntensity (1.2) for extra reflective drama.
- Idle animation: Slow Y-axis rotation, one full turn per 12 seconds.
- Tier variations: Adjust material color temperature per tier (see `05-screen-unlock.md` tier table).

---

## 8. INTERACTION LAYER

### Raycasting

- Use THREE.Raycaster for mouse/touch interaction.
- Cast against board squares and pieces separately (different interaction layers).
- Throttle raycasting to every 2 frames (30Hz) for performance.
- On mobile: No hover state. Tap = select, second tap = place. Long-press on a piece = drag.

### Drag & Drop

- When dragging a piece:
  - Piece lifts to 0.8 units and follows the cursor/touch position (projected onto the board plane).
  - Valid target squares glow (valid_move state).
  - Invalid drop: Piece springs back to origin (300ms, ease-spring).
  - Valid drop: Piece snaps to target square center (100ms, ease-out-expo).
- Drag uses a projected plane intersection (THREE.Plane at y=0) for smooth tracking.

### Touch Handling

- Distinguish tap from drag: If touch moves < 5px within 200ms, it's a tap.
- Prevent default touch behaviors (scroll, zoom) ONLY within the canvas.
- Support pinch-to-zoom on the board (camera dolly, clamped between FOV 25–50).

---

## 9. PERFORMANCE BUDGET

| Metric | Target | Ceiling |
|--------|--------|---------|
| Draw calls | < 80 | 120 |
| Triangles | < 50K | 80K |
| Textures | < 10MB VRAM | 20MB |
| Frame time | < 16ms (60fps) | 20ms (50fps) |
| JS heap | < 50MB | 80MB |
| First meaningful render | < 1.5s | 2.5s |

### Optimization Strategies
- **Instance meshes** for pawns (8 identical geometries, instanced rendering).
- **Shared materials** — all player pieces share ONE material instance. All opponent pieces share ONE. Clone only for temporary state overrides, then dispose.
- **Geometry caching** — build each piece type's geometry once, reuse via mesh cloning.
- **Conditional rendering** — disable post-processing bloom/grain on low-end devices (detect via renderer.capabilities or a user setting).
- **Dispose discipline** — every geometry, material, and texture created must have a corresponding dispose call when the component unmounts.

---

## 10. EXPORTS & API SURFACE

```typescript
// The public API that screen components consume

// React components (if using @react-three/fiber)
export { ChessScene } from './ChessScene';
export { ChessBoard } from './ChessBoard';
export { ChessPiece } from './ChessPiece';
export { ChessArrow } from './ChessArrow';
export { ChessParticles } from './ChessParticles';
export { TrophyObject } from './TrophyObject';

// Presets
export { CAMERA_PRESETS } from './presets/camera';
export { LIGHTING_STUDIO, LIGHTING_SPOTLIGHT, LIGHTING_DRAMATIC } from './presets/lighting';
export { MATERIALS } from './presets/materials';

// Animation utilities
export { PieceAnimator } from './animation/PieceAnimator';
export { CameraAnimator } from './animation/CameraAnimator';
export { LightingAnimator } from './animation/LightingAnimator';

// Types
export type { SquareState, CameraPreset, LightingRig, PieceType, PieceColor } from './types';

// Hooks (React)
export { useChessBoard } from './hooks/useChessBoard';       // Board state management
export { useChessCamera } from './hooks/useChessCamera';     // Camera preset transitions
export { useChessLighting } from './hooks/useChessLighting'; // Lighting mood transitions
```

---

## 11. BUILD & TEST CHECKLIST

Before integrating into any screen, verify the kit in isolation:

- [ ] Board renders with correct square colors and frame
- [ ] All 6 piece types render with correct proportions and materials
- [ ] Player vs opponent materials are visually distinct
- [ ] Key light creates visible shadows on the board surface
- [ ] Rim light creates edge highlights on chrome pieces
- [ ] Piece selection: lift animation works with spring easing
- [ ] Piece move: linear glide works for rook/bishop/queen paths
- [ ] Piece move: knight arc path clears neighboring pieces visually
- [ ] Piece capture: dissolve animation plays smoothly
- [ ] Piece entrance: staggered drop-in works with bounce
- [ ] Board flip: smooth 180° rotation
- [ ] Arrow: draws from source to target with correct head orientation
- [ ] Camera presets: transitions between all presets smoothly
- [ ] Lighting rigs: crossfade between STUDIO → DRAMATIC → SPOTLIGHT
- [ ] Bloom post-processing: visible on chrome highlights only, not blown out
- [ ] FPS stays above 50 on mid-range hardware
- [ ] All geometries and materials dispose correctly on unmount
- [ ] Touch interaction works on mobile (tap to select, tap to place)
- [ ] Keyboard: arrow keys navigate squares, Enter selects
- [ ] Trophy objects render with correct rotation idle
- [ ] Particles spawn, drift, and recycle without memory leaks

---

## 12. USAGE EXAMPLES PER SCREEN

Quick reference for how each screen consumes the kit:

| Screen | Camera Preset | Lighting Rig | Interaction | Special |
|--------|--------------|--------------|-------------|---------|
| **Learn** | `isometric` (dynamic, moves per step) | `STUDIO` | None (observation only) | Arrows, focus/dim materials, camera animation |
| **Practice** | `player_white` or `player_black` (static) | `STUDIO` | Full (select, move, drag) | Valid move highlights, wrong-move flash |
| **Challenge** | `player_white` or `player_black` (static) | `STUDIO` → `DRAMATIC` on checkmate | Full (select, move, drag, premove) | Check glow, captured piece tracking |
| **Report** | Not used (2D mini-boards via CSS) | N/A | None | — |
| **Unlock** | `showcase` | `SPOTLIGHT` | None (display only) | Trophy objects, particles, materialize animation |
