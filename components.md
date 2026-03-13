# DIREKTOR — All Scene Components (Working Remotion Code)

## 1. KINETIC TEXT SCENE

```tsx
// src/scenes/KineticTextScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const KineticTextScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const lines = scene.primaryText.split("\n");
  const bg = getBg(scene.background, scene.accentColor, frame);

  return (
    <AbsoluteFill style={{ ...bg, justifyContent: "center", alignItems: "center", flexDirection: "column", gap: 12 }}>
      <PatternInterrupt interrupts={scene.patternInterrupts} frame={frame} color={scene.accentColor} />
      {lines.map((line, i) => {
        const progress = spring({ frame: frame - i * 6, fps, config: { damping: 14, stiffness: 120 } });
        const y = interpolate(progress, [0, 1], [60, 0]);
        const opacity = progress;
        return (
          <div key={i} style={{
            transform: `translateY(${y}px)`,
            opacity,
            fontFamily: "'Bebas Neue', sans-serif",
            fontSize: scene.fontSize || 110,
            letterSpacing: 6,
            color: i % 2 === 0 ? "#fff" : scene.accentColor,
            textAlign: "center",
            lineHeight: 0.9,
            textShadow: `0 0 40px ${scene.accentColor}44`,
            padding: "0 40px",
          }}>
            {line}
          </div>
        );
      })}
      {scene.secondaryText && (
        <SubText text={scene.secondaryText} frame={frame} delay={lines.length * 6 + 10} fps={fps} />
      )}
    </AbsoluteFill>
  );
};
```

---

## 2. WORD EXPLOSION SCENE

```tsx
// src/scenes/WordExplosionScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const WordExplosionScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();

  // Phase 1: word grows from 0 → massive (0-12f)
  const growScale = spring({ frame, fps, config: { damping: 10, stiffness: 200 } });
  const bigScale = interpolate(growScale, [0, 1], [0.1, 3.5]);

  // Phase 2: shrink to normal size (12-25f)
  const shrinkProgress = spring({ frame: frame - 12, fps, config: { damping: 18, stiffness: 150 } });
  const finalScale = frame < 12 ? bigScale : interpolate(shrinkProgress, [0, 1], [3.5, 1]);

  // Particles burst at frame 12
  const particles = Array.from({ length: 16 }, (_, i) => {
    const angle = (i / 16) * Math.PI * 2;
    const dist = frame < 12 ? 0 : interpolate(frame - 12, [0, 20], [0, 280], { extrapolateRight: "clamp" });
    const pOpacity = frame < 12 ? 0 : interpolate(frame - 12, [0, 5, 20], [0, 1, 0], { extrapolateRight: "clamp" });
    return { x: Math.cos(angle) * dist, y: Math.sin(angle) * dist, opacity: pOpacity };
  });

  return (
    <AbsoluteFill style={{ background: scene.background === "pure-black" ? "#000" : "#0a0a0a", justifyContent: "center", alignItems: "center" }}>
      {/* Particles */}
      {particles.map((p, i) => (
        <div key={i} style={{
          position: "absolute", width: 8, height: 8, borderRadius: "50%",
          background: scene.accentColor,
          transform: `translate(${p.x}px, ${p.y}px) translate(-50%, -50%)`,
          opacity: p.opacity,
          boxShadow: `0 0 12px ${scene.accentColor}`,
        }} />
      ))}

      {/* Main word */}
      <div style={{
        fontFamily: "'Bebas Neue', sans-serif",
        fontSize: 160,
        color: scene.accentColor,
        letterSpacing: 8,
        transform: `scale(${finalScale})`,
        textShadow: `0 0 ${60 * (frame < 12 ? growScale : 0.3)}px ${scene.accentColor}`,
        zIndex: 10,
      }}>
        {scene.primaryText}
      </div>

      {/* Sub text after explosion */}
      {frame > 25 && (
        <SubText text={scene.secondaryText} frame={frame} delay={25} fps={fps} />
      )}
    </AbsoluteFill>
  );
};
```

---

## 3. COUNTER REVEAL SCENE

```tsx
// src/scenes/CounterRevealScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const CounterRevealScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const mg = scene.motionGraphic || {};
  const from = mg.from ?? 0;
  const to = mg.to ?? 100;

  // Count up over 45 frames
  const progress = spring({ frame, fps, config: { damping: 20, stiffness: 60 } });
  const currentVal = Math.round(interpolate(progress, [0, 1], [from, to]));

  // Scale entrance
  const scaleIn = spring({ frame, fps, config: { damping: 12, stiffness: 180 } });

  // Glow pulse
  const glow = 20 + Math.sin(frame * 0.15) * 10;

  return (
    <AbsoluteFill style={{
      background: scene.background === "dark-gradient"
        ? "linear-gradient(135deg, #0a0a0a 0%, #1a0a1a 100%)"
        : "#000",
      justifyContent: "center", alignItems: "center", flexDirection: "column", gap: 20
    }}>
      <PatternInterrupt interrupts={scene.patternInterrupts} frame={frame} color={scene.accentColor} />

      {/* Big number */}
      <div style={{
        fontFamily: "'Bebas Neue', sans-serif",
        fontSize: 220,
        lineHeight: 0.85,
        color: scene.accentColor,
        transform: `scale(${scaleIn})`,
        textShadow: `0 0 ${glow}px ${scene.accentColor}, 0 0 ${glow * 2}px ${scene.accentColor}44`,
        letterSpacing: -4,
      }}>
        {currentVal}{mg.suffix || ""}
      </div>

      {/* Label */}
      {mg.label && (
        <div style={{
          fontFamily: "'DM Sans', sans-serif",
          fontSize: 28, color: "rgba(255,255,255,0.65)",
          letterSpacing: 2, textTransform: "uppercase",
          opacity: interpolate(frame, [20, 35], [0, 1], { extrapolateRight: "clamp" }),
          textAlign: "center", padding: "0 60px",
        }}>
          {mg.label}
        </div>
      )}

      {/* Bottom accent line */}
      <div style={{
        width: interpolate(frame, [30, 60], [0, 200], { extrapolateRight: "clamp" }),
        height: 2, background: scene.accentColor,
        boxShadow: `0 0 10px ${scene.accentColor}`,
      }} />
    </AbsoluteFill>
  );
};
```

---

## 4. GLITCH BURST SCENE

```tsx
// src/scenes/GlitchBurstScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from "remotion";

export const GlitchBurstScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Glitch pattern: heavy at start, then settles
  const glitchPhase = frame < 15;
  const glitchOffset = glitchPhase ? Math.sin(frame * 80) * 18 : Math.sin(frame * 0.5) * 1;
  const isRGBSplit = glitchPhase || frame % 20 < 2;
  const scanLineY = (frame * 4) % 110;

  const textOpacity = interpolate(frame, [8, 20], [0, 1], { extrapolateRight: "clamp" });

  return (
    <AbsoluteFill style={{ background: "#000", overflow: "hidden" }}>
      {/* RGB Split layers */}
      {isRGBSplit && (
        <>
          <AbsoluteFill style={{ transform: `translateX(${glitchOffset}px)`, mixBlendMode: "screen", opacity: 0.55 }}>
            <div style={{ width: "100%", height: "100%", background: "rgba(255,0,80,0.08)" }} />
          </AbsoluteFill>
          <AbsoluteFill style={{ transform: `translateX(${-glitchOffset}px)`, mixBlendMode: "screen", opacity: 0.55 }}>
            <div style={{ width: "100%", height: "100%", background: "rgba(0,255,255,0.08)" }} />
          </AbsoluteFill>
        </>
      )}

      {/* Scan bar */}
      <div style={{
        position: "absolute", left: 0, right: 0,
        top: `${scanLineY}%`, height: 2,
        background: `linear-gradient(90deg, transparent, ${scene.accentColor}55, transparent)`,
      }} />

      {/* Main text — glitching */}
      <AbsoluteFill style={{ justifyContent: "center", alignItems: "center" }}>
        <div style={{ position: "relative", textAlign: "center", opacity: textOpacity }}>
          {isRGBSplit && (
            <>
              <div style={{ position: "absolute", inset: 0, color: "#ff0050", transform: `translateX(${glitchOffset}px)`, fontFamily: "'Bebas Neue',sans-serif", fontSize: 120, mixBlendMode: "screen" }}>
                {scene.primaryText}
              </div>
              <div style={{ position: "absolute", inset: 0, color: "#00ffff", transform: `translateX(${-glitchOffset}px)`, fontFamily: "'Bebas Neue',sans-serif", fontSize: 120, mixBlendMode: "screen" }}>
                {scene.primaryText}
              </div>
            </>
          )}
          <div style={{ fontFamily: "'Bebas Neue',sans-serif", fontSize: 120, color: "#fff", letterSpacing: 6 }}>
            {scene.primaryText}
          </div>
        </div>
      </AbsoluteFill>

      {/* Error code */}
      <div style={{ position: "absolute", bottom: 60, right: 40, fontFamily: "'Space Mono',monospace", fontSize: 11, color: "rgba(255,255,255,0.2)", letterSpacing: 2 }}>
        ERR_0x{frame.toString(16).padStart(4, "0").toUpperCase()}
      </div>
    </AbsoluteFill>
  );
};
```

---

## 5. SPLIT SCREEN SCENE

```tsx
// src/scenes/SplitScreenScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const SplitScreenScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps, width } = useVideoConfig();

  // Split animation
  const splitProgress = spring({ frame, fps, config: { damping: 16, stiffness: 120 } });
  const splitX = interpolate(splitProgress, [0, 1], [width / 2, width / 2]);

  const left = scene.splitLeft || { text: "BEFORE", bg: "#111" };
  const right = scene.splitRight || { text: "AFTER", bg: scene.accentColor };

  return (
    <AbsoluteFill style={{ overflow: "hidden" }}>
      {/* Left half */}
      <div style={{
        position: "absolute", left: 0, top: 0, bottom: 0,
        width: splitX, background: left.bg,
        display: "flex", alignItems: "center", justifyContent: "center", overflow: "hidden",
      }}>
        <div style={{ fontFamily: "'Bebas Neue',sans-serif", fontSize: 90, color: "rgba(255,255,255,0.4)", letterSpacing: 4, textAlign: "center", padding: "0 30px" }}>
          {left.text}
        </div>
      </div>

      {/* Right half */}
      <div style={{
        position: "absolute", left: splitX, top: 0, bottom: 0, right: 0,
        background: right.bg,
        display: "flex", alignItems: "center", justifyContent: "center", overflow: "hidden",
      }}>
        <div style={{ fontFamily: "'Bebas Neue',sans-serif", fontSize: 90, color: "#fff", letterSpacing: 4, textAlign: "center", padding: "0 30px" }}>
          {right.text}
        </div>
      </div>

      {/* Center divider */}
      <div style={{
        position: "absolute", top: 0, bottom: 0,
        left: splitX - 1, width: 2,
        background: "#fff", boxShadow: "0 0 20px #fff",
      }} />
    </AbsoluteFill>
  );
};
```

---

## 6. NEON GLOW SCENE

```tsx
// src/scenes/NeonGlowScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const NeonGlowScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const flicker = Math.sin(frame * 0.31) > 0.92 ? 0.6 : 1;
  const glow = 20 + Math.sin(frame * 0.08) * 10;
  const enter = spring({ frame, fps, config: { damping: 14, stiffness: 100 } });
  const y = interpolate(enter, [0, 1], [50, 0]);

  return (
    <AbsoluteFill style={{ background: "#030310" }}>
      {/* Grid */}
      <AbsoluteFill style={{
        backgroundImage: `linear-gradient(${scene.accentColor}08 1px, transparent 1px), linear-gradient(90deg, ${scene.accentColor}08 1px, transparent 1px)`,
        backgroundSize: "60px 60px",
      }} />
      {/* Scan line */}
      <div style={{
        position: "absolute", left: 0, right: 0,
        top: `${(frame * 1.5) % 110}%`, height: 2,
        background: `linear-gradient(90deg, transparent, ${scene.accentColor}44, transparent)`,
      }} />
      {/* Text */}
      <AbsoluteFill style={{ justifyContent: "center", alignItems: "center", flexDirection: "column", gap: 20, transform: `translateY(${y}px)` }}>
        <div style={{
          fontFamily: "'Bebas Neue',sans-serif",
          fontSize: 120, letterSpacing: 8,
          color: scene.accentColor,
          opacity: flicker,
          textShadow: `0 0 ${glow}px ${scene.accentColor}, 0 0 ${glow * 2}px ${scene.accentColor}, 0 0 ${glow * 4}px ${scene.accentColor}44`,
        }}>
          {scene.primaryText}
        </div>
        <div style={{ width: 200, height: 1, background: scene.accentColor, boxShadow: `0 0 15px ${scene.accentColor}` }} />
        {scene.secondaryText && (
          <div style={{ fontFamily: "'Space Mono',monospace", fontSize: 16, letterSpacing: 4, color: `${scene.accentColor}88`, textTransform: "uppercase" }}>
            {scene.secondaryText}
          </div>
        )}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

---

## 7. PARTICLE FIELD SCENE

```tsx
// src/scenes/ParticleFieldScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const ParticleFieldScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const particles = Array.from({ length: 45 }, (_, i) => {
    const seed = i * 137.5;
    return {
      x: (seed * 7.3) % 100,
      y: ((frame * (0.04 + (i % 7) * 0.012) + seed * 3.7) % 110) - 5,
      size: 1.5 + (i % 4) * 0.8,
      opacity: (0.06 + (i % 5) * 0.07) * (0.7 + Math.sin(frame * 0.08 + i) * 0.3),
    };
  });

  const textIn = spring({ frame: frame - 10, fps, config: { damping: 14, stiffness: 100 } });
  const y = interpolate(textIn, [0, 1], [40, 0]);

  return (
    <AbsoluteFill style={{ background: "#050510" }}>
      {/* Particles */}
      {particles.map((p, i) => (
        <div key={i} style={{
          position: "absolute",
          left: `${p.x}%`, top: `${p.y}%`,
          width: p.size, height: p.size,
          borderRadius: "50%",
          background: scene.accentColor,
          opacity: p.opacity,
          boxShadow: `0 0 ${p.size * 3}px ${scene.accentColor}`,
          transform: "translate(-50%, -50%)",
        }} />
      ))}
      {/* Text */}
      <AbsoluteFill style={{ justifyContent: "center", alignItems: "center", flexDirection: "column", gap: 24 }}>
        <div style={{
          fontFamily: "'Bebas Neue',sans-serif",
          fontSize: 100, letterSpacing: 6,
          color: "#fff",
          textShadow: `0 0 40px ${scene.accentColor}66`,
          transform: `translateY(${y}px)`,
          opacity: Math.min(textIn, 1),
          textAlign: "center", padding: "0 50px",
        }}>
          {scene.primaryText}
        </div>
        {scene.secondaryText && (
          <div style={{
            fontFamily: "'DM Sans',sans-serif",
            fontSize: 22, color: "rgba(255,255,255,0.6)",
            transform: `translateY(${y}px)`,
            opacity: Math.min(textIn, 1),
            textAlign: "center", padding: "0 60px",
          }}>
            {scene.secondaryText}
          </div>
        )}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

---

## 8. LINE DRAW SCENE

```tsx
// src/scenes/LineDrawScene.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from "remotion";

export const LineDrawScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { width } = useVideoConfig();

  const lineWidth = interpolate(frame, [0, 25], [0, width * 0.7], { extrapolateRight: "clamp" });
  const textOpacity = interpolate(frame, [20, 40], [0, 1], { extrapolateRight: "clamp" });
  const textY = interpolate(frame, [20, 40], [20, 0], { extrapolateRight: "clamp" });

  return (
    <AbsoluteFill style={{ background: "#fff", justifyContent: "center", alignItems: "center", flexDirection: "column", gap: 30 }}>
      {/* Top line */}
      <div style={{ width: lineWidth, height: 2, background: "#111" }} />

      {/* Text */}
      <div style={{
        fontFamily: "'Bebas Neue',sans-serif",
        fontSize: 110, letterSpacing: 8,
        color: "#111",
        opacity: textOpacity,
        transform: `translateY(${textY}px)`,
        textAlign: "center",
      }}>
        {scene.primaryText}
      </div>

      {scene.secondaryText && (
        <div style={{
          fontFamily: "'DM Sans',sans-serif",
          fontSize: 20, color: "#666",
          letterSpacing: 3, textTransform: "uppercase",
          opacity: interpolate(frame, [35, 50], [0, 1], { extrapolateRight: "clamp" }),
        }}>
          {scene.secondaryText}
        </div>
      )}

      {/* Bottom line */}
      <div style={{ width: lineWidth, height: 2, background: "#111" }} />
    </AbsoluteFill>
  );
};
```

---

## 9. IMAGE BG SCENE (শুধু real visual দরকার হলে)

```tsx
// src/scenes/ImageBgScene.tsx
import { AbsoluteFill, Img, useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const ImageBgScene = ({ scene }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Ken Burns zoom
  const zoom = interpolate(frame, [0, scene.duration * 30], [1.05, 1.0]);

  const textIn = spring({ frame: frame - 15, fps, config: { damping: 14, stiffness: 100 } });
  const y = interpolate(textIn, [0, 1], [30, 0]);

  return (
    <AbsoluteFill>
      {/* Image */}
      <AbsoluteFill style={{ transform: `scale(${zoom})` }}>
        <Img src={scene.resolvedImageUrl || "https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?w=1080"}
          style={{ width: "100%", height: "100%", objectFit: "cover" }} />
      </AbsoluteFill>
      {/* Gradient overlay */}
      <AbsoluteFill style={{ background: "linear-gradient(to top, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.15) 60%, transparent 100%)" }} />
      {/* Vignette */}
      <AbsoluteFill style={{ background: "radial-gradient(ellipse at center, transparent 40%, rgba(0,0,0,0.6) 100%)" }} />
      {/* Text */}
      <AbsoluteFill style={{ justifyContent: "flex-end", padding: "0 60px 120px", flexDirection: "column", gap: 16 }}>
        {scene.badge && (
          <div style={{ fontFamily: "'Space Mono',monospace", fontSize: 11, color: scene.accentColor, letterSpacing: 4, textTransform: "uppercase" }}>
            {scene.badge}
          </div>
        )}
        <div style={{
          fontFamily: "'Bebas Neue',sans-serif",
          fontSize: 90, letterSpacing: 4, color: "#fff",
          transform: `translateY(${y}px)`,
          opacity: Math.min(textIn, 1),
          lineHeight: 0.92,
        }}>
          {scene.primaryText}
        </div>
        {scene.secondaryText && (
          <div style={{
            fontFamily: "'DM Sans',sans-serif",
            fontSize: 20, color: "rgba(255,255,255,0.65)", lineHeight: 1.5,
            transform: `translateY(${y}px)`,
            opacity: Math.min(textIn, 1),
            maxWidth: 500,
          }}>
            {scene.secondaryText}
          </div>
        )}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

---

## SHARED HELPERS

```tsx
// src/scenes/helpers.tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring, AbsoluteFill } from "remotion";

// Sub text with fade-up entrance
export const SubText = ({ text, frame, delay, fps }) => {
  const enter = spring({ frame: frame - delay, fps, config: { damping: 14, stiffness: 100 } });
  const y = interpolate(enter, [0, 1], [25, 0]);
  return (
    <div style={{
      fontFamily: "'DM Sans',sans-serif",
      fontSize: 24, color: "rgba(255,255,255,0.65)",
      transform: `translateY(${y}px)`,
      opacity: Math.min(enter, 1),
      textAlign: "center", padding: "0 60px",
      lineHeight: 1.5,
    }}>
      {text}
    </div>
  );
};

// Pattern interrupt overlay
export const PatternInterrupt = ({ interrupts = [], frame, color }) => {
  if (!interrupts.includes("COLOR_FLASH")) return null;
  const flash = frame % 45 < 2 ? 0.15 : 0;
  return (
    <AbsoluteFill style={{ background: color, opacity: flash, pointerEvents: "none", zIndex: 50 }} />
  );
};

// Background helper
export const getBg = (type, accent, frame) => {
  const styles = {
    "pure-black":     { background: "#000" },
    "pure-white":     { background: "#fff" },
    "dark-gradient":  { background: "linear-gradient(135deg, #0a0a0a 0%, #1a0a1a 100%)" },
    "color-gradient": { background: `linear-gradient(135deg, ${accent}22 0%, #000 60%)` },
    "geometric-grid": {
      background: "#050508",
      backgroundImage: `linear-gradient(${accent}06 1px, transparent 1px), linear-gradient(90deg, ${accent}06 1px, transparent 1px)`,
      backgroundSize: "80px 80px",
    },
    "radial-glow": { background: `radial-gradient(ellipse at center, ${accent}11 0%, #000 70%)` },
  };
  return styles[type] || styles["pure-black"];
};
```
