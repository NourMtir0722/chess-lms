# BRIEF 00C: SOUND FOUNDATION
## Audio Identity & Engine — Source of Truth

**Prerequisites:** Read `00-design-foundation.md` for visual context. This brief defines the sonic equivalent — the audio DNA that every screen-level sound brief inherits.

---

## 1. SONIC IDENTITY

### What This Sounds Like

The audio mirrors the visual language: **dark, precise, metallic, cinematic.** Every sound feels like it was recorded inside a black marble room with a single microphone. Close. Dry. Controlled.

**Reference touchstones:**
- The UI sounds of a Porsche Taycan's dashboard
- The mechanical clicks in a Leica camera shutter
- The ambient tension of a Fincher film soundtrack (Trent Reznor / Atticus Ross)
- The restrained feedback sounds of a high-end watch winder

**This is NOT:**
- ❌ Wooden chess piece sounds (chess.com / lichess)
- ❌ Orchestral / classical music stings
- ❌ 8-bit or retro game sounds
- ❌ Bright, bubbly notification pings (Duolingo, Slack)
- ❌ Overly reverberant / cathedral ambience
- ❌ Sound effects that call attention to themselves — audio serves the experience, never dominates it

### Sonic Material Palette

Just as the visual design uses chrome, obsidian, and void — the sound design has its own materials:

| Sonic Material | Character | Used For |
|---------------|-----------|----------|
| **Ceramic-on-stone** | Short, dry, crisp impact. Like a polished marble ball landing on slate. | Piece placement, button clicks |
| **Metallic resonance** | A clean, sustained tone with slight harmonic overtone. Like striking a tuning fork. | Check, notifications, alerts |
| **Chrome slide** | Smooth, frictionless glide with a faint high-frequency shimmer. | Piece movement, transitions, sliders |
| **Subsonic pulse** | A felt-more-than-heard low-frequency throb. 40–60Hz. | Tension, captures, errors, checkmate |
| **Crystalline chime** | A single high-pitched, clear tone. Glass being tapped with a silver pin. | Achievements, unlocks, correct answers |
| **Dark ambient wash** | Nearly imperceptible, tonal bed. Filtered noise + distant sine waves. | Background atmosphere (optional) |

---

## 2. ELEVENLABS INTEGRATION

### Purpose

ElevenLabs provides the **lesson narrator voice** for the Learn screen. The narrator reads step explanations, key insights, and move annotations aloud — turning passive reading into a guided, cinematic experience.

### Voice Configuration

```javascript
// ElevenLabs API configuration
{
  voice_id: "<selected_voice_id>",     // See voice selection below
  model_id: "eleven_turbo_v2_5",       // Low latency for step-by-step playback
  
  voice_settings: {
    stability: 0.75,                   // High stability — authoritative, consistent tone
    similarity_boost: 0.80,            // Stay close to the selected voice character
    style: 0.15,                       // Low style exaggeration — understated delivery
    use_speaker_boost: true,
  },

  output_format: "mp3_44100_128",      // High quality, reasonable file size
}
```

### Voice Selection Criteria

Choose or clone a voice with these traits:
- **Gender:** Neutral preference — select what fits the brand. A deep, calm male voice or a clear, measured female voice both work. Avoid extremes.
- **Tone:** Authoritative but warm. Think: a museum audio guide narrator, not a sports commentator.
- **Pace:** Naturally slow-to-medium. No rushing. Pauses between sentences feel intentional.
- **Accent:** Neutral/international English. No strong regional accent. Clean diction.
- **Energy:** Low-to-medium. Never excited. Never bored. Steady, like a grandmaster explaining a position to a student they respect.

**Recommended ElevenLabs preset voices to audition:**
- "Daniel" — deep, calm, measured
- "Antoni" — warm, clear, slightly authoritative
- Clone a custom voice if the brand demands a unique identity

### Pre-generation vs Real-time

**Pre-generate lesson narration.** Do NOT stream TTS in real-time during lessons.

Reasons:
- Latency: Even Turbo v2.5 has 300–500ms latency. In a step-by-step lesson, that delay between clicking "next" and hearing the voice breaks the cinematic feel.
- Cost: Pre-generating once per lesson is far cheaper than per-user streaming.
- Quality: Pre-generated audio can be reviewed, trimmed, and normalized.

**Workflow:**
1. Write lesson step text.
2. Batch-generate audio via ElevenLabs API for all steps in a lesson.
3. Store audio files as MP3s alongside lesson data (e.g., `lessons/{id}/audio/step-01.mp3`).
4. The Learn screen loads and plays these files per step.

```typescript
// Pre-generation script (run offline, not in the client)
async function generateLessonAudio(lesson: Lesson) {
  for (const step of lesson.steps) {
    if (!step.explanation) continue;
    
    const response = await fetch(
      `https://api.elevenlabs.io/v1/text-to-speech/${VOICE_ID}`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'xi-api-key': process.env.ELEVENLABS_API_KEY,
        },
        body: JSON.stringify({
          text: step.explanation,
          model_id: 'eleven_turbo_v2_5',
          voice_settings: {
            stability: 0.75,
            similarity_boost: 0.80,
            style: 0.15,
            use_speaker_boost: true,
          },
        }),
      }
    );
    
    const audioBuffer = await response.arrayBuffer();
    await saveFile(`lessons/${lesson.id}/audio/step-${step.index}.mp3`, audioBuffer);
  }
}
```

### Narration Text Guidelines

When writing lesson text that will be narrated:
- Keep sentences short. 8–15 words per sentence.
- Use chess notation naturally: "Knight to f7, check" not "Nf7+". The voice should speak notation as words.
- Avoid parenthetical asides. They sound awkward spoken aloud.
- End steps with a slight hook: "And this is where it gets interesting." — to pull the user to the next step.
- Include natural pauses via punctuation. An em-dash or ellipsis creates a breath in the TTS output.

---

## 3. SOUND ENGINE ARCHITECTURE

### Web Audio API Setup

Use the Web Audio API directly (not a library like Howler.js) for maximum control over spatial audio, effects chains, and precise timing.

```typescript
class SoundEngine {
  private context: AudioContext;
  private masterGain: GainNode;
  private sfxBus: GainNode;
  private narrationBus: GainNode;
  private ambientBus: GainNode;
  
  // Separate volume channels
  private volumes = {
    master: 0.8,
    sfx: 0.7,
    narration: 1.0,
    ambient: 0.3,
  };

  constructor() {
    this.context = new AudioContext();
    
    // Master → output
    this.masterGain = this.context.createGain();
    this.masterGain.gain.value = this.volumes.master;
    this.masterGain.connect(this.context.destination);
    
    // Buses → master
    this.sfxBus = this.createBus(this.volumes.sfx);
    this.narrationBus = this.createBus(this.volumes.narration);
    this.ambientBus = this.createBus(this.volumes.ambient);
  }

  private createBus(volume: number): GainNode {
    const gain = this.context.createGain();
    gain.gain.value = volume;
    gain.connect(this.masterGain);
    return gain;
  }
}
```

### Audio Buses

Three separate buses with independent volume control:

| Bus | Purpose | Default Volume | User Adjustable |
|-----|---------|---------------|-----------------|
| **SFX** | Piece sounds, UI feedback, game events | 0.7 | Yes |
| **Narration** | ElevenLabs voice in Learn mode | 1.0 | Yes |
| **Ambient** | Background atmosphere (optional) | 0.3 | Yes (including off) |

### Sound Playback Utility

```typescript
interface PlayOptions {
  bus: 'sfx' | 'narration' | 'ambient';
  volume?: number;            // 0–1, relative to bus volume
  rate?: number;              // Playback speed. 1.0 = normal. Used for pitch variation.
  fadeIn?: number;            // Fade-in duration in ms
  fadeOut?: number;           // Fade-out duration in ms
  loop?: boolean;
  delay?: number;             // Delay before playback starts, in ms
  onEnd?: () => void;
}

// Usage
soundEngine.play('piece-place', { bus: 'sfx', volume: 0.8 });
soundEngine.play('step-03-narration', { bus: 'narration', fadeIn: 200 });
```

### Sound Sprite Strategy

For SFX, use a **sound sprite** approach — one audio file containing all short SFX, with start/end timestamps for each sound. This eliminates per-file HTTP requests and reduces latency.

```typescript
// Sound sprite definition
const SFX_SPRITE = {
  src: '/audio/sfx-sprite.mp3',
  sprites: {
    'piece-place':       { start: 0,     end: 0.4 },
    'piece-capture':     { start: 0.5,   end: 1.1 },
    'piece-slide':       { start: 1.2,   end: 1.8 },
    'check':             { start: 2.0,   end: 2.8 },
    'checkmate-impact':  { start: 3.0,   end: 4.5 },
    'wrong-move':        { start: 4.6,   end: 5.0 },
    'correct-move':      { start: 5.1,   end: 5.7 },
    'ui-click':          { start: 5.8,   end: 6.0 },
    'ui-hover':          { start: 6.1,   end: 6.25 },
    'timer-tick':        { start: 6.3,   end: 6.45 },
    'timer-warning':     { start: 6.5,   end: 7.0 },
    'unlock-chime':      { start: 7.1,   end: 9.0 },
    'unlock-reveal':     { start: 9.1,   end: 11.5 },
    'streak-milestone':  { start: 11.6,  end: 12.4 },
    'transition-whoosh': { start: 12.5,  end: 13.2 },
    'draw-offer':        { start: 13.3,  end: 13.8 },
    'game-start':        { start: 14.0,  end: 15.0 },
    'step-advance':      { start: 15.1,  end: 15.5 },
    'hint-reveal':       { start: 15.6,  end: 16.2 },
    'rating-tick':       { start: 16.3,  end: 16.5 },
  },
};
```

### Audio Generation Notes

Sound effects can be created via:
1. **ElevenLabs Sound Effects API** — Generate from text descriptions. Ideal for unique, high-quality sounds.
2. **Synthesis** — Generate procedurally in the Web Audio API (oscillators, noise, filters) for simple UI sounds.
3. **Recorded / sourced** — From royalty-free libraries, then processed to match the sonic identity.

**ElevenLabs Sound Effects prompts** (for generating the SFX via their API):

```
piece-place:       "A single short ceramic impact on polished stone surface, close microphone, no reverb, clean and dry"
piece-capture:     "Ceramic impact on stone followed by a subtle low resonant hum, close mic, dry"
piece-slide:       "Smooth glass sliding on polished metal surface, short, frictionless, subtle high frequency shimmer"
check:             "A single clean metallic chime, like striking a small tuning fork, clear fundamental tone, minimal overtones, dry"
checkmate-impact:  "Deep subsonic impact followed by a resonant metallic ring, cinematic, no reverb, felt in the chest"
wrong-move:        "A short muted thud, like a dampened ceramic drop, slightly lower pitch, unsatisfying, very short"
correct-move:      "A bright crystalline ting, single note, like tapping thin glass with a silver pin, clean and satisfying"
ui-click:          "Micro mechanical click, like a precision switch engaging, extremely short, dry"
ui-hover:          "Very subtle soft breath of air, barely audible high-frequency whisper, under 150ms"
unlock-chime:      "Rising three-note crystalline chime sequence, each note higher, glass bells, clean and celebratory but restrained"
unlock-reveal:     "Slow cinematic low-frequency swell building to a bright metallic shimmer, 2 seconds, dramatic reveal"
timer-tick:        "Precise mechanical clock tick, single, dry, no resonance, like a watch escapement"
timer-warning:     "Same clock tick but with added subtle metallic tension ring, slightly urgent"
streak-milestone:  "Two quick ascending metallic chimes followed by a soft harmonic sustain, achievement feel"
game-start:        "Low ambient swell rising to a clean single chime, like a match bell, signaling the beginning"
```

---

## 4. SOUND TOKENS

Like CSS variables for visuals, define audio constants:

```typescript
const SOUND_TOKENS = {

  // Volume levels (relative to bus)
  VOLUME_FULL:        1.0,
  VOLUME_PROMINENT:   0.8,
  VOLUME_DEFAULT:     0.6,
  VOLUME_SUBTLE:      0.4,
  VOLUME_WHISPER:     0.2,
  VOLUME_SUBLIMINAL:  0.08,

  // Fade durations
  FADE_INSTANT:       0,        // ms
  FADE_QUICK:         100,
  FADE_NORMAL:        300,
  FADE_SLOW:          800,
  FADE_CINEMATIC:     1500,

  // Pitch variation (for natural feel — same sound, slight random pitch shift)
  PITCH_VARIATION:    0.04,     // ±4% random pitch shift on each play
  PITCH_NONE:         0,

  // Timing sync with motion (from design foundation)
  SYNC_PIECE_LAND:    600,      // Sound triggers at piece animation end (600ms)
  SYNC_STEP_CHANGE:   150,      // Sound triggers as old content fades out
  SYNC_CAMERA_START:  0,        // Sound triggers at camera transition start
  SYNC_REVEAL:        500,      // Sound triggers at the "beat" pause in unlock sequence
};
```

---

## 5. AUDIO CONTEXT MANAGEMENT

### Autoplay Policy

Browsers block AudioContext until user interaction. Handle this:

```typescript
// Resume audio context on first user interaction
async function initAudio() {
  if (soundEngine.context.state === 'suspended') {
    await soundEngine.context.resume();
  }
}

// Attach to the first meaningful interaction
document.addEventListener('click', initAudio, { once: true });
document.addEventListener('keydown', initAudio, { once: true });
document.addEventListener('touchstart', initAudio, { once: true });
```

### Preloading Strategy

- **Critical SFX sprite:** Preload on app init. This is one file (~500KB) containing all short sounds.
- **Narration audio:** Preload the CURRENT step + NEXT step. Lazy-load the rest as the user progresses. Discard played steps from memory.
- **Ambient loops:** Lazy-load. Start loading when the user enters a screen that uses ambient sound. Fade in when ready.
- **Unlock sounds:** Preload when achievement progress > 80% (anticipate the trigger).

### Mute & Preferences

```typescript
interface AudioPreferences {
  masterMute: boolean;
  sfxVolume: number;          // 0–1
  narrationVolume: number;    // 0–1
  ambientVolume: number;      // 0–1
  narrationEnabled: boolean;  // Some users prefer reading
  ambientEnabled: boolean;    // Off by default — opt-in
}
```

- Persist preferences in localStorage.
- Mute icon in every screen's settings/controls area.
- When narration is disabled, the Learn screen still functions — the text is always visible. Audio is an enhancement, never a requirement.

---

## 6. SYNC WITH MOTION

Sound must be tightly coupled with the visual motion system. Misaligned audio/visual is worse than no audio.

### Sync Rules

**RULE 1 — SOUND ON IMPACT, NOT ON INITIATION**
When a piece moves, the placement sound plays when the piece LANDS (at animation end), not when the move starts. The slide sound plays during the glide.

**RULE 2 — AUDIO LEADS VISUAL FOR REVEALS**
For unlock sequences and dramatic moments, a subtle audio swell begins 200–300ms BEFORE the visual appears. The ear primes the brain for what the eye is about to see.

**RULE 3 — ONE SOUND AT A TIME IN THE SFX BUS**
Never layer more than one SFX simultaneously (excluding ambient). If a new SFX triggers while one is playing, the previous one is cut (fast fade, 50ms). This keeps the mix clean.

**RULE 4 — NARRATION DUCKS SFX**
When narration is playing, SFX volume automatically reduces by 30%. When narration pauses or ends, SFX volume returns over 200ms. This prevents piece sounds from masking the voice.

**RULE 5 — PITCH MICRO-VARIATION**
For repeated sounds (piece placement, timer ticks), apply a random ±4% pitch shift each time. This prevents the "machine gun effect" where identical repetitions sound artificial.

**RULE 6 — SILENCE IS A SOUND**
Deliberate silence before a big moment (the 500ms void before the unlock reveal, the beat after checkmate) is as designed as any sound. Never fill silence with ambient just because it feels empty.

---

## 7. EXPORTS

```typescript
export { SoundEngine } from './SoundEngine';
export { SOUND_TOKENS } from './tokens';
export { SFX_SPRITE } from './sprites';

// Per-screen sound controllers
export { LearnSoundController } from './controllers/LearnSoundController';
export { PracticeSoundController } from './controllers/PracticeSoundController';
export { ChallengeSoundController } from './controllers/ChallengeSoundController';
export { ReportSoundController } from './controllers/ReportSoundController';
export { UnlockSoundController } from './controllers/UnlockSoundController';

// Hooks
export { useSoundEngine } from './hooks/useSoundEngine';
export { useNarration } from './hooks/useNarration';

// Types
export type { AudioPreferences, PlayOptions } from './types';
```
