#### Core Concept
Easing is the **personality** of animation. It's the difference between robotic motion and organic, nuanced movement. GSAP gives you unparalleled control—from built-in ease types, to custom cubic-bezier definitions, to completely bespoke ease configurators.

#### Deep Dive

**GSAP Ease Categories**:
- `power1.in` through `power4.in` (Quad → Quart)
- `sine`, `expo`, `circ` (Each has `.in`, `.out`, `.inOut`)
- `back` — Overshoots (great for pop-in effects)
- `elastic` — Rubber band feel
- `bounce` — Bouncing ball physics
- `rough` — Hand-drawn/jittery aesthetic
- `slow` — Exaggerated slow-motion feel
- `steps()` — Stepped/keyframe animation
- `expoScale()` — Hybrid ease for massive scale changes (prevents clipping)

**Ease Configuration**: The special ones (`back`, `elastic`, `slow`, `rough`, `expoScale`) accept configuration parameters.

**Custom Ease**: Use `CustomEase` (Club GSAP) or `cubic-bezier()` string for ultimate control.

**Ease Visualizer**: Always check `https://gsap.com/docs/v3/Eases/` — visual feedback is essential for picking the right feel.

#### ⚠️ Common Pitfalls & Pro Tips

1. **Default ease is `"power1.out"`**: GSAP tweens default to this. Timelines default to `"none"` (linear). Explicit is always better.

2. **`ease: "none"` ≠ `ease: "linear"`**: They're functionally identical. GSAP's `"none"` is just an alias.

3. **`rough()` can cause performance issues**: The jittery calculation happens every frame. Use sparingly, especially on mobile.

4. **Elastic with short durations looks broken**: `elastic.out` needs sufficient time to settle. Durations under 0.8s often look jarring.

5. **Custom cubic-bezier format**: `"cubic-bezier(0.25, 0.1, 0.25, 1)"` — only x values (time) 0-1. y values (progress) can exceed 1 for overshoot effects.

#### Example Code

```javascript
// ============================================
// CORE EASE TYPES DEMONSTRATION
// ============================================

// Power eases — workhorses of professional animation
// power1=Quad, power2=Cubic, power3=Quart, power4=Quint
gsap.to(".power-demo", {
  x: 400,
  duration: 2,
  ease: "power4.inOut" // Strong acceleration + deceleration
});

// Exponential — fastest start/end, nearly linear middle
gsap.to(".expo-demo", {
  x: 400, 
  duration: 2,
  ease: "expo.out" // Fires off fast, settles slowly
});

// Back ease — overshoots, perfect for UI elements
gsap.from(".modal", {
  scale: 0.5,
  opacity: 0,
  duration: 0.8,
  ease: "back.out(1.7)" // 1.7 = overshoot amount (default: 1.7)
});

// ============================================
// CONFIGURABLE EASE FUNCTIONS
// ============================================

// Elastic — amplitude and period control
gsap.to(".elastic-box", {
  x: 300,
  duration: 2,
  ease: "elastic.out(1, 0.3)" 
  // amplitude: 1 (strength of bounce)
  // period: 0.3 (time per wave, lower = more waves)
});

// Bounce — number of bounces
gsap.to(".ball", {
  y: -200,
  duration: 1.5,
  ease: "bounce.out" // Default bounce count based on duration
});
gsap.to(".ball-custom", {
  y: -200,
  duration: 1.5,
  ease: "bounce.out(4)" // Exactly 4 bounces (if enough time)
});

// ============================================
// ADVANCED: Rough Ease (organic, hand-drawn feel)
// ============================================
gsap.to(".sketch-element", {
  x: 300,
  duration: 2,
  ease: "rough({
    template: power1.out,
    strength: 1,
    points: 20,
    taper: 'out',
    randomize: true,
    clamp: true
  })"
  // template: base ease pattern
  // strength: jitter intensity (default 1)
  // points: sample points for roughness
  // taper: apply roughess to 'in', 'out', or 'both'
  // clamp: don't exceed start/end values
});

// ============================================
// ADVANCED: expoScale for massive scale transforms
// ============================================
// Problem: scaling from 0→100 with normal ease clips viewport
// Solution: expoScale adjusts ease to stay in view then rocket out
gsap.from(".mega-scale", {
  scale: 0,
  duration: 2,
  ease: "expoScale(30, 100, power2.in)"
  // startScale: 30 (where true exponential curve starts)
  // endScale: 100 (target scale)
  // ease: underlying ease to blend with
});

// ============================================
// EASE COMPARISON: Interactive easing demo
// ============================================
const eases = [
  "power1.out", "power2.out", "power3.out", "power4.out",
  "back.out(1.7)", "elastic.out(1, 0.3)", "bounce.out"
];

eases.forEach((ease, index) => {
  gsap.to(`.bar-${index}`, {
    height: "100%",
    duration: 1.2,
    delay: index * 0.1,
    ease: ease,
    repeat: -1,
    yoyo: true,
    repeatDelay: 0.5
  });
});

// ============================================
// CUSTOM EASE: cubic-bezier string
// ============================================
gsap.to(".custom", {
  x: 400,
  duration: 2,
  ease: "cubic-bezier(0.34, 1.56, 0.64, 1)" 
  // Custom overshoot curve
  // x1,y1,x2,y2 — y values > 1 create overshoot
});

// ============================================
// PRO PATTERN: Ease matching for seamless transitions
// ============================================
// When chaining animations, match the OUT of one 
// with the IN of the next for seamless flow

const tl = gsap.timeline();

// Element enters with back.out (overshoots to final position)
tl.from(".element-1", {
  x: -200,
  opacity: 0,
  duration: 1,
  ease: "back.out(1.4)"
})
// Next element: use the INVERSE ease pattern
.from(".element-2", {
  x: -200,
  opacity: 0,
  duration: 0.8,
  // power3.in for strong initial acceleration
  // Complementing the power3.out deceleration of typical UI
  ease: "power3.in"
}, "-=0.3");

// ============================================
// EASE FOR SCROLL-BASED ANIMATION
// ============================================
// Linear is often best for scrubbed scroll animations
gsap.to(".parallax-bg", {
  y: "30%",
  ease: "none", // Linear — direct 1:1 relationship with scroll
  scrollTrigger: {
    trigger: ".section",
    start: "top bottom",
    end: "bottom top",
    scrub: 1
  }
});

// ============================================
// STEPS EASE: For sprite-like animation
// ============================================
gsap.to(".sprite", {
  backgroundPosition: "-500px 0",
  duration: 1,
  ease: "steps(12)" // 12 discrete steps, no interpolation
});
```

---
