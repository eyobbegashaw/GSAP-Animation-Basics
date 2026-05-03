#### Core Concept
`ScrollTrigger` connects animations to scroll position. It's not just a scroll observer—it's a full scroll-driven animation engine that maps vertical/horizontal scroll progress to timeline progress with surgical precision.

#### Deep Dive

**Core Configuration Properties**:
- `trigger`: The element that triggers the animation (or viewport if omitted)
- `start`: When the trigger's position meets the scroller's position ("triggerTop scrollerBottom")
- `end`: When the animation completes
- `scrub`: Smoothly ties animation progress to scroll position (boolean or seconds for lag)
- `markers`: Debug visualization (remove in production)
- `toggleActions`: String of 4 actions: "onEnter onLeave onEnterBack onLeaveBack"
- `onToggle`: Callback with `{ isActive, progress, direction, ... }`

**Start/End Syntax**:
- `"top center"` — When trigger's top edge hits viewport center
- `"bottom 80%"` — When trigger's bottom edge is at 80% viewport height
- `"+=300"` — 300px from default trigger start
- `"top-=100"` — 100px above trigger's top edge
- Keywords: `top`, `center`, `bottom`, `left`, `right`, percentages, pixels

**ScrollTrigger as Standalone**: Can exist without a tween/timeline—just for observing scroll events with callbacks.

#### ⚠️ Common Pitfalls & Pro Tips

1. **Always call `ScrollTrigger.refresh()` after DOM/layout changes**: Content injection, font loading, responsive breakpoints all change element positions. Call `ScrollTrigger.refresh()` or it uses stale positions.

2. **`scrub: true` vs `scrub: number`**: `scrub: true` maps scroll 1:1 with animation progress. `scrub: 2` smooths over 2 seconds of scroll velocity (like a lagging spring).

3. **`pin` and z-index**: Pinned elements get a div wrapper. If your layout breaks on pin, check that wrapper styles don't interfere with grid/flexbox.

4. **Multiple triggers on same element**: ScrollTrigger IDs auto-generate. For dynamic content, provide explicit `id` property to avoid conflicts.

5. **Performance**: ScrollTrigger uses requestAnimationFrame internally. Too many triggers (100+) on a page WILL degrade performance. Batch animations into fewer timelines.

#### Example Code

```javascript
// ============================================
// REGISTER THE PLUGIN (Required)
// ============================================
gsap.registerPlugin(ScrollTrigger);

// ============================================
// BASIC SCROLL TRIGGER — Tween linked to scroll
// ============================================
gsap.to(".scroll-box", {
  x: 500,
  rotation: 360,
  scrollTrigger: {
    trigger: ".scroll-box",          // Element to watch
    start: "top 80%",               // When box top hits 80% viewport
    end: "top 20%",                 // When box top hits 20% viewport
    scrub: true,                    // Animation tied 1:1 to scroll
    markers: true,                  // See start/end visually (debug)
  }
});

// ============================================
// SCROLL TRIGGER WITH TOGGLE ACTIONS
// ============================================
gsap.from(".reveal-element", {
  y: 100,
  opacity: 0,
  duration: 1,
  scrollTrigger: {
    trigger: ".reveal-element",
    start: "top bottom",
    end: "top center",
    // onEnter, onLeave, onEnterBack, onLeaveBack
    toggleActions: "play none none reverse"
    // Play on enter, do nothing on leave,
    // do nothing on re-enter from bottom, reverse on leave back to top
  }
});

// ============================================
// SCROLL TRIGGER AS STANDALONE OBSERVER
// (No animation, just scroll-based callbacks)
// ============================================
ScrollTrigger.create({
  trigger: ".milestone-section",
  start: "top center",
  end: "bottom center",
  onToggle: (self) => {
    if (self.isActive) {
      console.log("User is viewing the milestone section!");
      document.querySelector(".indicator").classList.add("active");
    } else {
      document.querySelector(".indicator").classList.remove("active");
    }
  },
  onEnter: () => console.log("Entered"),
  onLeave: () => console.log("Left"),
  onEnterBack: () => console.log("Entered from bottom"),
  onLeaveBack: () => console.log("Left to top"),
});

// ============================================
// MULTIPLE TRIGGERS — Batch animation pattern
// ============================================
const sections = gsap.utils.toArray(".fade-section");
sections.forEach((section) => {
  gsap.from(section, {
    autoAlpha: 0, // opacity + visibility combined
    y: 60,
    duration: 1,
    scrollTrigger: {
      trigger: section,
      start: "top 85%",
      end: "top 30%",
      scrub: false,                   // One-shot on scroll
      toggleActions: "play none none reverse",
      // markers: true,
    }
  });
});

// ============================================
// SCROLL TRIGGER WITH DYNAMIC START/END
// (Responsive to viewport or content)
// ============================================
gsap.to(".responsive-anim", {
  x: "50vw",
  scrollTrigger: {
    trigger: ".responsive-anim",
    start: () => {
      // Dynamic start based on viewport
      if (window.innerWidth < 768) {
        return "top 90%"; // Mobile: start later
      } else {
        return "top 70%"; // Desktop: start earlier
      }
    },
    end: () => `+=${document.querySelector('.responsive-anim').offsetHeight * 2}`,
    scrub: 1,
  }
});

// ============================================
// SCROLL TRIGGER WITH TIMELINE
// ============================================
const scrollTimeline = gsap.timeline({
  scrollTrigger: {
    trigger: ".timeline-section",
    start: "top top",
    end: "+=2000",               // 2000px of scrolling
    scrub: 2,                    // Smooth scrub with 2s lag
    pin: true,                   // Pin section during scroll
    anticipatePin: 1,            // Prevents brief unpin flicker
    markers: false,
  }
});

scrollTimeline
  .from(".step-1", { x: "-100%", opacity: 0 })
  .from(".step-2", { y: "50vh", opacity: 0 })
  .from(".step-3", { x: "100%", opacity: 0 })
  .from(".step-4", { scale: 0, opacity: 0, ease: "back.out(1.7)" });

// ============================================
// SCROLL TRIGGER WITH CALLBACKS
// ============================================
gsap.to(".callback-element", {
  scale: 1.5,
  scrollTrigger: {
    trigger: ".callback-section",
    start: "top center",
    end: "bottom center",
    scrub: true,
    onUpdate: (self) => {
      // self.progress: 0 to 1
      // self.direction: 1 (down) or -1 (up)
      // self.velocity: scroll speed
      const progress = self.progress.toFixed(2);
      document.querySelector(".progress-display").textContent = 
        `Progress: ${progress}, Direction: ${self.direction}`;

      // Change opacity based on progress
      gsap.to(".overlay", {
        opacity: self.progress * 0.5,
        duration: 0.1 // Quick update for smoothness
      });
    },
    onEnter: () => {
      // Fire analytics event
      console.log("User scrolled to animation point");
    }
  }
});

// ============================================
// PRO PATTERN: ScrollTrigger with media queries
// ============================================
ScrollTrigger.matchMedia({
  // Mobile (< 768px)
  "(max-width: 767px)": function() {
    gsap.to(".mobile-only", {
      x: 0,
      opacity: 1,
      scrollTrigger: {
        trigger: ".mobile-section",
        start: "top 90%",
        toggleActions: "play none none reverse"
      }
    });
  },

  // Tablet (768px - 1024px)
  "(min-width: 768px) and (max-width: 1024px)": function() {
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: ".tablet-section",
        start: "top 80%",
        end: "bottom 20%",
        scrub: 1
      }
    });
    tl.to(".tablet-el", { x: 200, rotation: 90 });
  },

  // Desktop (> 1024px)
  "(min-width: 1025px)": function() {
    gsap.from(".desktop-parallax", {
      y: "30%",
      ease: "none",
      scrollTrigger: {
        trigger: ".desktop-section",
        start: "top bottom",
        end: "bottom top",
        scrub: true
      }
    });
  },

  // All breakpoints
  "all": function() {
    // Common animations for all sizes
    ScrollTrigger.create({
      trigger: ".global-trigger",
      start: "top bottom",
      onEnter: () => console.log("Seen on all devices!")
    });
  }
});

// ============================================
// DYNAMIC SCROLLTRIGGER REFRESH
// ============================================
// After AJAX content load, DOM changes, etc.
function handleContentLoaded() {
  // Wait for DOM updates to render
  requestAnimationFrame(() => {
    ScrollTrigger.refresh(); // Recalculate all trigger positions
  });
}

// Font loading can shift layout
document.fonts.ready.then(() => {
  ScrollTrigger.refresh();
});
```

---
