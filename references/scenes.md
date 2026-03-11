# Scenes, Animations & Effects — Complete Library

---

## BASE SCENE (সব scene এর ভিত্তি)

```tsx
// components/BaseScene.tsx
import { AbsoluteFill, Img, interpolate, useCurrentFrame, useVideoConfig } from 'remotion';

const COLOR_GRADES = {
  warm:          'sepia(0.25) saturate(1.3) hue-rotate(-10deg) brightness(1.05)',
  cool:          'saturate(0.85) hue-rotate(18deg) brightness(1.1) contrast(1.05)',
  'teal-orange': 'saturate(1.35) hue-rotate(4deg) contrast(1.12)',
  noir:          'grayscale(0.75) contrast(1.35) brightness(0.9)',
  golden:        'sepia(0.45) saturate(1.5) hue-rotate(-15deg) brightness(1.1)',
  arctic:        'saturate(0.6) hue-rotate(22deg) brightness(1.18) contrast(1.08)',
  vintage:       'sepia(0.5) saturate(0.75) contrast(1.15) brightness(0.92)',
  vivid:         'saturate(1.6) contrast(1.08) brightness(1.02)',
  desaturated:   'saturate(0.35) brightness(1.08) contrast(1.12)',
};

export const BaseScene = ({ imageUrl, colorGrade = 'warm', kenBurns = true, vignette = true, grain = true, children }) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();

  // Ken Burns — slow zoom out (cinematic)
  const scale = kenBurns
    ? interpolate(frame, [0, durationInFrames], [1.1, 1.0], { extrapolateRight: 'clamp' })
    : 1;

  // Slight pan (left to right)
  const panX = kenBurns
    ? interpolate(frame, [0, durationInFrames], [-1, 1], { extrapolateRight: 'clamp' })
    : 0;

  return (
    <AbsoluteFill style={{ background: '#000' }}>
      {/* BG Image */}
      <AbsoluteFill style={{
        transform: `scale(${scale}) translateX(${panX}%)`,
        filter: COLOR_GRADES[colorGrade] || 'none',
        transition: 'none',
      }}>
        <Img src={imageUrl} style={{ width: '100%', height: '100%', objectFit: 'cover' }} />
      </AbsoluteFill>

      {/* Gradient overlay */}
      <AbsoluteFill style={{
        background: 'linear-gradient(to top, rgba(0,0,0,0.88) 0%, rgba(0,0,0,0.25) 55%, rgba(0,0,0,0.08) 100%)',
      }} />

      {/* Vignette */}
      {vignette && (
        <AbsoluteFill style={{
          background: 'radial-gradient(ellipse at center, transparent 48%, rgba(0,0,0,0.65) 100%)',
        }} />
      )}

      {/* Film grain */}
      {grain && (
        <AbsoluteFill style={{
          backgroundImage: `url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='g'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23g)'/%3E%3C/svg%3E")`,
          opacity: 0.045,
          mixBlendMode: 'overlay',
          // Animated grain
          transform: `translate(${(frame * 7) % 4}px, ${(frame * 11) % 4}px)`,
        }} />
      )}

      {children}
    </AbsoluteFill>
  );
};
```

---

## ANIMATED TEXT — সব style

```tsx
// components/AnimatedText.tsx
import { interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';

export const AnimatedText = ({ text, animation = 'fadeUp', delay = 0, style = {} }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const f = Math.max(0, frame - delay);

  const animations = {
    // Smooth rise from below
    fadeUp: () => {
      const opacity = interpolate(f, [0, 18], [0, 1], { extrapolateRight: 'clamp' });
      const y = spring({ frame: f, fps, config: { damping: 18, stiffness: 80 }, from: 45, to: 0 });
      return { opacity, transform: `translateY(${y}px)` };
    },

    // Spring slide from left
    slideIn: () => {
      const x = spring({ frame: f, fps, config: { damping: 16, stiffness: 110 }, from: -80, to: 0 });
      const opacity = interpolate(f, [0, 10], [0, 1], { extrapolateRight: 'clamp' });
      return { opacity, transform: `translateX(${x}px)` };
    },

    // Word by word reveal (most cinematic)
    reveal: () => null, // handled below

    // Typewriter
    typewriter: () => null, // handled below

    // Scale pop
    scale: () => {
      const s = spring({ frame: f, fps, config: { damping: 12, stiffness: 90 }, from: 0.3, to: 1 });
      const opacity = interpolate(f, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
      return { opacity, transform: `scale(${s})` };
    },

    // Fade only
    fade: () => {
      const opacity = interpolate(f, [0, 20], [0, 1], { extrapolateRight: 'clamp' });
      return { opacity };
    },
  };

  // REVEAL — word by word (best for headlines)
  if (animation === 'reveal') {
    const words = text.split(' ');
    return (
      <div style={{ ...style, display: 'flex', flexWrap: 'wrap', gap: '0.28em' }}>
        {words.map((word, i) => {
          const wf = Math.max(0, f - i * 5 - delay);
          const opacity = interpolate(wf, [0, 12], [0, 1], { extrapolateRight: 'clamp' });
          const y = spring({ frame: wf, fps, config: { damping: 20, stiffness: 100 }, from: 30, to: 0 });
          return (
            <span key={i} style={{ opacity, transform: `translateY(${y}px)`, display: 'inline-block' }}>
              {word}
            </span>
          );
        })}
      </div>
    );
  }

  // TYPEWRITER
  if (animation === 'typewriter') {
    const chars = Math.floor(interpolate(f, [0, text.length * 2.5], [0, text.length], { extrapolateRight: 'clamp' }));
    return (
      <div style={style}>
        {text.slice(0, chars)}
        <span style={{ opacity: f % 5 < 3 ? 1 : 0, marginLeft: 2 }}>▋</span>
      </div>
    );
  }

  const anim = animations[animation]?.() || {};
  return <div style={{ ...style, ...anim }}>{text}</div>;
};
```

---

## CINEMATIC SCENE — Complete

```tsx
// components/CinematicScene.tsx
export const CinematicScene = ({ imageUrl, badge, headline, subtext, colorGrade = 'warm', textAnimation = 'reveal', showBars = true }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  // Scene fade in/out
  const sceneOpacity = interpolate(
    frame,
    [0, 12, durationInFrames - 12, durationInFrames],
    [0, 1, 1, 0],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity }}>
      <BaseScene imageUrl={imageUrl} colorGrade={colorGrade}>

        {/* Content */}
        <AbsoluteFill style={{
          display: 'flex', flexDirection: 'column',
          justifyContent: 'flex-end',
          padding: showBars ? '100px 120px' : '80px 100px',
        }}>

          {/* Scene badge */}
          {badge && (
            <AnimatedText
              text={badge}
              animation="slideIn"
              delay={8}
              style={{
                fontFamily: '"Space Mono", monospace',
                fontSize: 13, letterSpacing: 5,
                color: '#ff3c3c',
                padding: '5px 14px',
                background: 'rgba(255,60,60,0.12)',
                border: '1px solid rgba(255,60,60,0.3)',
                marginBottom: 18,
                width: 'fit-content',
                textTransform: 'uppercase',
              }}
            />
          )}

          {/* Main headline */}
          <AnimatedText
            text={headline}
            animation={textAnimation}
            delay={14}
            style={{
              fontFamily: '"Bebas Neue", sans-serif',
              fontSize: 108, letterSpacing: 3,
              lineHeight: 0.95,
              color: '#ffffff',
              marginBottom: 22,
              textShadow: '0 6px 30px rgba(0,0,0,0.6)',
            }}
          />

          {/* Subtext */}
          {subtext && (
            <AnimatedText
              text={subtext}
              animation="fadeUp"
              delay={28}
              style={{
                fontFamily: '"DM Sans", sans-serif',
                fontSize: 22, lineHeight: 1.65,
                color: 'rgba(255,255,255,0.72)',
                maxWidth: 680,
              }}
            />
          )}

          {/* Animated accent line */}
          <div style={{
            width: interpolate(frame, [20, 50], [0, 80], { extrapolateRight: 'clamp' }),
            height: 2,
            background: '#ff3c3c',
            boxShadow: '0 0 10px #ff3c3c',
            marginTop: 22,
          }} />
        </AbsoluteFill>

        {/* Scene number watermark */}
        <div style={{
          position: 'absolute', top: showBars ? 90 : 30, right: 50,
          fontFamily: '"Bebas Neue", sans-serif',
          fontSize: 80, color: 'rgba(255,255,255,0.04)',
          userSelect: 'none',
        }}>
          {String(1).padStart(2, '0')}
        </div>

      </BaseScene>

      {/* 2.35:1 Cinematic bars */}
      {showBars && <CinematicBars />}
    </AbsoluteFill>
  );
};

export const CinematicBars = ({ height = 68 }) => (
  <>
    <div style={{ position: 'absolute', top: 0, left: 0, right: 0, height, background: '#000', zIndex: 50 }} />
    <div style={{ position: 'absolute', bottom: 0, left: 0, right: 0, height, background: '#000', zIndex: 50 }} />
  </>
);
```

---

## BACKGROUND REMOVED SUBJECT

```tsx
// components/BGRemoved.tsx — Floating subject with shadow
export const BGRemovedSubject = ({ transparentUrl, delay = 0, position = 'right', scale = 1 }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const f = Math.max(0, frame - delay);

  // Entrance animation
  const y = spring({ frame: f, fps, config: { damping: 14, stiffness: 55 }, from: 100, to: 0 });
  const opacity = interpolate(f, [0, 20], [0, 1], { extrapolateRight: 'clamp' });

  // Floating idle animation
  const float = Math.sin(frame * 0.04) * 10;

  // Shadow breathing
  const shadowBlur = 40 + Math.sin(frame * 0.04) * 10;

  const positions = {
    right:  { right: '8%', bottom: '5%' },
    left:   { left: '8%', bottom: '5%' },
    center: { left: '50%', bottom: '5%', transform: 'translateX(-50%)' },
  };

  return (
    <AbsoluteFill style={{ pointerEvents: 'none' }}>
      <div style={{
        position: 'absolute',
        ...positions[position],
        opacity,
        transform: `translateY(${y + float}px) scale(${scale})`,
        filter: `drop-shadow(0 ${shadowBlur}px 60px rgba(0,0,0,0.8)) drop-shadow(0 10px 20px rgba(0,0,0,0.5))`,
      }}>
        <Img src={transparentUrl} style={{ height: 700, objectFit: 'contain' }} />
      </div>
    </AbsoluteFill>
  );
};
```

---

## MOTION GRAPHICS COMPONENTS

```tsx
// components/MotionGraphics.tsx

// Animated stat counter
export const StatCounter = ({ from = 0, to, suffix = '', prefix = '', style = {} }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const progress = spring({ frame, fps, config: { damping: 18, stiffness: 35 }, from: 0, to: 1 });
  const value = Math.round(from + (to - from) * progress);
  return <div style={style}>{prefix}{value.toLocaleString()}{suffix}</div>;
};

// Progress bar
export const ProgressBar = ({ progress, color = '#ff3c3c', height = 6, label, delay = 0 }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const f = Math.max(0, frame - delay);
  const width = spring({ frame: f, fps, config: { damping: 20, stiffness: 50 }, from: 0, to: progress });
  return (
    <div>
      {label && <div style={{ marginBottom: 8, fontSize: 16, color: 'rgba(255,255,255,0.7)' }}>{label}</div>}
      <div style={{ background: 'rgba(255,255,255,0.08)', height, borderRadius: height, overflow: 'hidden' }}>
        <div style={{
          width: `${width}%`, height: '100%',
          background: `linear-gradient(90deg, ${color}, ${color}cc)`,
          borderRadius: height,
          boxShadow: `0 0 14px ${color}88`,
        }} />
      </div>
    </div>
  );
};

// Lower third
export const LowerThird = ({ title, subtitle, accent = '#ff3c3c', delay = 0 }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const f = Math.max(0, frame - delay);
  const x = spring({ frame: f, fps, config: { damping: 18, stiffness: 90 }, from: -250, to: 0 });
  const opacity = interpolate(f, [0, 10], [0, 1], { extrapolateRight: 'clamp' });
  return (
    <div style={{
      position: 'absolute', bottom: 120, left: 80,
      transform: `translateX(${x}px)`, opacity,
      display: 'flex', alignItems: 'stretch', gap: 0,
    }}>
      <div style={{ width: 4, background: accent, borderRadius: 2, marginRight: 18, boxShadow: `0 0 10px ${accent}` }} />
      <div style={{ background: 'rgba(0,0,0,0.75)', padding: '10px 20px', backdropFilter: 'blur(10px)' }}>
        <div style={{ fontSize: 28, fontWeight: 700, color: '#fff', fontFamily: 'sans-serif' }}>{title}</div>
        {subtitle && <div style={{ fontSize: 16, color: 'rgba(255,255,255,0.6)', marginTop: 2 }}>{subtitle}</div>}
      </div>
    </div>
  );
};

// Floating particles background
export const ParticleField = ({ color = '#ff3c3c', count = 25, speed = 1 }) => {
  const frame = useCurrentFrame();
  const particles = Array.from({ length: count }, (_, i) => {
    const seed = (i * 137.508) % 100;
    const x = seed;
    const yBase = ((frame * speed * (0.05 + (i % 7) * 0.015) + (i * 41.2)) % 110) - 5;
    const size = 1.5 + (i % 4) * 0.8;
    const opacity = 0.08 + (i % 5) * 0.08;
    const pulse = Math.sin(frame * 0.08 + i) * 0.3 + 0.7;
    return { x, y: yBase, size, opacity: opacity * pulse };
  });

  return (
    <AbsoluteFill style={{ pointerEvents: 'none', zIndex: 5 }}>
      {particles.map((p, i) => (
        <div key={i} style={{
          position: 'absolute', left: `${p.x}%`, top: `${p.y}%`,
          width: p.size, height: p.size, borderRadius: '50%',
          background: color, opacity: p.opacity,
          boxShadow: `0 0 ${p.size * 4}px ${color}`,
        }} />
      ))}
    </AbsoluteFill>
  );
};
```

---

## TRANSITIONS

```tsx
// components/Transitions.tsx

// Fade (default, always safe)
export const FadeTransition = ({ children, durationInFrames }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(
    frame,
    [0, 15, durationInFrames - 15, durationInFrames],
    [0, 1, 1, 0],
    { extrapolateRight: 'clamp' }
  );
  return <AbsoluteFill style={{ opacity }}>{children}</AbsoluteFill>;
};

// Zoom in transition
export const ZoomTransition = ({ children }) => {
  const frame = useCurrentFrame();
  const scale = interpolate(frame, [0, 20], [1.18, 1], { extrapolateRight: 'clamp' });
  const opacity = interpolate(frame, [0, 15], [0, 1], { extrapolateRight: 'clamp' });
  return <AbsoluteFill style={{ transform: `scale(${scale})`, opacity }}>{children}</AbsoluteFill>;
};

// Wipe right
export const WipeTransition = ({ children, direction = 'right' }) => {
  const frame = useCurrentFrame();
  const p = interpolate(frame, [0, 20], [0, 100], { extrapolateRight: 'clamp' });
  const clips = {
    right: `inset(0 ${100 - p}% 0 0)`,
    left:  `inset(0 0 0 ${100 - p}%)`,
    up:    `inset(${100 - p}% 0 0 0)`,
    down:  `inset(0 0 ${100 - p}% 0)`,
  };
  return <AbsoluteFill style={{ clipPath: clips[direction] }}>{children}</AbsoluteFill>;
};

// RGB Glitch transition
export const GlitchTransition = ({ children }) => {
  const frame = useCurrentFrame();
  const t = frame / 20;
  const offsetR = Math.sin(t * 80) * 8 * Math.max(0, 1 - t);
  const opacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });
  return (
    <AbsoluteFill style={{ opacity }}>
      <AbsoluteFill style={{ transform: `translateX(${offsetR}px)`, mixBlendMode: 'screen', opacity: 0.4, filter: 'url(#red)' }}>{children}</AbsoluteFill>
      <AbsoluteFill style={{ transform: `translateX(${-offsetR}px)`, mixBlendMode: 'screen', opacity: 0.4, filter: 'url(#cyan)' }}>{children}</AbsoluteFill>
      {children}
    </AbsoluteFill>
  );
};
```

---

## CAPTIONS / SUBTITLES

```tsx
// components/Captions.tsx
export const Captions = ({ captions, style = {} }) => {
  const frame = useCurrentFrame();
  const active = captions.find(c => frame >= c.startFrame && frame <= c.endFrame);
  if (!active) return null;

  const opacity = interpolate(
    frame,
    [active.startFrame, active.startFrame + 6, active.endFrame - 6, active.endFrame],
    [0, 1, 1, 0],
    { extrapolateRight: 'clamp' }
  );

  return (
    <div style={{
      position: 'absolute', bottom: 90, left: '10%', right: '10%',
      textAlign: 'center', opacity, zIndex: 60,
    }}>
      <span style={{
        display: 'inline-block',
        background: 'rgba(0,0,0,0.82)',
        backdropFilter: 'blur(8px)',
        color: '#fff',
        fontSize: 30, lineHeight: 1.5,
        padding: '10px 24px',
        borderRadius: 4,
        fontFamily: '"DM Sans", sans-serif',
        fontWeight: 400,
        ...style,
      }}>
        {active.text}
      </span>
    </div>
  );
};
```
