#### Core Concept
The **Position Parameter** is a superpower. It controls *when* a tween starts relative to the timeline. Without it, tweens play sequentially (one after another). With it, you craft complex overlapping rhythms, offsets, and precise temporal architecture.

#### Deep Dive

**Position Parameter Types**:

1. **Absolute time (number)**: `0` = start of timeline, `1` = 1 second in.
2. **Relative offset (string)**:
   - `"+=0.5"` — 0.5 seconds after the **previous tween's end**
   - `"-=0.5"` — 0.5 seconds before the **previous tween's end** (overlap)
   - `">"` — At the **previous tween's end** (immediate sequence)
   - `"<"` — At the **previous tween's start** (simultaneous start)
   - `">+0.3"` — 0.3 seconds after previous tween's end
   - `"<0.3"` — 0.3 seconds after previous tween's start
3. **Label reference**: `"myLabel"`, `"myLabel+=0.5"`, `"myLabel-=0.2"`
4. **Label creation**: You can create a position and label simultaneously.

#### ⚠️ Common Pitfalls & Pro Tips

1. **The `"<"` confusion**: It's "start with the previous tween's START", not "the previous start time slot." In staggered contexts, each tween's previous is the individual before it.

2. **Overlapping too many tweens**: With `"<"`, many tweens start simultaneously. GSAP handles this beautifully, but too many overlapping transforms on the same element can create unexpected visual states. Always test.

3. **Label creation mid-chain**: `tl.to(".el", {x:100}, "labelName")` creates label "labelName" AT that animation's start. You can reference it later with `tl.to(".el2", {y:50}, "labelName+=0.5")`.

4. **Position parameter in nested timelines**: When adding a child timeline, the position parameter sets when that child timeline *starts playing* within the parent timeline, not when it finishes.

#### Example Code

```javascript
// ============================================
// POSITION PARAMETER BASICS
// ============================================
const tl = gsap.timeline();

// Immediate sequential (default) — each waits for previous to finish
tl.to(".a", { x: 200, duration: 1 })           // Starts at 0
  .to(".b", { x: 200, duration: 1 })           // Starts at 1
  .to(".c", { x: 200, duration: 1 });          // Starts at 2

// ============================================
// ABSOLUTE POSITIONING
// ============================================
const tlAbs = gsap.timeline();

tlAbs.to(".a", { x: 200, duration: 1 })         // Starts at 0
     .to(".b", { x: 200, duration: 1 }, 0.5)   // Starts at 0.5 (overlaps!)
     .to(".c", { x: 200, duration: 1 }, 0);    // Starts at 0 (aligned with .a)

// ============================================
// RELATIVE POSITIONING — The Real Power
// ============================================
const tlRel = gsap.timeline();

tlRel
  // += (after end)
  .to(".step-1", { x: 200, duration: 0.8 })
  .to(".step-2", { x: 200, duration: 0.8 }, "+=0.3") // 0.3s gap
  .to(".step-3", { x: 200, duration: 0.8 }, "+=0.3") // 0.3s gap

  // -= (overlap — before previous ends)
  .to(".overlap-1", { y: 100, duration: 1 }, "+=0.5")
  .to(".overlap-2", { y: 100, duration: 1 }, "-=0.5") // Overlap 0.5s
  .to(".overlap-3", { y: 100, duration: 1 }, "-=0.5") // Overlap 0.5s

  // < (at start of previous)
  .to(".simul-1", { scale: 1.5, duration: 1 }, "+=0.5")
  .to(".simul-2", { rotation: 360, duration: 1 }, "<") // Start together!
  .to(".simul-3", { borderRadius: "50%", duration: 1 }, "<"); // All three in sync!

// ============================================
// LABEL-BASED POSITIONING
// ============================================
const tlLabels = gsap.timeline();

tlLabels
  .addLabel("start")
  .to(".hero", { opacity: 1, duration: 0.5 }, "start")

  // Create label "cardsStart" AT this tween's start position
  .addLabel("cardsEnter", "start+=0.3") // Label without a tween
  .from(".card", {
    y: 50,
    opacity: 0,
    stagger: 0.1,
    duration: 0.4
  }, "cardsEnter")

  // Reference label with offset
  .to(".title", { x: 20, duration: 0.3 }, "cardsEnter+=0.8")

  // Before a label
  .to(".pre-cards", { opacity: 0, duration: 0.3 }, "cardsEnter-=0.3")

  // End alignment with label
  .addLabel("exitPoint", "start+=3"); // Absolute time from timeline start

// ============================================
// COMPLEX CHOREOGRAPHY PATTERN
// ============================================
const choreo = gsap.timeline({
  defaults: { duration: 0.6, ease: "power2.out" }
});

choreo
  // Phase 1: Entrance (all start together with <)
  .from(".logo", { y: -50, opacity: 0 })
  .from(".nav-item", { x: -20, opacity: 0, stagger: 0.05 }, "<0.2") // 0.2 after logo starts
  .from(".cta", { scale: 0, opacity: 0, ease: "back.out(1.7)" }, "<0.3")

  // Gap between phases
  .addLabel("heroPhase", "+=0.5")

  // Phase 2: Hero section (staggered but overlapping)
  .from(".hero-title", { y: 100, opacity: 0 }, "heroPhase")
  .from(".hero-subtitle", { y: 50, opacity: 0 }, "heroPhase+=0.2")
  .from(".hero-image", { 
    scale: 0.8, 
    opacity: 0, 
    duration: 1,
    ease: "power3.out" 
  }, "heroPhase+=0.1") // Overlaps with title

  // Phase 3: Cards cascade (using previous tween start <)
  .addLabel("cards", ">+0.4")
  .from(".card-1", { x: -100, opacity: 0 }, "cards")
  .from(".card-2", { y: 100, opacity: 0 }, "cards+=0.1")
  .from(".card-3", { x: 100, opacity: 0 }, "cards+=0.2")
  .from(".card-cta", { 
    opacity: 0, 
    duration: 0.4 
  }, ">+0.2"); // After all cards finish

// ============================================
// PRO PATTERN: Responsive positioning with 
// function-based labels
// ============================================
function createResponsiveSequence(isMobile) {
  const tl = gsap.timeline();

  if (isMobile) {
    // Simpler, faster sequence for mobile
    tl.from(".hero", { opacity: 0, duration: 0.3 })
      .from(".content", { y: 20, opacity: 0 }, "<0.1");
  } else {
    // Rich desktop sequence with overlaps
    tl.from(".hero", { y: 80, opacity: 0, duration: 0.8 })
      .from(".hero-decor", { 
        scale: 0, 
        duration: 1, 
        ease: "elastic.out(1, 0.4)" 
      }, "<0.3")
      .from(".content", { 
        x: -50, 
        opacity: 0,
        duration: 0.6 
      }, "-=0.2"); // Overlap with hero
  }

  return tl;
}

// Usage: Detect viewport and create appropriate sequence
const isMobile = window.innerWidth < 768;
const responsiveTL = createResponsiveSequence(isMobile);
```

---
