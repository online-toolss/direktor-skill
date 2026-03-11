# Gemini AI Brain — Complete Pipeline

FREE unlimited flash model দিয়ে full script → scenes pipeline।

---

## Setup

```bash
# Free API key: https://aistudio.google.com
# gemini-1.5-flash = effectively unlimited free
npm install @google/generative-ai
```

---

## Full Pipeline

```typescript
// pipeline/gemini.ts
import { GoogleGenerativeAI } from '@google/generative-ai';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);
const flash = genAI.getGenerativeModel({ model: 'gemini-1.5-flash' });
const pro   = genAI.getGenerativeModel({ model: 'gemini-1.5-pro' });

// ── STEP 1: Write Script ──────────────────────────────────
export async function writeScript(topic: string, duration: number, style: string, lang = 'en'): Promise<string> {
  const result = await pro.generateContent(`
You are a world-class video scriptwriter. Write a ${duration}-second ${style} video script.
Language: ${lang === 'bn' ? 'Bengali (Bangla)' : 'English'}
Topic: ${topic}

Structure:
- Hook (0-3s): Grab attention immediately. Ask a question or make a bold statement.
- Body: Tell the story or explain the topic with vivid language.
- CTA (last 5s): Clear action for the viewer.

Write in natural spoken language only. No timestamps. No scene labels. Just the words.
  `);
  return result.response.text();
}

// ── STEP 2: Break into Scenes ─────────────────────────────
export async function scriptToScenes(script: string, style: string, lang = 'en'): Promise<Scene[]> {
  const result = await flash.generateContent(`
You are a professional video director breaking a script into scenes.
Visual style: ${style}
Language: ${lang}

Script:
${script}

Return ONLY a JSON array. No markdown, no explanation, no backticks.

Each scene:
{
  "id": number,
  "duration": 4-8 (seconds),
  "headline": "2-4 WORD CAPS HEADLINE",
  "subtext": "One supporting sentence (max 12 words)",
  "voiceover": "Exact words narrator says during this scene",
  "imageQuery": "very specific Google image search query — include style keywords like 'cinematic bokeh dramatic lighting'",
  "bgRemove": false,
  "effectStyle": "cinematic|neon|minimal|glitch|nature|corporate|retro|dark",
  "textAnimation": "reveal|fadeUp|slideIn|typewriter|scale|fade",
  "transition": "fade|zoom|wipe|glitch",
  "colorGrade": "warm|cool|teal-orange|noir|golden|arctic|vintage|vivid",
  "musicMood": "epic|calm|upbeat|emotional|dark|inspiring|tense|peaceful",
  "motionGraphic": null | "counter|progress|lowerThird|particles"
}

Rules:
- 4-7 scenes total
- imageQuery must be SPECIFIC (bad: "city", good: "cinematic city skyline night rain bokeh 4k")
- Set bgRemove:true only for scenes needing floating subject effect
- Match colorGrade to mood and style
  `);

  const text = result.response.text().replace(/```json|```/g, '').trim();
  return JSON.parse(text);
}

// ── STEP 3: Generate Captions ─────────────────────────────
export function generateCaptions(scenes: Scene[], fps: number): Caption[] {
  const captions: Caption[] = [];
  let cursor = 0;

  for (const scene of scenes) {
    const words = scene.voiceover.split(' ');
    const durationInFrames = scene.duration * fps;
    const framesPerWord = durationInFrames / Math.max(words.length, 1);

    // Group 4 words per caption
    for (let i = 0; i < words.length; i += 4) {
      const chunk = words.slice(i, i + 4).join(' ');
      const startFrame = Math.round(cursor + i * framesPerWord);
      const endFrame = Math.round(startFrame + 4 * framesPerWord);
      captions.push({ text: chunk, startFrame, endFrame });
    }

    cursor += durationInFrames;
  }

  return captions;
}

// ── STEP 4: Enhance Image Queries ─────────────────────────
export async function enhanceImageQueries(scenes: Scene[]): Promise<Scene[]> {
  const result = await flash.generateContent(`
Improve these image search queries to get the best Google Images results.
Make each query very specific with visual style keywords.

Current queries:
${scenes.map((s, i) => `${i + 1}. "${s.imageQuery}"`).join('\n')}

Return ONLY a JSON array of improved queries (same order):
["improved query 1", "improved query 2", ...]
  `);

  const text = result.response.text().replace(/```json|```/g, '').trim();
  const enhanced: string[] = JSON.parse(text);

  return scenes.map((s, i) => ({ ...s, imageQuery: enhanced[i] || s.imageQuery }));
}
```

---

## Complete generate.ts Script

```typescript
// scripts/generate.ts
import { writeScript, scriptToScenes, generateCaptions } from '../src/pipeline/gemini';
import { collectAllAssets } from '../src/pipeline/google-search';
import { removeImageBG } from '../src/pipeline/bgremove';
import { generateVoiceover } from '../src/pipeline/voiceover';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';
import fs from 'fs';

interface GenerateOptions {
  style?: string;
  duration?: number;
  lang?: string;
  format?: 'youtube' | 'tiktok' | 'reels' | 'shorts';
  voiceover?: boolean;
}

const FORMATS = {
  youtube:  { width: 1920, height: 1080, fps: 30 },
  tiktok:   { width: 1080, height: 1920, fps: 60 },
  reels:    { width: 1080, height: 1920, fps: 30 },
  shorts:   { width: 1080, height: 1920, fps: 60 },
};

export async function generateVideo(prompt: string, opts: GenerateOptions = {}) {
  const {
    style = 'cinematic',
    duration = 60,
    lang = 'en',
    format = 'youtube',
    voiceover = true,
  } = opts;

  console.log('━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
  console.log('🎬 DIREKTOR — AI Video Director');
  console.log('━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');

  // Step 1
  console.log('\n🧠 Step 1/6 — Writing script with Gemini...');
  const script = await writeScript(prompt, duration, style, lang);
  console.log(`   ✓ Script: ${script.split(' ').length} words`);

  // Step 2
  console.log('\n🎬 Step 2/6 — Breaking into scenes...');
  let scenes = await scriptToScenes(script, style, lang);
  console.log(`   ✓ ${scenes.length} scenes planned`);

  // Step 3
  console.log('\n🌐 Step 3/6 — Collecting images from Google...');
  scenes = await collectAllAssets(scenes, style);
  console.log(`   ✓ ${scenes.length} images collected`);

  // Step 4 — BG removal where needed
  const bgScenes = scenes.filter(s => s.bgRemove);
  if (bgScenes.length > 0) {
    console.log(`\n✂️  Step 4/6 — Removing backgrounds (${bgScenes.length} scenes)...`);
    for (const scene of bgScenes) {
      scene.imageUrl = await removeImageBG(scene.imageUrl);
      console.log(`   ✓ Scene ${scene.id} BG removed`);
    }
  } else {
    console.log('\n✂️  Step 4/6 — No BG removal needed');
  }

  // Step 5 — Voiceover
  const { fps } = FORMATS[format];
  const captions = generateCaptions(scenes, fps);

  if (voiceover) {
    console.log('\n🎙️  Step 5/6 — Generating voiceover...');
    const fullVoiceover = scenes.map(s => s.voiceover).join(' ');
    const audioBuffer = await generateVoiceover(fullVoiceover, lang);
    fs.writeFileSync('public/voiceover.mp3', audioBuffer);
    console.log(`   ✓ Voiceover: ${(audioBuffer.length / 1024).toFixed(0)}KB`);
  } else {
    console.log('\n🎙️  Step 5/6 — Skipping voiceover');
  }

  // Step 6 — Render
  console.log('\n🎞️  Step 6/6 — Rendering video...');
  const { width, height } = FORMATS[format];
  const totalFrames = scenes.reduce((a, s) => a + s.duration * fps, 0);

  const bundled = await bundle({ entryPoint: path.join(__dirname, '../src/index.ts') });
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'CinematicVideo',
    inputProps: { scenes, captions, format },
  });

  const outputPath = `output/${style}-${Date.now()}.mp4`;
  fs.mkdirSync('output', { recursive: true });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { scenes, captions, format },
    onProgress: ({ progress }) => {
      const bar = '█'.repeat(Math.floor(progress * 20)) + '░'.repeat(20 - Math.floor(progress * 20));
      process.stdout.write(`   [${bar}] ${(progress * 100).toFixed(0)}%\r`);
    },
  });

  console.log(`\n\n✅ DONE! → ${outputPath}`);
  console.log(`   Duration: ${(totalFrames / fps).toFixed(0)}s | Format: ${format} (${width}x${height})`);

  return outputPath;
}

// CLI usage
const [,, prompt, ...flags] = process.argv;
const style = flags.find(f => f.startsWith('--style='))?.split('=')[1] || 'cinematic';
const duration = parseInt(flags.find(f => f.startsWith('--duration='))?.split('=')[1] || '60');
const lang = flags.find(f => f.startsWith('--lang='))?.split('=')[1] || 'en';
const format = (flags.find(f => f.startsWith('--format='))?.split('=')[1] || 'youtube') as any;

generateVideo(prompt, { style, duration, lang, format }).catch(console.error);
```

---

## Free Tier Limits

| Model | Free/day | Best for |
|---|---|---|
| gemini-1.5-flash | ~1.5M tokens | Scene breakdown, captions, query enhancement |
| gemini-1.5-pro | ~50K tokens | Script writing (use sparingly) |
| gemini-2.0-flash | ~1.5M tokens | Best quality, same speed as flash |

**Strategy:** `flash` সব কাজে, `pro` শুধু script writing এ।
