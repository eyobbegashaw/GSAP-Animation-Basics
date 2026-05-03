#### Core Concept
A **Timeline** is a container that sequences multiple tweens. It's the difference between juggling and choreography. Individual tweens are notes; timelines are the symphony.

#### Deep Dive

**Why Timelines Matter**:
- **Precision sequencing**: No delay math gymnastics
- **Global control**: Pause, resume, reverse, speed up an entire sequence
- **Overlap control**: Position parameter enables sophisticated overlaps
- **Nested timelines**: Compose complex animations from modular pieces
- **Cleanup**: One `kill()` cleans up dozens of tweens

**Constructor Options**:
- `repeat`, `yoyo`, `repeatDelay` — Looping behavior
- `paused` — Create paused, play on demand
- `defaults` — Inherited by all child tweens
- `smoothChildTiming` — Auto-adjust child positions when duration changes
- `onComplete`, `onStart`, `onUpdate` — Lifecycle callbacks

#### ⚠️ Common Pitfalls & Pro Tips

1. **Timelines don't auto-destroy by default**: If you create timelines dynamically (e.g., on hover/click), they accumulate in memory. Use `tl.kill()` or scope them.

2. **`defaults` caveat**: Timeline defaults override tween-level settings. Set `ease: "none"` in timeline defaults only if all children are linear.

3. **Nested timelines scale complexity exponentially**: Master function-returned timelines for clean, composable code. Each function returns its own timeline, then parent timelines `.add()` them.

4. **`paused: true` for interactive timelines**: Create the full animation paused, then control via user interaction (buttons, scroll, events).

#### Example Code

```javascript
// ============================================
// BASIC TIMELINE CREATION
// ============================================
const tl = gsap.timeline({
  // Optional configuration
  repeat: 0,
  yoyo: false,
  defaults: {
    duration: 0.8,
    ease: "power3.out"
  }
});

// Add tweens to timeline
tl.to(".box-1", { x: 300, rotation: 360 })
  .to(".box-2", { y: -200, scale: 1.5 })
  .to(".box-3", { x: -300, borderRadius: "50%" });

// ============================================
// TIMELINE WITH DEFAULTS AND LIFECYCLE HOOKS
// ============================================
const masterTL = gsap.timeline({
  defaults: {
    duration: 0.6,
    ease: "power2.inOut"
  },
  paused: true, // Create paused, play later
  onStart: () => console.log("Sequence started"),
  onComplete: () => {
    console.log("Sequence complete");
    // Auto-destroy to free memory
    masterTL.kill();
  },
  onUpdate: () => {
    // Called on every frame while active
    console.log("Progress:", masterTL.progress());
  }
});

masterTL
  .from(".hero-title", { y: -50, opacity: 0 })
  .from(".hero-subtitle", { y: -30, opacity: 0 })
  .from(".cta-button", { scale: 0, opacity: 0, ease: "back.out(1.7)" });

// Trigger playback later (e.g., on page load or user action)
document.addEventListener("DOMContentLoaded", () => masterTL.play());

// ============================================
// NESTED TIMELINES (Function-returned pattern)
// ============================================
function createEntranceAnimation() {
  const entrance = gsap.timeline({ paused: true });

  entrance
    .from(".element", {
      y: 30,
      opacity: 0,
      stagger: 0.1,
      duration: 0.5,
      ease: "power2.out"
    })
    .from(".element", {
      // Additional animation on same elements
      rotation: -5,
      duration: 0.3,
      ease: "power1.out"
    }, "start"); // Absolute position "start" (time 0)

  return entrance;
}

function createLoopingAnimation() {
  const loop = gsap.timeline({
    repeat: -1,
    yoyo: true,
    repeatDelay: 1
  });

  loop
    .to(".pulse", { scale: 1.1, duration: 0.5, ease: "power1.inOut" })
    .to(".pulse", { opacity: 0.7 }, "<"); // Same start time

  return loop;
}

// Compose master timeline from child timelines
const mainShow = gsap.timeline();

mainShow
  .add(createEntranceAnimation().play()) // Play entrance immediately
  .addLabel("loopStart")
  .add(createLoopingAnimation().play(), "+=0.5") // 0.5s after previous ends
  .add(() => console.log("Looping animation started"), "loopStart");

// ============================================
// DYNAMIC TIMELINE CONSTRUCTION (For iterated content)
// ============================================
function buildGalleryTimeline(items) {
  const gallery = gsap.timeline({ paused: true });

  items.forEach((item, i) => {
    // Build individual item animation
    const itemTL = gsap.timeline();

    itemTL
      .from(item.image, {
        scale: 0.8,
        opacity: 0,
        duration: 0.4
      })
      .from(item.caption, {
        y: 20,
        opacity: 0,
        duration: 0.3
      }, "-=0.1");

    // Add to gallery timeline with stagger effect
    gallery.add(itemTL, i * 0.15);
  });

  return gallery;
}

// Usage
const galleryItems = document.querySelectorAll(".gallery-item");
const galleryAnimation = buildGalleryTimeline(galleryItems);
galleryAnimation.play();

// ============================================
// PRO PATTERN: Timeline with ScrollTrigger
// ============================================
const scrollTL = gsap.timeline({
  scrollTrigger: {
    trigger: ".scroll-section",
    start: "top 80%",     // Trigger when section top hits 80% viewport
    end: "bottom 20%",
    scrub: 1,             // Smooth scrubbing
    markers: false,
    toggleActions: "play none none reverse"
  }
});

scrollTL
  .from(".anim-title", { y: 100, opacity: 0 })
  .from(".anim-card-1", { x: -200, opacity: 0 }, "-=0.2")
  .from(".anim-card-2", { x: 200, opacity: 0 }, "<")
  .from(".anim-card-3", { scale: 0, opacity: 0, ease: "back.out(1.7)" });
```

---
