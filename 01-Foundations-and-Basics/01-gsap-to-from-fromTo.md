#### Core Concept
The holy trinity of GSAP tweens. These three methods define **where your animation goes**, **where it comes from**, and **exactly where it starts and ends**.

#### Deep Dive

**`gsap.to()`** — Animates *from current state → defined state*
- Target starts at its natural position and animates TO the values you specify.
- Most commonly used (roughly 70% of all tweens).
- Think: "Go to these values."

**`gsap.from()`** — Animates *from defined state → current state*
- Target starts at the values you specify and animates TO its natural position.
- Think: "Come from these values."
- Critical nuance: The element renders at its natural state first, then immediately jumps to the "from" values and animates back. You MUST set `visibility: hidden` in CSS for elements animating from `autoAlpha: 0` to prevent FOAC (Flash of Animated Content).

**`gsap.fromTo()`** — Animates *from defined state → to defined state*
- Full control over both start AND end states.
- No dependence on the element's current/natural state.
- Perfect when you need deterministic outcomes regardless of current state.

#### ⚠️ Common Pitfalls & Pro Tips

1. **Rendering order with `from()`**: The browser renders the element in its natural CSS state for ~1 frame before GSAP takes over. Solution: Use `fromTo()` when precise control is needed, or set initial CSS with GSAP's `set()`.

2. **Overwriting default durations**: All three methods default to 0.5s duration. Always be explicit in production code.

3. **Property value interpolation**: Strings with mixed units won't animate (e.g., "10px solid red" won't smooth to "20px dashed blue"). Animate numeric parts separately or use CSS variables.

#### Example Code

```javascript
// ============================================
// GSAP TO — Most common, animate to defined values
// ============================================
gsap.to(".box-to", {
  x: 300,           // Move 300px right
  rotation: 360,    // Full spin
  backgroundColor: "#ff6b6b", // Will auto-convert colors
  duration: 1.5,
  ease: "power3.out"
});

// ============================================
// GSAP FROM — Animate FROM defined values to natural state
// ============================================
// CRITICAL: Element must have final styles set in CSS
// The element will JUMP to from values, then animate to natural state
gsap.from(".box-from", {
  x: -500,          // Start 500px left of natural position
  opacity: 0,
  scale: 0.3,
  rotation: -180,
  duration: 1.2,
  ease: "back.out(1.7)" // Overshoot effect
});

// ============================================
// GSAP FROMTO — Full control, independent of current state
// ============================================
gsap.fromTo(
  ".box-fromTo",
  {
    // FROM state (starting values)
    x: -200,
    y: -100,
    opacity: 0,
    borderRadius: "50%",
    scale: 0
  },
  {
    // TO state (ending values)
    x: 200,
    y: 50,
    opacity: 1,
    borderRadius: "10px",
    scale: 1.5,
    duration: 1.5,
    ease: "elastic.out(1, 0.5)",
    repeat: -1,      // Infinite repeat
    yoyo: true       // Reverse on repeat
  }
);

// ============================================
// PRO PATTERN: Staggered entrance with fromTo
// ============================================
gsap.fromTo(
  ".card",
  {
    y: 60,
    opacity: 0,
    rotationX: -15
  },
  {
    y: 0,
    opacity: 1,
    rotationX: 0,
    duration: 0.8,
    stagger: {
      each: 0.12,        // 120ms between each element
      from: "center",    // Start from center element outward
      ease: "power2.out"
    },
    ease: "back.out(1.4)"
  }
);

// ============================================
// ADVANCED: Using gsap.set() to set initial state invisibly
// ============================================
// Prevents flash of unstyled content when using from()
gsap.set(".hero-title", { 
  autoAlpha: 0,  // autoAlpha combines opacity + visibility
  y: 30 
});

// Now animate in (no flash because visibility is already hidden)
gsap.to(".hero-title", {
  autoAlpha: 1,
  y: 0,
  duration: 1,
  delay: 0.5
});
```

---
