#### Core Concept
Duration, delay, and stagger form the **rhythm section** of animation. Master these and you control the pacing, sequence, and musicality of every animation.

#### Deep Dive

**Duration**: The engine's cycle time. Globally, you can set `gsap.defaults({ duration: 1 })` to avoid repetitive declarations.

**Delay**: Can be a number (seconds) OR a function for dynamic sequencing. Function-based delays unlock staggered entrances without the stagger property.

**Stagger**: The secret weapon. Not just a static number—it accepts objects for staggering sophistication.

**Stagger Object Properties**:
- `each`: Time between each element's start
- `from`: Starting point ("start", "center", "edges", "end", "random", or index/array)
- `grid`: `[columns, rows]` for grid-based staggering
- `axis`: For grid, stagger by "x" or "y" axis
- `ease`: Distribute the stagger timings along an ease curve
- `repeat`: Repeat the stagger pattern
- `yoyo`: Reverse on repeat

#### ⚠️ Common Pitfalls & Pro Tips

1. **Nested staggers cause massive delays**: If you stagger a parent timeline with staggered children, the cumulative delay multiplies. Use `immediateRender: false` or adjust timing carefully.

2. **Don't confuse stagger `amount` vs `each`**: `stagger: 0.5` means 0.5s between each. `stagger: { amount: 0.5, from: "start" }` means total time for ALL elements is 0.5s, and timings are distributed automatically.

3. **Stagger on transformOrigin inconsistency**: When staggering elements with different dimensions, set `transformOrigin` individually per element, not in the stagger tween.

#### Example Code

```javascript
// ============================================
// BASIC DURATION & DELAY
// ============================================
gsap.to(".element", {
  x: 400,
  duration: 2,    // Animation plays over 2 seconds
  delay: 0.5       // Waits 0.5s before starting
});

// ============================================
// FUNCTION-BASED DELAY (Dynamic sequencing)
// ============================================
gsap.to(".word", {
  opacity: 1,
  y: 0,
  duration: 0.6,
  // Each element's delay is calculated per index
  delay: (index, target, targets) => {
    // index: position in targets array
    // target: the actual DOM element
    // targets: array of all selected elements
    return index * 0.1; // 100ms between each
  },
  ease: "power3.out"
});

// ============================================
// STAGGER BASICS
// ============================================
gsap.from(".stagger-item", {
  y: 50,
  opacity: 0,
  duration: 0.8,
  stagger: 0.15, // 150ms between each element's start
  ease: "power2.out"
});

// ============================================
// ADVANCED STAGGER CONFIGURATION
// ============================================
gsap.fromTo(
  ".gallery-item",
  { 
    scale: 0.2, 
    opacity: 0,
    rotation: -10 
  },
  {
    scale: 1,
    opacity: 1,
    rotation: 0,
    duration: 0.7,
    stagger: {
      each: 0.08,          // 80ms between starts
      from: "center",      // Animate center elements first
      grid: [4, 3],        // 4 columns, 3 rows grid
      axis: "x",           // Stagger along x-axis within grid
      ease: "power2.inOut", // Stagger timings follow this curve
      repeat: -1,          // Infinite
      yoyo: true,          // Reverse stagger on repeat
      repeatDelay: 1       // Wait 1 second between cycles
    },
    ease: "back.out(1.5)"
  }
);

// ============================================
// STAGGER WITH AMOUNT (Time-based distribution)
// ============================================
// Instead of setting "each" time, set TOTAL time
gsap.to(".list-item", {
  x: 100,
  stagger: {
    amount: 1.5,         // All items animate within 1.5s total
    from: "random",      // Random start order
    ease: "power1.in"    // More items start earlier (eased stagger)
  }
});

// ============================================
// PRO PATTERN: Sequential animation chain with stagger
// ============================================
const tl = gsap.timeline({
  defaults: { duration: 0.6, ease: "power3.out" }
});

tl.from(".header", { y: -50, opacity: 0 })
  .from(".subtitle", { y: -30, opacity: 0 }, "-=0.3") // Overlap
  .from(".card", {
    y: 60,
    opacity: 0,
    stagger: {
      each: 0.15,
      from: "start"
    }
  }, "-=0.2");

// ============================================
// DYNAMIC STAGGER: Function-based stagger values
// ============================================
gsap.to(".bar", {
  scaleY: (index, target) => {
    // Read data attribute for dynamic value
    return target.dataset.height || 1;
  },
  transformOrigin: "bottom",
  duration: 0.5,
  stagger: (index, target) => {
    // Stagger based on element position
    const rect = target.getBoundingClientRect();
    return rect.left / 1000; // Leftmost elements animate first
  }
});
```

---
