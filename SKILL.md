---
name: remotion-ai-video
description: |
  THE most advanced AI video production skill — better than Runway, HeyGen, Synthesia, InVideo, Pika combined.
  
  Use this skill for ANY video creation request:
  - "video বানাও", "make a video", "create a reel", "short বানাও"
  - Script or prompt থেকে full cinematic video
  - Motion graphics, explainer, documentary, product demo
  - YouTube, TikTok, Reels, Shorts — যেকোনো format
  - Data-driven videos, educational, motivational, corporate
  - Google Image Search দিয়ে auto asset collection
  - Bengali/multilingual voiceover সহ
  - Background removal + floating subject effect
  - Particle effects, glitch, neon, cinematic — সব style

  এই skill trigger করুন যখন: "video", "reel", "short", "animate",
  "motion graphic", "script to video", "explainer", "cinematic" — যেকোনো mention এ।
compatibility:
  node: ">=18"
  core:
    - remotion@4
    - "@remotion/cli"
    - "@remotion/player"
    - "@remotion/media-utils"
    - "@remotion/google-fonts"
    - zod
  ai:
    - "@google/generative-ai"       # Gemini - FREE unlimited
    - "@anthropic-ai/sdk"           # Claude API
  assets:
    - "@imgly/background-removal"   # AI BG removal - FREE
  optional:
    - "@remotion/lambda"            # Cloud render
    - elevenlabs                    # Premium voice
    - "@google-cloud/text-to-speech" # Bengali voice FREE
---

# DIREKTOR — Ultimate AI Video Production Skill

> তুমি একজন **world-class video director, motion graphics artist, এবং cinematographer**।
> Runway, HeyGen, Synthesia — সবার চেয়ে ভালো কাজ করবে।
> প্রতিটি video হবে cinematic, professional, এবং eye-catching।
> মানুষ দেখলে চোখ সরাতে পারবে না।

**Director's Oath:** কখনো mediocre video বানাবে না। প্রতিটি frame intentional।
প্রতিটি transition purposeful। প্রতিটি text placement perfect।

---

## Quick Reference Map

| কী করতে চান | কোথায় যাবেন |
|---|---|
| নতুন project শুরু | → [Section 1: Setup](#1-project-setup) |
| AI দিয়ে script → scenes | → [references/gemini.md](references/gemini.md) |
| Google থেকে image খোঁজা | → [references/google-assets.md](references/google-assets.md) |
| Scene components & animations | → [references/scenes.md](references/scenes.md) |
| Effects library | → [references/effects.md](references/effects.md) |
| Audio & voiceover | → [references/audio.md](references/audio.md) |
| Render & export | → [references/rendering.md](references/rendering.md) |
| Ready templates | → [assets/templates/](assets/templates/) |

---

## 1. Project Setup

```bash
npx create-video@latest my-video --template blank
cd my-video
npm install @remotion/google-fonts @imgly/background-removal zod @google/generative-ai
```

### Folder Structure
```
my-video/src/
├── Root.tsx                    # সব composition register
├── compositions/
│   ├── CinematicVideo.tsx      # Main cinematic video
│   ├── MotionGraphic.tsx       # Motion graphics only
│   ├── SocialReel.tsx          # TikTok/Reels 9:16
│   └── DataVideo.tsx           # Data-driven stats video
├── components/
│   ├── BaseScene.tsx           # Base scene (BG + overlay + grain)
│   ├── AnimatedText.tsx        # All text animations
│   ├── Transitions.tsx         # Scene transitions
│   ├── Effects.tsx             # Neon, glitch, particles
│   ├── BGRemoved.tsx           # Floating subject component
│   ├── MotionGraphics.tsx      # Counters, bars, shapes
│   └── Captions.tsx            # Auto subtitles
├── pipeline/
│   ├── gemini.ts               # Gemini AI brain
│   ├── google-search.ts        # Google image search (Serper)
│   ├── bgremove.ts             # Background removal
│   └── voiceover.ts            # TTS pipeline
└── types.ts
```

---

## 2. THE AI PIPELINE — Prompt → Professional Video

এটাই আমাদের secret weapon। অন্য কোনো tool এটা এভাবে করে না।

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 1 ▸ USER PROMPT
         "60s motivational video about success"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         ↓
STEP 2 ▸ GEMINI AI BRAIN
         • Script লেখে (hook + body + CTA)
         • ৫-৮টা scene এ ভাগ করে
         • প্রতি scene এর জন্য:
           - Headline, subtext, voiceover
           - Google search query (specific!)
           - Color grade, effect, transition
           - BG remove দরকার কিনা
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         ↓
STEP 3 ▸ GOOGLE IMAGE SEARCH (Serper API)
         • প্রতি scene এর জন্য Google এ search
         • Best HD image URL নিয়ে আসে
         • Fallback: Unsplash → Pexels
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         ↓
STEP 4 ▸ BACKGROUND REMOVAL (AI)
         • Subject আলাদা করে
         • Transparent PNG বানায়
         • Floating effect এর জন্য ready
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         ↓
STEP 5 ▸ REMOTION COMPOSITION
         • Ken Burns effect (image zoom/pan)
         • Cinematic color grading
         • Animated text (fadeUp/reveal/typewriter)
         • Smooth transitions
         • Motion graphics overlay
         • Particle effects
         • Cinematic letterbox bars
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         ↓
STEP 6 ▸ VOICEOVER + AUDIO
         • Google TTS (Bengali FREE)
         • ElevenLabs (premium)
         • Background music
         • Audio ducking during voice
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         ↓
STEP 7 ▸ RENDER → MP4
         • Platform-specific format
         • YouTube / TikTok / Reels ready
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. কেন এটা সবার চেয়ে Better?

| Feature | Runway | HeyGen | Synthesia | InVideo | **এই Skill** |
|---|---|---|---|---|---|
| Script → Video | ✅ | ✅ | ✅ | ✅ | ✅ |
| Google Image Search | ❌ | ❌ | ❌ | ❌ | ✅ |
| BG Remove + Float | ❌ | ✅ | ❌ | ❌ | ✅ |
| Bengali Voice | ❌ | limited | ❌ | ❌ | ✅ FREE |
| Motion Graphics Code | ❌ | ❌ | ❌ | ❌ | ✅ full |
| Full Code Control | ❌ | ❌ | ❌ | ❌ | ✅ |
| Offline Render | ❌ | ❌ | ❌ | ❌ | ✅ |
| 100% Free Pipeline | ❌ | ❌ | ❌ | ❌ | ✅ |
| Custom Effects | limited | ❌ | ❌ | limited | ✅ unlimited |
| Particle Systems | ❌ | ❌ | ❌ | ❌ | ✅ |
| Glitch / Neon / Retro | limited | ❌ | ❌ | ❌ | ✅ |
| Data-Driven Video | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## 4. Video Style Catalog

প্রতিটি style এর জন্য specific implementation আছে [references/effects.md](references/effects.md) এ।

```
🎬 CINEMATIC      Film-grade · letterbox · color LUT · Ken Burns
⚡ NEON/CYBER     Grid overlay · glow text · dark bg · RGB accent
◻  MINIMAL        White space · elegant type · subtle motion
📡 GLITCH         RGB split · scan lines · digital artifacts
🌿 NATURE         Breathing scale · organic colors · soft light
🏢 CORPORATE      Clean data · animated stats · professional type
📼 RETRO/VHS      Grain · color bleed · vintage palette · noise
🌑 DARK/MOODY     Deep blacks · dramatic shadows · slow reveal
💫 PARTICLES      Floating particles · burst effects · trails
🎭 DOCUMENTARY    Lower thirds · interview style · b-roll cuts
```

---

## 5. Director's Quality Checklist

প্রতিটি video বানানোর পর এটা check করো:

**Visuals**
- [ ] প্রতিটি scene এ color grade আছে — raw footage acceptable নয়
- [ ] Ken Burns effect আছে (static image = boring)
- [ ] Vignette + grain আছে (cinematic feel)
- [ ] Typography Google Fonts থেকে — system font নয়
- [ ] Text safe zone মানা হয়েছে (10% padding)

**Pacing**
- [ ] Hook: প্রথম ৩ সেকেন্ড attention grab করে
- [ ] Fast content: ৩-৫s per scene
- [ ] Documentary: ৬-১০s per scene
- [ ] শেষ scene এ strong CTA আছে

**Audio**
- [ ] Background music আছে
- [ ] Voiceover এর সময় music ducked (30% volume)
- [ ] Hard audio cut নেই — fade in/out আছে
- [ ] Captions/subtitles generate হয়েছে

**Technical**
- [ ] Target platform format সঠিক
- [ ] Render settings optimize হয়েছে
- [ ] Props schema Zod দিয়ে validate হয়েছে

---

## 6. Environment Setup

```env
# AI (Gemini FREE — unlimited flash model)
GEMINI_API_KEY=get_free_at_aistudio.google.com

# Google Image Search (2500 FREE queries/month)
SERPER_API_KEY=get_free_at_serper.dev

# Voiceover (Bengali FREE)
# Google Cloud TTS — bn-BD-Standard-A

# Optional Premium
ELEVENLABS_API_KEY=optional_premium_voice
PEXELS_API_KEY=optional_fallback_images
UNSPLASH_ACCESS_KEY=optional_fallback_images
```

---

## 7. One-Line Video Generation

```bash
# Install dependencies
npm install

# Generate from prompt — এটুকুই দরকার!
npx ts-node scripts/generate.ts "your video prompt here"

# With options
npx ts-node scripts/generate.ts "motivational video" --style=cinematic --duration=60 --lang=bn --format=youtube
```

সব detail এর জন্য reference files পড়ো।
