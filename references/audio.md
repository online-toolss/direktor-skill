# Audio, Voiceover & Rendering

---

## Voiceover Pipeline

```typescript
// pipeline/voiceover.ts

// Google TTS — Bengali FREE
export async function googleTTS(text: string, lang = 'bn-BD'): Promise<Buffer> {
  const voices = {
    'bn-BD': 'bn-BD-Standard-A',  // Bengali
    'en-US': 'en-US-Neural2-D',   // English male
    'en-GB': 'en-GB-Neural2-B',   // British
    'hi-IN': 'hi-IN-Standard-A',  // Hindi
  };

  const res = await fetch(
    `https://texttospeech.googleapis.com/v1/text:synthesize?key=${process.env.GOOGLE_TTS_KEY}`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        input: { text },
        voice: { languageCode: lang, name: voices[lang] || voices['en-US'] },
        audioConfig: { audioEncoding: 'MP3', speakingRate: 0.95, pitch: -1.0 },
      }),
    }
  );
  const data = await res.json();
  return Buffer.from(data.audioContent, 'base64');
}

// ElevenLabs — Premium quality
export async function elevenLabsTTS(text: string, voiceId = 'pNInz6obpgDQGcFmaJgB'): Promise<Buffer> {
  const res = await fetch(`https://api.elevenlabs.io/v1/text-to-speech/${voiceId}`, {
    method: 'POST',
    headers: {
      'xi-api-key': process.env.ELEVENLABS_API_KEY!,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      text,
      model_id: 'eleven_multilingual_v2',
      voice_settings: { stability: 0.55, similarity_boost: 0.75, style: 0.3 },
    }),
  });
  return Buffer.from(await res.arrayBuffer());
}

// Auto-select based on available keys
export async function generateVoiceover(text: string, lang = 'en'): Promise<Buffer> {
  if (process.env.ELEVENLABS_API_KEY && lang === 'en') {
    return elevenLabsTTS(text);
  }
  return googleTTS(text, lang === 'bn' ? 'bn-BD' : 'en-US');
}
```

---

## Audio in Remotion

```tsx
import { Audio, useCurrentFrame, interpolate } from 'remotion';

// Background music with smart ducking
export const SmartAudio = ({ musicUrl, voiceUrl, voiceStart, voiceDuration }) => {
  const frame = useCurrentFrame();

  const voiceEnd = voiceStart + voiceDuration;
  const isDuringVoice = frame >= voiceStart - 10 && frame <= voiceEnd + 10;

  // Music volume: 0.3 normally, 0.07 during voiceover
  const musicVol = interpolate(
    frame,
    [voiceStart - 10, voiceStart, voiceEnd, voiceEnd + 10],
    [0.28, 0.07, 0.07, 0.28],
    { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
  );

  return (
    <>
      <Audio src={musicUrl} volume={musicVol} loop />
      {voiceUrl && <Audio src={voiceUrl} startFrom={voiceStart} volume={1} />}
    </>
  );
};
```

---

## Export Formats

```typescript
export const FORMATS = {
  youtube:  { width: 1920, height: 1080, fps: 30, codec: 'h264', label: 'YouTube 16:9' },
  shorts:   { width: 1080, height: 1920, fps: 60, codec: 'h264', label: 'YouTube Shorts' },
  tiktok:   { width: 1080, height: 1920, fps: 60, codec: 'h264', label: 'TikTok 9:16' },
  reels:    { width: 1080, height: 1920, fps: 30, codec: 'h264', label: 'Instagram Reels' },
  instagram:{ width: 1080, height: 1080, fps: 30, codec: 'h264', label: 'Instagram Square' },
  twitter:  { width: 1280, height: 720,  fps: 30, codec: 'h264', label: 'Twitter/X' },
  linkedin: { width: 1920, height: 1080, fps: 30, codec: 'h264', label: 'LinkedIn' },
};
```

---

## Render Commands

```bash
# Local render
npx remotion render CinematicVideo out.mp4

# With specific format
npx remotion render SocialReel out.mp4 --width=1080 --height=1920

# Fast (lower quality preview)
npx remotion render CinematicVideo preview.mp4 --jpeg-quality=60 --scale=0.5

# AI generation one-liner
npx ts-node scripts/generate.ts "your prompt" --style=cinematic --format=youtube --lang=en
```
