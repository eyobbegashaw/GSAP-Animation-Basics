#### Core Concept
**Scrubbing** directly maps scroll position to animation progress, creating a physical "scroll drives the animation" relationship. **Pinning** locks an element in place while its scroll-triggered animation plays. Combined, they create the scroll-driven storytelling that's become the hallmark of modern web experiences.

#### Deep Dive

**Scrubbing Nuances**:
- `scrub: true` — 1:1 mapping, animation feels rigidly connected to scroll
- `scrub: number` — Introduces a "lag" (in seconds), creating silky smooth easing
- Higher scrub values = smoother but feels "disconnected" from scroll
- Can scrub ONLY the trigger section (not entire page) using `end` values
- Scrub works with both tweens AND timelines

**Pinning Mechanics**:
- Creates a wrapper div around the pinned element (may break grid layouts)
- `pinSpacing: true` (default) adds padding to prevent content jump
- `pinSpacing: false` for overlapping "layered" pin effects
- `anticipatePin: 1` — Adds 1px tolerance to prevent brief unpin on fast scroll
- `pinReparent: true` — Moves pinned element to <body> during pin (avoids overflow clipping)
- `pinnedContainer` — Reference the container when using nested pinning (critical for complex layouts)

**pinType**: `"fixed"` (default) or `"transform"`. Transform-based pinning is better for elements inside `will-change: transform` containers, preventing jittery fixed positioning.

#### ⚠️ Common Pitfalls & Pro Tips

1. **`pinSpacing: false` and content overlapping**: When you disable spacing, subsequent content will slide UNDER the pinned section. Plan your z-indexing accordingly.

2. **Browser scrollbar jumping**: Pinning can cause the scrollbar to jump if `pinSpacing` latency (the added spacer) isn't accounted for. Always test with actual content below.

3. **`scrub` on timelines with many tweens**: The scrub value applies to the ENTIRE timeline progress, not individual tweens. A 3-second scrub lag on a 1-second timeline means the animation lags heavily.

4. **Mobile browsers handle pinning poorly**: iOS Safari sometimes requires `pinType: "transform"` for smooth pinning. Use `ScrollTrigger.isTouch` to branch logic.

5. **Don't pin elements with `position: sticky` or within scroll containers** — conflicts arise. Stick to standard position contexts.

#### Example Code

```javascript
gsap.registerPlugin(ScrollTrigger);

// ============================================
// BASIC SCRUB CONFIGURATION
// ============================================
// 1:1 rigid scrub
gsap.to(".rigid-scrub", {
  x: "80vw",
  scrollTrigger: {
    trigger: ".scrub-container",
    start: "top 80%",
    end: "+=1000",
    scrub: true,               // Direct 1:1 scroll mapping
    markers: true
  }
});

// Smooth scrubbed scrub (with lag)
gsap.to(".smooth-scrub", {
  x: "80vw",
  rotation: 360,
  scrollTrigger: {
    trigger: ".scrub-container",
    start: "top 80%",
    end: "+=1500",
    scrub: 2,                  // 2-second catch-up lag = buttery smooth
  }
});

// ============================================
// PINNING BASICS
// ============================================
gsap.to(".pinned-element", {
  scale: 2,
  opacity: 0,
  scrollTrigger: {
    trigger: ".pin-container",
    start: "top top",
    end: "+=800",              // Pin for 800px of scroll
    pin: true,                 // Lock the element
    pinSpacing: true,          // Reserve space (default)
    // markers: true,
    onUpdate: (self) => {
      console.log(`Pinned for ${self.progress * 100}% of pin duration`);
    }
  }
});

// ============================================
// ADVANCED PINNING: Layered Sections (pinSpacing: false)
// ============================================
// Section 1 (bottom layer)
gsap.to(".layer-background", {
  y: "-20%",
  ease: "none",
  scrollTrigger: {
    trigger: ".layered-container",
    start: "top top",
    end: "+=2000",
    pin: true,
    pinSpacing: false,        // Don't push content down
    scrub: true,
  }
});

// Section 2 (overlays on top of section 1)
gsap.to(".layer-foreground", {
  opacity: 0,
  scale: 1.2,
  scrollTrigger: {
    trigger: ".layered-container",
    start: "top top",
    end: "+=1000",            // Shorter end = finishes before section 1
    pin: true,
    pinSpacing: false,
    scrub: 1,
  }
});

// ============================================
// PINNED HORIZONTAL SCROLL EFFECT
// ============================================
const horizontalSection = document.querySelector(".horizontal-scroll");
const cardsContainer = horizontalSection.querySelector(".cards-wrapper");

// Calculate the width to scroll
const scrollWidth = cardsContainer.scrollWidth - window.innerWidth;

gsap.to(cardsContainer, {
  x: -scrollWidth,            // Scroll left by full width
  ease: "none",
  scrollTrigger: {
    trigger: horizontalSection,
    start: "top top",
    end: () => `+=${scrollWidth}`, // Dynamic end based on content width
    pin: true,
    scrub: 1,
    anticipatePin: 1,
    invalidateOnRefresh: true,    // Recalculate on resize
  }
});

// ============================================
// SCRUB WITH SCALE-BASED PARALLAX
// ============================================
gsap.to(".parallax-bg", {
  y: "30%",                    // Move down 30% of element height
  ease: "none",
  scrollTrigger: {
    trigger: ".parallax-section",
    start: "top bottom",
    end: "bottom top",
    scrub: true,               // Direct scroll mapping
  }
});

gsap.to(".parallax-fg", {
  y: "-15%",                   // Move UP for foreground
  ease: "none",
  scrollTrigger: {
    trigger: ".parallax-section",
    start: "top bottom",
    end: "bottom top",
    scrub: true,
  }
});

// ============================================
// PROGRESSIVE DISCLOSURE WITH PIN & SCRUB
// ============================================
const disclosureTL = gsap.timeline({
  scrollTrigger: {
    trigger: ".product-section",
    start: "top top",
    end: "+=3000",
    pin: true,
    scrub: 2,                  // Smooth lag
    pinSpacing: true,
  }
});

disclosureTL
  .from(".product-image", { 
    scale: 0.5, 
    opacity: 0, 
    duration: 1 
  })
  .from(".feature-1", { 
    x: -100, 
    opacity: 0 
  })
  .to(".product-image", { 
    rotation: 10,
    scale: 1.1 
  })
  .from(".feature-2", { 
    x: -100, 
    opacity: 0 
  })
  .to(".feature-1", { 
    opacity: 0.3 
  })
  .from(".feature-3", { 
    y: 100, 
    opacity: 0 
  })
  .to(".product-image", { 
    scale: 1.5, 
    filter: "brightness(1.2)" 
  })
  .from(".cta-final", { 
    scale: 0, 
    opacity: 0,
    ease: "back.out(2)" 
  });

// ============================================
// DYNAMIC PINNING BASED ON CONTENT HEIGHT
// ============================================
function setupDynamicPin(section) {
  const content = section.querySelector(".dynamic-content");
  const viewportHeight = window.innerHeight;
  const contentHeight = content.scrollHeight;

  // Calculate pin duration
  let pinDistance;
  if (contentHeight > viewportHeight) {
    // Content larger than viewport: allow full scroll through
    pinDistance = contentHeight - viewportHeight;
  } else {
    // Content smaller: pin for a fixed amount
    pinDistance = Math.max(500, contentHeight * 1.5);
  }

  gsap.to(content, {
    y: -pinDistance,
    ease: "none",
    scrollTrigger: {
      trigger: section,
      start: "top top",
      end: `+=${pinDistance}`,
      pin: true,
      scrub: 0.5,
      anticipatePin: 1,
    }
  });
}

// Apply to all dynamic sections
document.querySelectorAll(".dynamic-pin-section").forEach(setupDynamicPin);

// ============================================
// SCROLL-BASED VIDEO SCRUB (Advanced)
// ============================================
const video = document.querySelector(".scrub-video");
let videoFrames = 0;

// Get video metadata once loaded
video.addEventListener("loadedmetadata", () => {
  videoFrames = Math.floor(video.duration * 30); // 30fps
  setupVideoScrub();
});

function setupVideoScrub() {
  ScrollTrigger.create({
    trigger: ".video-container",
    start: "top top",
    end: "+=2000",
    pin: true,
    scrub: 0.5,
    onUpdate: (self) => {
      // Map scroll progress to video frame
      if (videoFrames > 0) {
        const frame = Math.floor(self.progress * videoFrames);
        video.currentTime = frame / 30;
      }
    }
  });
}

// ============================================
// PRO PATTERN: Multi-section pinned narrative
// ============================================
class ScrollNarrative {
  constructor(sections) {
    this.sections = sections;
    this.masterTL = gsap.timeline({
      scrollTrigger: {
        trigger: ".narrative-container",
        start: "top top",
        end: "+=5000",
        pin: true,
        scrub: 1.5,
      }
    });
    this.buildNarrative();
  }

  buildNarrative() {
    // Each section animates in, holds, then fades
    this.sections.forEach((section, i) => {
      const position = i * (1 / this.sections.length); // Distribute in timeline

      this.masterTL
        .from(section, {
          opacity: 0,
          scale: 0.8,
          duration: 0.2,
        }, position)
        .to(section, {
          opacity: 0,
          scale: 1.2,
          duration: 0.2,
        }, position + 0.15); // Slight overlap
    });
  }
}

// Usage
const narrativeSections = document.querySelectorAll(".story-section");
new ScrollNarrative(narrativeSections);
```

---
