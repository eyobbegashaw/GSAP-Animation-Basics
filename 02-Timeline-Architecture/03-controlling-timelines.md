#### Core Concept
Timeline control is interaction design. GSAP provides a rich API for playing, pausing, reversing, seeking, and manipulating timelines at runtime. Mastering these methods transforms static animations into responsive, interactive experiences.

#### Deep Dive

**Core Control Methods**:
- Basic: `play()`, `pause()`, `resume()`, `restart()`, `reverse()`
- Seeking: `seek()`, `progress()`, `time()`, `duration()`
- Speed: `timeScale()`
- State: `isActive()`, `paused()`, `reversed()`
- Lifecycle: `kill()`, `invalidate()`, `clear()`

**Callback-Driven Architecture**: 
- Timeline callbacks (`onStart`, `onComplete`, `onUpdate`, `onRepeat`, `onReverseComplete`) can drive complex state machines.
- Use `call()` to insert arbitrary functions at precise timeline positions.

**Interactive Patterns**:
- **Scrubbing**: `progress(0.5)` jumps to 50% through the entire timeline.
- **Speed ramping**: `timeScale(2)` double-speed, `timeScale(-1)` plays reversed.
- **Dynamic modification**: Add/remove tweens, change properties at runtime.

#### ⚠️ Common Pitfalls & Pro Tips

1. **`restart()` vs `play(0)`**: `restart()` respects the `paused` state. If timeline is paused, restart doesn't play. After `play(0)` it plays from 0.

2. **Memory leaks with event listeners**: If timelines have event callbacks that reference DOM elements removed from the document, you've got a leak. Always `kill()` timelines when their context is destroyed.

3. **Over-using `timeScale()`**: Radically changing `timeScale(10)` can cause frame drops and visual stuttering. Smooth ramp with `gsap.to(tl, {timeScale: 10, duration: 1})`.

4. **`progress()` vs `time()`**: `progress()` is 0-1 (percentage-based). `time()` is absolute seconds. `progress()` is safer for responsive timelines where `duration()` might change.

#### Example Code

```javascript
// ============================================
// BASIC PLAYBACK CONTROL
// ============================================
const tl = gsap.timeline({
  paused: true, // Start paused for manual control
  onComplete: () => console.log("Finished!")
});

tl.to(".box", { x: 400, duration: 2 })
  .to(".box", { y: 200, duration: 1 })
  .to(".box", { rotation: 360, duration: 1.5 });

// Control via buttons
document.querySelector("#play").addEventListener("click", () => tl.play());
document.querySelector("#pause").addEventListener("click", () => tl.pause());
document.querySelector("#resume").addEventListener("click", () => tl.resume());
document.querySelector("#reverse").addEventListener("click", () => tl.reverse());
document.querySelector("#restart").addEventListener("click", () => tl.restart());

// ============================================
// ADVANCED SEEKING AND PROGRESS
// ============================================
const seekTL = gsap.timeline({ paused: true });

seekTL
  .addLabel("phase1")
  .to(".phase1-el", { opacity: 1, duration: 1 })
  .addLabel("phase2")
  .to(".phase2-el", { x: 200, duration: 1 })
  .addLabel("phase3")
  .to(".phase3-el", { scale: 1.5, duration: 1 });

// Jump to specific label
document.querySelector("#jump-phase2").addEventListener("click", () => {
  seekTL.seek("phase2"); // Jump instantly
  seekTL.play();          // Continue from there
});

// Scrub via slider
document.querySelector("#scrubber").addEventListener("input", (e) => {
  const progress = e.target.value / 100; // 0 to 1
  seekTL.progress(progress).pause();     // Jump and hold
});

// ============================================
// TIME SCALE MANIPULATION
// ============================================
const speedTL = gsap.timeline({ repeat: -1, yoyo: true });

speedTL
  .to(".rotator", { rotation: 360, duration: 3, ease: "none" })
  .to(".pulser", { scale: 1.2, duration: 0.5 }, "<");

// Smooth speed ramp (not jarring)
gsap.to(speedTL, {
  timeScale: 0.2,          // Slow to 20% speed
  duration: 1.5,           // Over 1.5 seconds
  ease: "power2.inOut",
  onComplete: () => {
    // After slowing, wait, then ramp back up
    gsap.to(speedTL, {
      timeScale: 1,         // Resume normal speed
      duration: 1,
      delay: 1,             // Hold slow-mo for 1 second
      ease: "power2.inOut"
    });
  }
});

// ============================================
// INTERACTIVE SCRUBBABLE TIMELINE
// ============================================
const iphoneAnimation = gsap.timeline({ 
  paused: true,
  onUpdate: () => {
    // Update UI indicator showing animation progress
    const pct = Math.round(iphoneAnimation.progress() * 100);
    document.querySelector(".progress-indicator").textContent = `${pct}%`;
  }
});

iphoneAnimation
  .to(".phone", { y: -50, rotation: 15, duration: 1 })
  .to(".screen-content", { opacity: 1, scale: 1, duration: 0.5 })
  .to(".detail-view", { x: 0, opacity: 1, duration: 0.7 })
  .to(".final-message", { scale: 1, duration: 0.5, ease: "back.out(1.7)" });

// Bind to scroll for scrubbing effect
window.addEventListener("scroll", () => {
  const scrollY = window.scrollY;
  const section = document.querySelector(".iphone-section");
  const bounds = section.getBoundingClientRect();
  const sectionHeight = section.offsetHeight;
  const viewportH = window.innerHeight;

  // Calculate progress based on scroll within section
  const start = bounds.top - viewportH * 0.5;
  const end = start + sectionHeight * 0.8;
  let progress = (scrollY - start) / (end - start);

  // Clamp between 0 and 1
  progress = Math.max(0, Math.min(1, progress));

  // Apply to timeline
  iphoneAnimation.progress(progress);
});

// ============================================
// DYNAMIC TIMELINE MANIPULATION
// ============================================
const dynamicTL = gsap.timeline();

function addAnimationToTimeline(element, x, duration) {
  // Add new tween at the end of the timeline
  dynamicTL.to(element, { 
    x: x, 
    duration: duration,
    ease: "power2.inOut"
  });
}

function extendTimeline(additionalTweens) {
  additionalTweens.forEach(tween => {
    dynamicTL.to(tween.target, tween.vars, tween.position);
  });
  // Invalidate to ensure GSAP recalculates
  dynamicTL.invalidate();
}

// Insert a function call at a precise time
dynamicTL
  .to(".step1", { x: 100, duration: 1 })
  .add(() => {
    console.log("Step 1 complete, setting up step 2");
    document.querySelector(".step2").classList.add("active");
  })
  .to(".step2", { x: 200, duration: 1 })
  .call(() => console.log("All steps complete"), null, ">+0.5");

// ============================================
// PRO PATTERN: Timeline State Machine
// ============================================
class AnimationStateMachine {
  constructor() {
    this.currentState = "idle";
    this.timeline = gsap.timeline({ paused: true });

    this.timeline
      .addLabel("idle")
      .to(".element", { scale: 1, duration: 0.3 }, "idle")
      .addLabel("hover")
      .to(".element", { scale: 1.05, duration: 0.3 }, "hover")
      .addLabel("active")
      .to(".element", { scale: 0.95, duration: 0.15 }, "active")
      .to(".element", { scale: 1.08, duration: 0.2 })
      .to(".element", { scale: 1, duration: 0.3 })
      .addLabel("disabled")
      .to(".element", { opacity: 0.5, scale: 1, duration: 0.3 }, "disabled");
  }

  transition(newState) {
    if (newState === this.currentState) return;

    const labels = ["idle", "hover", "active", "disabled"];
    const fromIndex = labels.indexOf(this.currentState);
    const toIndex = labels.indexOf(newState);

    // Calculate direction
    if (toIndex > fromIndex) {
      // Moving forward
      this.timeline.tweenFromTo(
        this.currentState, 
        newState,
        { duration: 0.5 } // Smooth transition
      );
    } else {
      // Moving backward
      this.timeline.tweenFromTo(
        this.currentState,
        newState,
        { duration: 0.5 }
      );
    }

    this.currentState = newState;
  }
}

// Usage
const animState = new AnimationStateMachine();
document.querySelector(".element").addEventListener("mouseenter", 
  () => animState.transition("hover"));
document.querySelector(".element").addEventListener("mouseleave", 
  () => animState.transition("idle"));
document.querySelector(".element").addEventListener("mousedown", 
  () => animState.transition("active"));
```

---
