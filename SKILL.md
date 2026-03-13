---
name: remotion-ai-video
description: |
  Animation-first AI video production skill for Remotion.
  Motion graphics & kinetic typography by default.
  Image only when absolutely necessary (real person/place/product).
  Trigger: "video", "reel", "short", "animate", "motion graphic", "script to video"
---

# DIREKTOR v3 — Animation-First Video Skill

## CORE PHILOSOPHY

**DEFAULT = ANIMATION + MOTION GRAPHICS**
**IMAGE = শুধু real person / real place / real product এর জন্য**

যখন কেউ video বানাতে বলবে:
1. প্রতিটা scene এর জন্য প্রথমে ভাবো: "animation দিয়ে কি এটা বোঝানো যায়?"
2. যদি যায় → animation দাও, image দিও না
3. শুধু real visual must হলে image দাও

---

## PROJECT SETUP

```bash
npx create-remotion-app@latest my-video
cd my-video
npm install @remotion/player
```

---

## MAIN COMPOSITION

```tsx
// src/Root.tsx
import { Composition } from "remotion";
import { VideoComposition } from "./VideoComposition";
import scenes from "./scenes.json";

export const RemotionRoot = () => (
  <Composition
    id="MainVideo"
    component={VideoComposition}
    durationInFrames={scenes.reduce((a, s) => a + s.duration * 30, 0)}
    fps={30}
    width={1080}
    height={1920}
    defaultProps={{ scenes }}
  />
);
```

```tsx
// src/VideoComposition.tsx
import { AbsoluteFill, Series } from "remotion";
import { SceneRouter } from "./scenes/SceneRouter";

export const VideoComposition = ({ scenes }) => (
  <AbsoluteFill style={{ background: "#000" }}>
    <Series>
      {scenes.map((scene, i) => (
        <Series.Sequence key={i} durationInFrames={scene.duration * 30}>
          <SceneRouter scene={scene} />
        </Series.Sequence>
      ))}
    </Series>
  </AbsoluteFill>
);
```

```tsx
// src/scenes/SceneRouter.tsx
import { KineticTextScene }   from "./KineticTextScene";
import { WordExplosionScene } from "./WordExplosionScene";
import { ParticleFieldScene } from "./ParticleFieldScene";
import { CounterRevealScene } from "./CounterRevealScene";
import { GlitchBurstScene }   from "./GlitchBurstScene";
import { SplitScreenScene }   from "./SplitScreenScene";
import { NeonGlowScene }      from "./NeonGlowScene";
import { LineDrawScene }      from "./LineDrawScene";
import { ShapeMorphScene }    from "./ShapeMorphScene";
import { ImageBgScene }       from "./ImageBgScene";

export const SceneRouter = ({ scene }) => {
  const map = {
    KINETIC_TEXT:   <KineticTextScene   scene={scene} />,
    WORD_EXPLOSION: <WordExplosionScene scene={scene} />,
    PARTICLE_FIELD: <ParticleFieldScene scene={scene} />,
    COUNTER_REVEAL: <CounterRevealScene scene={scene} />,
    GLITCH_BURST:   <GlitchBurstScene   scene={scene} />,
    SPLIT_SCREEN:   <SplitScreenScene   scene={scene} />,
    NEON_GLOW:      <NeonGlowScene      scene={scene} />,
    LINE_DRAW:      <LineDrawScene      scene={scene} />,
    SHAPE_MORPH:    <ShapeMorphScene    scene={scene} />,
    IMAGE_BG:       <ImageBgScene       scene={scene} />,
  };
  return map[scene.sceneType] ?? <KineticTextScene scene={scene} />;
};
```

---

## SCENE TYPE GUIDE

```
KINETIC_TEXT    → animated text + geometric/gradient bg
                  কখন: motivational quote, bold statement, hook

WORD_EXPLOSION  → একটা word পুরো screen ভরে EXPLODE করে
                  কখন: শুরুতে hook, climax, CTA

PARTICLE_FIELD  → floating particles + text overlay
                  কখন: dreamy, emotional, inspiring scene

COUNTER_REVEAL  → animated number বড় হয়ে দেখায়
                  কখন: statistic, achievement, social proof

GLITCH_BURST    → RGB split glitch দিয়ে text burst
                  কখন: shock moment, dramatic reveal

SPLIT_SCREEN    → screen দুভাগ — contrast দেখাতে
                  কখন: before/after, problem/solution

NEON_GLOW       → dark bg + neon glowing text
                  কখন: tech, gaming, energy feel

LINE_DRAW       → lines draw হয়ে text তৈরি হয়
                  কখন: elegant reveal, premium feel

SHAPE_MORPH     → shapes morph করে message বোঝায়
                  কখন: transformation, change, growth

IMAGE_BG        → real image background
                  কখন: ONLY real person/place/product
```

---

## IMAGE দেবে না — বদলে এটা দাও

```
"success" বোঝাতে    → COUNTER_REVEAL বা WORD_EXPLOSION
"hope" বোঝাতে       → PARTICLE_FIELD
"change" বোঝাতে     → SHAPE_MORPH
"energy" বোঝাতে     → NEON_GLOW
"contrast" বোঝাতে   → SPLIT_SCREEN
"dramatic feel"      → KINETIC_TEXT + pure-black bg
"emotional moment"   → PARTICLE_FIELD + slow music
"achievement"        → COUNTER_REVEAL + WORD_EXPLOSION
```

---

## BACKGROUND OPTIONS

```
pure-black      → #000 — dramatic, cinematic
pure-white      → #fff — minimal, clean
dark-gradient   → গাঢ় থেকে গাঢ়, depth effect
color-gradient  → 2 bold color flow
geometric-grid  → subtle neon grid lines
noise-grain     → animated grain texture
radial-glow     → center থেকে glow ছড়ায়
```

---

## PATTERN INTERRUPT (প্রতি ২-৩ সেকেন্ডে একটা হবেই)

```
WORD_POP        → key word হঠাৎ 3x বড় হয়ে ছোট হয়
COLOR_FLASH     → 3 frame background color বদলায়
PARTICLE_BURST  → particles explode করে
GLITCH_HIT      → 6 frame RGB split
ZOOM_PUNCH      → 1.0 → 1.15 zoom, তারপর normal
LINE_SLASH      → diagonal line screen কাটে
SHAKE           → 3px screen shake, 4 frame
```

---

## OUTPUT JSON FORMAT

```json
[
  {
    "id": 1,
    "startTime": "00:00",
    "endTime": "00:03",
    "duration": 3,
    "voiceover": "exact spoken words",
    "sceneType": "WORD_EXPLOSION",
    "background": "pure-black",
    "primaryText": "STOP",
    "secondaryText": "তুমি কি সত্যিই চেষ্টা করেছিলে?",
    "accentColor": "#ff3c3c",
    "textAnimation": "scale-punch",
    "transition": "glitch",
    "patternInterrupts": ["COLOR_FLASH", "PARTICLE_BURST"],
    "motionGraphic": null,
    "musicMood": "dramatic",
    "imageQuery": null
  },
  {
    "id": 2,
    "startTime": "00:03",
    "endTime": "00:07",
    "duration": 4,
    "voiceover": "spoken words",
    "sceneType": "COUNTER_REVEAL",
    "background": "dark-gradient",
    "primaryText": "97%",
    "secondaryText": "মানুষ প্রথম চেষ্টায় হার মেনে নেয়",
    "accentColor": "#f5c518",
    "textAnimation": "count-up",
    "transition": "zoom",
    "patternInterrupts": ["ZOOM_PUNCH", "WORD_POP"],
    "motionGraphic": { "type": "counter", "from": 0, "to": 97, "suffix": "%" },
    "musicMood": "tense",
    "imageQuery": null
  },
  {
    "id": 3,
    "startTime": "00:07",
    "endTime": "00:12",
    "duration": 5,
    "voiceover": "spoken words",
    "sceneType": "IMAGE_BG",
    "background": "image",
    "imageQuery": "lone person mountain top sunrise dramatic cinematic 4k",
    "primaryText": "THEY KEPT GOING",
    "secondaryText": "বাকি ৩% ইতিহাস লিখেছে",
    "accentColor": "#ffffff",
    "textAnimation": "reveal",
    "transition": "fade",
    "patternInterrupts": ["ZOOM_PUNCH"],
    "motionGraphic": null,
    "musicMood": "epic",
    "note": "Image — real mountain/person visual দরকার"
  }
]
```

---

## DIRECTOR RULE

ভালো video এর ratio:
```
70%+ → animation scenes (KINETIC_TEXT, WORD_EXPLOSION, COUNTER_REVEAL...)
30%  → IMAGE_BG (শুধু যেখানে real visual must)
```

→ Full component code: [references/components.md](references/components.md)
→ Hook prompt generator: [HOOK_PROMPT.md](HOOK_PROMPT.md)
