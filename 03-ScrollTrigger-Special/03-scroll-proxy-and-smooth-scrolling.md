## Core Concept
ScrollTrigger's `scroller` property lets you target custom scroll containers, not just `window`. **Scroll smoothing** (with lenticular libraries like Lenis or Locomotive Scroll) requires a **proxy** — an intermediary that synchronizes the native scroll position with the smooth scroll library's internal virtual scroll state. This is industrial-grade scroll orchestration.

#### Deep Dive

**Custom Scroll Containers**:
- Any element with `overflow: scroll/auto` can become a `scroller`
- The `scroller` property must reference the scroll container element
- Nested scroll containers work: each ScrollTrigger binds to its specified scroller

**The Scroll Proxy Problem**:
- Libraries like Lenis hijack smooth scrolling using `transform` on a wrapper
- The native `window.scrollY` becomes unreliable (it stays at 0 or jumps erratically)
- ScrollTrigger reads native scroll position, so animations break
- Solution: Create a proxy object that ScrollTrigger reads/writes instead

**How a ScrollProxy Works**:
1. Create a plain object: `{ y: 0 }`
2. Override ScrollTrigger's scroll getters: `ScrollTrigger.scrollerProxy(target, proxyObject)`
3. The smooth scroll library updates `proxyObject.y` on every frame
4. ScrollTrigger reads from the proxy, not native scroll
5. `ScrollTrigger.update()` is called on each smooth scroll frame

**Lenis Integration Pattern**:
- Lenis dispatches a 'scroll' event with the current smooth position
- In the event handler, update proxy.y and call `ScrollTrigger.update()`
- This synchronizes GSAP ScrollTrigger with the virtual scroll position

#### ⚠️ Common Pitfalls & Pro Tips

1. **Forgetting `.refresh()` on content height changes**: Smooth scroll wrappers change the page height. Failing to `ScrollTrigger.refresh()` means start/end points are stale.

2. **Double scroll bars**: If the smooth scroller creates a wrapper with its own overflow and `height`, ensure the body overflow is hidden to prevent browser-native scrollbar fighting with smooth scroll.

3. **`scroller` property conflicts**: If you set `scroller` to a custom element, ALL start/end calculations use that element's scroll metrics. Window-based triggers won't work unless you switch scrollers.

4. **Lenis `wrapper` vs `content`**: Lenis wraps your content in a smooth scroll container. The wrapper is fixed, the content translates. Understanding this DOM structure is key to debugging layout shifts.

5. **Touch devices and smooth scroll**: Many smooth scroll libraries struggle with iOS momentum scrolling. Test extensively on physical devices.

#### Example Code

```javascript
gsap.registerPlugin(ScrollTrigger);

// ============================================
// CUSTOM SCROLL CONTAINER (Non-window scroller)
// ============================================
const scrollContainer = document.querySelector(".custom-scroll-container");

// Animate element INSIDE a custom scroll container
gsap.to(".inner-animated", {
  x: 200,
  scrollTrigger: {
    trigger: ".inner-section",
    start: "top 80%",
    end: "bottom 20%",
    scroller: scrollContainer,    // Target this container, not window
    scrub: true,
    // markers: true,
  }
});

// Pin INSIDE a custom scroll container
gsap.to(".inner-pin", {
  opacity: 0.5,
  scrollTrigger: {
    trigger: ".inner-pin-container",
    start: "top top",
    end: "+=500",
    scroller: scrollContainer,
    pin: true,
    pinSpacing: true,
  }
});

// ============================================
// NESTED SCROLL CONTAINERS
// ============================================
const outerScroller = document.querySelector(".outer-container");
const innerScroller = document.querySelector(".inner-container");

// Animation that triggers with outer scroll
gsap.from(".outer-content", {
  y: 50,
  opacity: 0,
  scrollTrigger: {
    trigger: ".outer-content",
    start: "top 80%",
    scroller: outerScroller,
    toggleActions: "play none none reverse"
  }
});

// Different animation bound to inner scroll
gsap.from(".inner-content", {
  x: -50,
  opacity: 0,
  scrollTrigger: {
    trigger: ".inner-content",
    start: "top 90%",
    scroller: innerScroller,
    toggleActions: "play none none reverse"
  }
});

// ============================================
// LENIS SMOOTH SCROLL INTEGRATION
// ============================================
// (Assumes Lenis is loaded via script tag or import)

// Step 1: Initialize Lenis
const lenis = new Lenis({
  duration: 1.2,            // Scroll duration (smoothness)
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // Exponential ease
  direction: 'vertical',
  gestureDirection: 'vertical',
  smooth: true,
  mouseMultiplier: 1,
  smoothTouch: false,
  touchMultiplier: 2,
  infinite: false,
});

// Step 2: Create proxy object
const scrollProxy = { y: 0 };

// Step 3: Configure ScrollTrigger to use the proxy
ScrollTrigger.scrollerProxy(document.body, {
  scrollTop(value) {
    if (arguments.length) {
      // SETTER: When to Lenis
      // (Usually not needed, Lenis controls scroll position)
      return; 
    }
    // GETTER: Return the smooth scroll position from proxy
    return scrollProxy.y;
  },
  // getBoundingClientRect sometimes needed for body
  getBoundingClientRect() {
    return {
      top: 0,
      left: 0,
      width: window.innerWidth,
      height: window.innerHeight
    };
  },
  // Lenis handles pinning differently
  pinType: document.body.style.transform ? "transform" : "fixed"
});

// Step 4: Update proxy on each Lenis frame
lenis.on('scroll', ({ scroll, limit, velocity, direction, progress }) => {
  // Update the proxy with current smooth scroll position
  scrollProxy.y = scroll;
  
  // CRITICAL: Tell ScrollTrigger to recalculate
  ScrollTrigger.update();
});

// Step 5: RAF loop for Lenis (it needs continuous calling)
function raf(time) {
  lenis.raf(time);
  requestAnimationFrame(raf);
}
requestAnimationFrame(raf);

// ============================================
// NOW CREATE SCROLLTRIGGERS NORMALLY
// ============================================
// They'll work with smooth scrolling transparently

// Parallax with smooth scroll
gsap.to(".smooth-parallax", {
  y: "20%",
  ease: "none",
  scrollTrigger: {
    trigger: ".smooth-section",
    start: "top bottom",
    end: "bottom top",
    scrub: true,
  }
});

// Pinned timeline with smooth scroll
const smoothTL = gsap.timeline({
  scrollTrigger: {
    trigger: ".smooth-pin-section",
    start: "top top",
    end: "+=2000",
    pin: true,
    scrub: 1.5,
    anticipatePin: 1,
  }
});

smoothTL
  .from(".smooth-card-1", { y: 100, opacity: 0 })
  .from(".smooth-card-2", { y: 100, opacity: 0 })
  .from(".smooth-card-3", { y: 100, opacity: 0 })
  .to(".smooth-card-1", { opacity: 0.3 })
  .to(".smooth-card-2", { opacity: 0.3 });

// ============================================
// LOCOMOTIVE SCROLL INTEGRATION (Alternative)
// ============================================
// (Assumes Locomotive Scroll v4 or later)

// import LocomotiveScroll from 'locomotive-scroll';
// const locoScroll = new LocomotiveScroll();

// let locoProxy = { y: 0 };

// locoScroll.on('scroll', (obj) => {
//   locoProxy.y = obj.scroll.y;
//   ScrollTrigger.update();
// });

// ScrollTrigger.scrollerProxy('.smooth-scroll-container', {
//   scrollTop(value) {
//     return arguments.length 
//       ? locoScroll.scrollTo(value, 0, { disableLerp: true }) 
//       : locoProxy.y;
//   },
//   getBoundingClientRect() {
//     return {
//       top: 0, left: 0,
//       width: window.innerWidth,
//       height: window.innerHeight
//     };
//   },
//   pinType: 'transform'
// });

// // Refresh on Locomotive updates
// locoScroll.on('scroll', ScrollTrigger.update);
// ScrollTrigger.addEventListener('refresh', () => locoScroll.update());

// ============================================
// PRO PATTERN: Hybrid Normal + Smooth Scroll
// ============================================
function setupScrollStrategy() {
  // Check if user prefers reduced motion
  const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  if (prefersReducedMotion) {
    // Use native scroll (no smoothing)
    console.log("Using native scroll (accessibility preference)");
    
    // Standard ScrollTrigger (no proxy needed)
    setupScrollTriggers();
  } else {
    // Performance check for mobile
    const isMobile = window.innerWidth < 768;

    if (isMobile) {
      // Lenis with reduced smoothness on mobile
      const mobileScroll = new Lenis({
        duration: 0.8,
        smoothTouch: false,
      });

      const mobileProxy = { y: 0 };
      
      ScrollTrigger.scrollerProxy(document.body, {
        scrollTop(value) {
          return arguments.length ? null : mobileProxy.y;
        },
        getBoundingClientRect() {
          return { top: 0, left: 0, width: window.innerWidth, height: window.innerHeight };
        }
      });

      mobileScroll.on('scroll', ({ scroll }) => {
        mobileProxy.y = scroll;
        ScrollTrigger.update();
      });

      requestAnimationFrame(function raf(time) {
        mobileScroll.raf(time);
        requestAnimationFrame(raf);
      });
    } else {
      // Full smooth scroll on desktop
      const desktopScroll = new Lenis({
        duration: 1.5,
      });

      const desktopProxy = { y: 0 };
      
      ScrollTrigger.scrollerProxy(document.body, {
        scrollTop(value) {
          return arguments.length ? null : desktopProxy.y;
        },
        getBoundingClientRect() {
          return { top: 0, left: 0, width: window.innerWidth, height: window.innerHeight };
        }
      });

      desktopScroll.on('scroll', ({ scroll }) => {
        desktopProxy.y = scroll;
        ScrollTrigger.update();
      });

      requestAnimationFrame(function raf(time) {
        desktopScroll.raf(time);
        requestAnimationFrame(raf);
      });
    }

    setupScrollTriggers(); // Setup triggers after proxy
  }
}

function setupScrollTriggers() {
  // All ScrollTrigger animations here
  // They'll work with or without smooth scroll
  gsap.utils.toArray('.fade-in').forEach(el => {
    gsap.from(el, {
      y: 40,
      opacity: 0,
      scrollTrigger: {
        trigger: el,
        start: 'top 85%',
        toggleActions: 'play none none reverse'
      }
    });
  });
}

// Initialize
setupScrollStrategy();

// ============================================
// SCROLLTRIGGER REFRESH ON SMOOTH SCROLL RECALCULATION
// ============================================
// Call this after any DOM changes that affect layout
function handleLayoutChange() {
  // Update the smooth scroller's internal calculations
  if (window.lenis) {
    window.lenis.resize();
  }
  
  // Refresh all ScrollTrigger positions
  ScrollTrigger.refresh();
}

window.addEventListener('resize', handleLayoutChange);
// After dynamic content loads:
// fetchData().then(() => handleLayoutChange());
```

---
