# SOUND BRIEF 01S: LEARN SCREEN
## Audio Layer — The Lesson Theater

**Prerequisites:** Read `00C-sound-foundation.md` for engine setup, tokens, and sonic identity. This brief maps sounds to every interaction and moment already built in the Learn screen.

---

## NARRATION (ElevenLabs Voice)

The Learn screen is the ONLY screen with voice narration. This is its defining audio feature.

### When Narration Plays

| Moment | Behavior |
|--------|----------|
| Step loads (explanation text appears) | Narration begins after a 400ms delay (let the visual settle first). Fade-in: 200ms. |
| User clicks NEXT before narration finishes | Current narration fades out (300ms). Next step's narration begins with standard 400ms delay. Never cut abruptly — always fade. |
| User clicks PREV | Same fade behavior. Previous step's narration replays from the start. |
| Auto-play mode active | Narration plays fully, then a 1.5s pause, then auto-advance. The step does NOT advance until narration finishes + pause completes. The progress bar under the narration panel syncs to narration duration, not a fixed timer. |
| Lesson loads (first step) | Narration begins after the board entrance animation completes (~2.5s into the screen load). |
| Key insight callout (if present) | Narration reads the main explanation first, then a 600ms pause, then reads the insight with a slightly lower pitch shift (rate: 0.97). This creates a subtle "aside" tone — like the narrator leaning in. |

### Narration + Board Motion Sync

When a step involves a piece move:
1. Narration begins → reads the move notation ("Knight to f3...").
2. As the notation is spoken, the piece begins its move animation. Time the move to START when the notation STARTS in the audio. This requires knowing the narration timing — either manually align, or detect the move notation portion via timestamp markers in the pre-generated audio.
3. Narration continues with the explanation as/after the piece arrives.

If precise sync is too complex for v1: Start the piece move at the same moment narration starts. The overlap is close enough to feel intentional.

### Narration Controls

- A small speaker icon near the narration panel: toggles narration on/off for this session.
- When off: No audio plays, text is still fully readable. No visual change to the layout.

---

## SOUND EFFECTS MAP

### Board Entrance (First Load)

| Event | Sound | Timing | Token |
|-------|-------|--------|-------|
| Light fades up on board | `ambient-wash` — a very low, filtered noise swell | Starts at 0ms, fades in over 600ms | Bus: ambient, VOLUME_SUBLIMINAL |
| Each piece drops into place | `piece-place` with pitch variation | On each piece's landing frame (staggered with the 40ms visual stagger) | Bus: sfx, VOLUME_SUBTLE |
| All pieces settled | Brief silence (200ms), then a soft `game-start` swell | After last piece lands | Bus: sfx, VOLUME_DEFAULT |

**Important:** The staggered piece-place sounds should create a rhythmic "rain of pieces" effect. With 40ms stagger × 16 pieces, the total is ~640ms of rapid tapping. Each tap has ±4% pitch variation so it doesn't sound mechanical.

### Step Navigation

| Event | Sound | Timing | Token |
|-------|-------|--------|-------|
| Click NEXT / press Right Arrow | `step-advance` — a soft, clean transition sound | On click, immediately | Bus: sfx, VOLUME_SUBTLE |
| Click PREV / press Left Arrow | `step-advance` pitched down 5% (rate: 0.95) — same sound, but the lower pitch signals "going back" | On click, immediately | Bus: sfx, VOLUME_SUBTLE |
| Step dot transitions | No sound — the visual dot slide is enough | — | — |

### Piece Movement (Within a Step)

| Event | Sound | Timing | Token |
|-------|-------|--------|-------|
| Piece begins gliding | `piece-slide` — faint chrome-on-glass glide | At move animation start | Bus: sfx, VOLUME_WHISPER |
| Piece lands at destination | `piece-place` | At move animation end (600ms) | Bus: sfx, VOLUME_DEFAULT |
| Piece captures another | `piece-capture` (impact + low resonance) replacing `piece-place` | At move animation end | Bus: sfx, VOLUME_PROMINENT |
| Check occurs | `check` chime, 200ms after `piece-place` | 200ms after landing | Bus: sfx, VOLUME_PROMINENT |
| Arrow draws on board | `hint-reveal` — soft, subtle | On arrow draw start | Bus: sfx, VOLUME_WHISPER |

### Camera Movement

| Event | Sound | Timing | Token |
|-------|-------|--------|-------|
| Camera orbits/dollies to new position | `transition-whoosh` — very subtle, barely perceptible directional air movement | At camera animation start | Bus: sfx, VOLUME_SUBLIMINAL |
| Camera settles at new position | No sound — the narration or next piece move handles the "arrival" | — | — |

### Controls

| Event | Sound | Timing | Token |
|-------|-------|--------|-------|
| Hover over PREV/NEXT buttons | `ui-hover` | On mouseenter | Bus: sfx, VOLUME_WHISPER |
| Click any button | `ui-click` | On click | Bus: sfx, VOLUME_SUBTLE |
| Toggle auto-play ON | `ui-click` + brief ascending tone (rate: 1.05) | On click | Bus: sfx, VOLUME_SUBTLE |
| Toggle auto-play OFF | `ui-click` + brief descending tone (rate: 0.95) | On click | Bus: sfx, VOLUME_SUBTLE |
| Flip board | `transition-whoosh` at VOLUME_DEFAULT (more pronounced than camera move) | On flip animation start | Bus: sfx, VOLUME_DEFAULT |

### Lesson Complete State

| Event | Sound | Timing | Token |
|-------|-------|--------|-------|
| "Lesson Complete" card appears | `streak-milestone` — two ascending chimes + sustain | On card fade-in start | Bus: sfx, VOLUME_PROMINENT |
| Camera pulls back to wide shot | `transition-whoosh` slow and quiet | On camera move start | Bus: sfx, VOLUME_SUBLIMINAL |
| Narration (if enabled) | A final line: "Well played. Ready to put this into practice?" — pre-generated, warm tone | 800ms after card appears | Bus: narration, VOLUME_FULL |

---

## AMBIENT (Optional, Off by Default)

If the user enables ambient audio:
- A continuous dark ambient pad — low-frequency drone, almost subliminal.
- Fade in over 2 seconds when entering the Learn screen.
- Fade out over 1.5 seconds when leaving.
- Ducks (volume -50%) when narration is playing.
- Suggested generation prompt for ElevenLabs Sound Effects: "Dark minimal ambient drone, very low frequency, subtle tonal movement, no melody, no rhythm, continuous, like a quiet room in a museum at night, barely audible"
