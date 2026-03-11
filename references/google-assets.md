# Google Image Search — Serper API

Pexels/Unsplash এর চেয়ে অনেক বেশি powerful। Real Google results।
**Free: 2,500 queries/month** — পুরো মাসে অনেক video বানানো যাবে।

---

## Setup

```bash
# Get free API key: https://serper.dev
# Free tier: 2,500 queries/month
```

---

## Core Image Search

```typescript
// pipeline/google-search.ts

interface ImageResult {
  url: string;
  title: string;
  source: string;
  width?: number;
  height?: number;
}

export async function googleImageSearch(
  query: string,
  options: {
    count?: number;
    safe?: boolean;
    hd?: boolean;
  } = {}
): Promise<ImageResult[]> {
  const { count = 5, safe = true, hd = true } = options;

  // Make query more specific for better results
  const enhancedQuery = hd
    ? `${query} high quality HD 4K`
    : query;

  const res = await fetch('https://google.serper.dev/images', {
    method: 'POST',
    headers: {
      'X-API-KEY': process.env.SERPER_API_KEY!,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      q: enhancedQuery,
      num: count,
      safe: safe ? 'active' : 'off',
    }),
  });

  const data = await res.json();
  return (data.images || []).map((img: any) => ({
    url: img.imageUrl,
    title: img.title,
    source: img.source,
    width: img.imageWidth,
    height: img.imageHeight,
  }));
}

// Best single image for a scene
export async function getBestImage(query: string): Promise<string> {
  const results = await googleImageSearch(query, { count: 5, hd: true });

  // Prefer landscape, high resolution images
  const sorted = results
    .filter(r => r.url && r.url.match(/\.(jpg|jpeg|png|webp)/i))
    .sort((a, b) => {
      // Prefer wider images for video backgrounds
      const aScore = (a.width || 0) / (a.height || 1);
      const bScore = (b.width || 0) / (b.height || 1);
      return bScore - aScore;
    });

  return sorted[0]?.url || '';
}
```

---

## Smart Query Builder

Gemini generated queries কে আরো powerful করো:

```typescript
// Enhance scene query for best Google results
export function buildImageQuery(scene: Scene, style: string): string {
  const styleModifiers: Record<string, string> = {
    cinematic:  'cinematic photography film still dramatic lighting bokeh',
    neon:       'neon lights cyberpunk dark night city glow',
    minimal:    'minimal clean white background aesthetic',
    nature:     'nature landscape photography golden hour',
    corporate:  'professional business modern office clean',
    retro:      'vintage retro film grain aesthetic',
    glitch:     'digital abstract technology dark',
    dark:       'dark moody atmospheric dramatic',
  };

  const modifier = styleModifiers[style] || 'HD high quality';
  return `${scene.imageQuery} ${modifier} wallpaper`;
}
```

---

## Fallback Chain

Google → Unsplash → Pexels → Solid Color

```typescript
export async function getSceneImage(query: string): Promise<string> {
  // 1. Try Google (Serper)
  try {
    const url = await getBestImage(query);
    if (url) return url;
  } catch (e) {
    console.log('Google search failed, trying Unsplash...');
  }

  // 2. Try Unsplash
  try {
    const res = await fetch(
      `https://api.unsplash.com/search/photos?query=${encodeURIComponent(query)}&per_page=1&orientation=landscape`,
      { headers: { Authorization: `Client-ID ${process.env.UNSPLASH_ACCESS_KEY}` } }
    );
    const data = await res.json();
    const url = data.results?.[0]?.urls?.regular;
    if (url) return url;
  } catch (e) {
    console.log('Unsplash failed, trying Pexels...');
  }

  // 3. Try Pexels
  try {
    const res = await fetch(
      `https://api.pexels.com/v1/search?query=${encodeURIComponent(query)}&per_page=1`,
      { headers: { Authorization: process.env.PEXELS_API_KEY! } }
    );
    const data = await res.json();
    const url = data.photos?.[0]?.src?.large2x;
    if (url) return url;
  } catch (e) {
    console.log('Pexels failed, using solid color...');
  }

  // 4. Solid color fallback
  return `https://via.placeholder.com/1920x1080/0a0a0f/ffffff?text=${encodeURIComponent(query)}`;
}
```

---

## Video Search (Motion Background)

```typescript
export async function googleVideoSearch(query: string): Promise<string> {
  // Search for video content
  const res = await fetch('https://google.serper.dev/videos', {
    method: 'POST',
    headers: {
      'X-API-KEY': process.env.SERPER_API_KEY!,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ q: `${query} free stock video footage` }),
  });
  const data = await res.json();

  // Return first result link (Pexels/Pixabay video links work well)
  return data.videos?.[0]?.link || '';
}
```

---

## Batch Collection (All Scenes at Once)

```typescript
// Collect all scene images in parallel — fast!
export async function collectAllAssets(scenes: Scene[], style: string): Promise<Scene[]> {
  console.log(`🌐 Collecting ${scenes.length} images from Google...`);

  const enriched = await Promise.all(
    scenes.map(async (scene, i) => {
      const query = buildImageQuery(scene, style);
      const imageUrl = await getSceneImage(query);
      console.log(`  ✓ Scene ${i + 1}: ${query.slice(0, 50)}...`);
      return { ...scene, imageUrl };
    })
  );

  return enriched;
}
```
