#### Core Concept
The `useGSAP()` hook (available in GSAP 3.12+) is a first-class React hook that wraps `gsap.context()` into an ergonomic, React-idiomatic API. It automatically handles cleanup, provides a scoped selector, and respects React's Strict Mode by properly reverting on double-mount. It's the recommended way to use GSAP in React now.

#### Deep Dive

**Hook Signature**: `useGSAP(callback, { scope, dependencies, revertOnUpdate })`

- **`callback`**: Function receiving `(context, contextSafe)` where:
  - `context`: The GSAP context instance (for `.selector()`, `.add()`, etc.)
  - `contextSafe`: A wrapper function that ensures ANY GSAP code runs within the safe context
- **`scope`**: Optional React ref or DOM element. If omitted, `scope` defaults to the component's root.
- **`dependencies`**: Optional array (like useEffect deps). Re-runs callback when deps change.
- **`revertOnUpdate`**: If `true` (default), kills all animations from PREVIOUS render before creating new ones.

**`contextSafe` Function**: The superpower. Wrap any event handler or async callback with `contextSafe()` to ensure animations created in those callbacks are automatically cleaned up when the component unmounts. Without it, hover handlers, click handlers, and interval-based animations leak.

**Benefits over Manual `gsap.context()`**:
- Automatic cleanup (no forgetting `return () => ctx.revert()`)
- React Strict Mode compatible (handles double-invocation)
- Cleaner syntax, aligns with React mental model
- Built-in dependency tracking
- TypeScript support out of the box

#### ⚠️ Common Pitfalls & Pro Tips

1. **Forgetting `contextSafe` on event handlers**: If you create a GSAP tween in an `onClick` handler without wrapping it in `contextSafe`, that tween WILL persist after unmount.

2. **Dependencies array behaves like useEffect**: Including `scopeRef.current` in dependencies will cause infinite loops. Use the `scope` option instead of putting it in deps.

3. **`revertOnUpdate: false` for incremental animations**: If you're adding to a persistent timeline on each render, set `revertOnUpdate: false` and manually manage the timeline's lifecycle.

4. **Multiple `useGSAP` calls in one component**: Perfectly fine and encouraged! Each can have its own scope, dependencies, and cleanup. They don't conflict.

5. **ScrollTrigger with `useGSAP`**: It integrates perfectly. Kill/refresh is handled by the cleanup. Just ensure you `registerPlugin(ScrollTrigger)`.

#### Example Code

```jsx
// ============================================
// BASIC useGSAP (Simple Entrance Animation)
// ============================================
import { useRef } from 'react';
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';

function HeroSection() {
  const heroRef = useRef(null);

  useGSAP(() => {
    // Scope automatically defaults to heroRef's current value
    // (because it's the containing ref)
    
    gsap.from(".hero-title", {
      y: 80,
      opacity: 0,
      duration: 1,
      ease: "power4.out"
    });

    gsap.from(".hero-subtitle", {
      y: 40,
      opacity: 0,
      duration: 0.8,
      delay: 0.3,
      ease: "power3.out"
    });

    gsap.from(".hero-cta", {
      scale: 0.5,
      opacity: 0,
      duration: 0.6,
      delay: 0.7,
      ease: "back.out(1.7)"
    });

    // Cleanup is automatic!
  }, { scope: heroRef }); // Optional: explicit scope

  return (
    <section ref={heroRef} className="hero">
      <h1 className="hero-title">Build Something Great</h1>
      <p className="hero-subtitle">With GSAP and React</p>
      <button className="hero-cta">Get Started</button>
    </section>
  );
}
```

```jsx
// ============================================
// useGSAP WITH DEPENDENCIES (Reactive Animation)
// ============================================
import { useState, useRef } from 'react';
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';

function CounterAnimation({ count }) {
  const counterRef = useRef(null);

  // Animation re-runs whenever `count` changes
  useGSAP(() => {
    gsap.from(counterRef.current, {
      scale: 1.5,
      duration: 0.3,
      ease: "back.out(1.7)",
      // Will kill previous animation and start new one
    });
  }, { 
    scope: counterRef,
    dependencies: [count],           // React to count changes
    revertOnUpdate: true             // Kill previous animation first
  });

  return <span ref={counterRef}>{count}</span>;
}
```

```jsx
// ============================================
// useGSAP WITH contextSafe (Event Handlers)
// ============================================
import { useRef } from 'react';
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';

function InteractiveCard() {
  const cardRef = useRef(null);
  const timelineRef = useRef(null);

  // Destructure contextSafe from the callback parameter
  const { contextSafe } = useGSAP(() => {
    
    // Create a hover timeline ONCE
    timelineRef.current = gsap.timeline({ paused: true });
    timelineRef.current
      .to(cardRef.current, {
        scale: 1.05,
        boxShadow: "0 10px 30px rgba(0,0,0,0.2)",
        duration: 0.3,
        ease: "power2.out"
      })
      .to(".card-content", {
        y: -5,
        duration: 0.2,
        ease: "power1.out"
      }, 0); // Start at same time

  }, { scope: cardRef });

  // Wrap event handlers with contextSafe
  // These tweens will auto-cleanup on unmount!
  const handleMouseEnter = contextSafe(() => {
    timelineRef.current?.play();
  });

  const handleMouseLeave = contextSafe(() => {
    timelineRef.current?.reverse();
  });

  const handleClick = contextSafe(() => {
    // Even complex click animations are safe
    gsap.timeline()
      .to(cardRef.current, {
        scale: 0.95,
        duration: 0.1
      })
      .to(cardRef.current, {
        scale: 1.02,
        duration: 0.2,
        ease: "power2.out"
      })
      .to(cardRef.current, {
        scale: 1,
        duration: 0.15,
        ease: "power1.inOut"
      });
  });

  return (
    <div
      ref={cardRef}
      className="interactive-card"
      onMouseEnter={handleMouseEnter}
      onMouseLeave={handleMouseLeave}
      onClick={handleClick}
    >
      <div className="card-content">Click or hover me!</div>
    </div>
  );
}
```

```jsx
// ============================================
// useGSAP + ScrollTrigger Integration
// ============================================
import { useRef } from 'react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';

gsap.registerPlugin(ScrollTrigger);

function ScrollDrivenSection() {
  const sectionRef = useRef(null);

  useGSAP(() => {
    const ctx = gsap.context(() => {
      
      // Animated timeline with pinning
      const pinTL = gsap.timeline({
        scrollTrigger: {
          trigger: ".scroll-pin",
          start: "top top",
          end: "+=1500",
          pin: true,
          scrub: 2,
        }
      });

      pinTL
        .from(".step-1", { x: -100, opacity: 0 })
        .from(".step-2", { y: 100, opacity: 0 })
        .from(".step-3", { x: 100, opacity: 0 })
        .to(".step-1", { opacity: 0.3 })
        .to(".step-2", { opacity: 0.3 });

      // Parallax elements
      gsap.to(".parallax-slow", {
        y: "-15%",
        ease: "none",
        scrollTrigger: {
          trigger: ".parallax-container",
          start: "top bottom",
          end: "bottom top",
          scrub: true,
        }
      });

      gsap.to(".parallax-fast", {
        y: "-30%",
        ease: "none",
        scrollTrigger: {
          trigger: ".parallax-container",
          start: "top bottom",
          end: "bottom top",
          scrub: true,
        }
      });

    }, sectionRef);

    return () => ctx.revert(); // Still needed for ScrollTrigger cleanup
  }, { scope: sectionRef });

  return (
    <section ref={sectionRef}>
      <div className="scroll-pin">
        <div className="step-1">Step 1</div>
        <div className="step-2">Step 2</div>
        <div className="step-3">Step 3</div>
      </div>
      <div className="parallax-container">
        <div className="parallax-slow">Slow</div>
        <div className="parallax-fast">Fast</div>
      </div>
    </section>
  );
}
```

```jsx
// ============================================
// PRO PATTERN: Custom useGSAP Composition
// ============================================
import { useRef } from 'react';
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';

// Extract animation logic into custom hooks
function useEntranceAnimation(ref) {
  useGSAP(() => {
    gsap.from(ref.current, {
      y: 60,
      opacity: 0,
      duration: 0.8,
      ease: "power3.out",
      scrollTrigger: {
        trigger: ref.current,
        start: "top 80%",
        toggleActions: "play none none reverse"
      }
    });
  }, { scope: ref, dependencies: [ref] });
}

function useHoverScale(ref, scale = 1.05, duration = 0.3) {
  useGSAP(() => {
    const tl = gsap.timeline({ paused: true });
    tl.to(ref.current, { scale, duration, ease: "power2.out" });

    const handleEnter = () => tl.play();
    const handleLeave = () => tl.reverse();

    ref.current?.addEventListener("mouseenter", handleEnter);
    ref.current?.addEventListener("mouseleave", handleLeave);

    return () => {
      // Use contextSafe-like pattern for cleanup
      ref.current?.removeEventListener("mouseenter", handleEnter);
      ref.current?.removeEventListener("mouseleave", handleLeave);
    };
  }, { scope: ref });
}

// Compose them in a component
function FeatureCard({ title, description }) {
  const cardRef = useRef(null);
  
  useEntranceAnimation(cardRef);
  useHoverScale(cardRef, 1.03, 0.25);

  return (
    <div ref={cardRef} className="feature-card">
      <h3>{title}</h3>
      <p>{description}</p>
    </div>
  );
}
```

```jsx
// ============================================
// ADVANCED: Multiple useGSAP calls with different scopes
// ============================================
import { useRef } from 'react';
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

function ComplexPage() {
  const headerRef = useRef(null);
  const heroRef = useRef(null);
  const galleryRef = useRef(null);

  // Header animation (always visible)
  useGSAP(() => {
    gsap.from(".nav-item", {
      y: -20,
      opacity: 0,
      stagger: 0.1,
      duration: 0.5,
      ease: "power2.out"
    });
  }, { scope: headerRef });

  // Hero section with ScrollTrigger
  useGSAP(() => {
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: heroRef.current,
        start: "top top",
        end: "bottom center",
        scrub: 1,
      }
    });

    tl.to(".hero-bg", { scale: 1.2 })
      .to(".hero-text", { y: 100, opacity: 0 }, "-=0.3");
  }, { scope: heroRef });

  // Gallery stagger on scroll
  useGSAP(() => {
    gsap.from(".gallery-item", {
      y: 80,
      opacity: 0,
      stagger: 0.15,
      duration: 0.7,
      scrollTrigger: {
        trigger: galleryRef.current,
        start: "top 75%",
        toggleActions: "play none none reverse"
      }
    });
  }, { scope: galleryRef });

  return (
    <>
      <header ref={headerRef}>
        <nav>
          <span className="nav-item">Home</span>
          <span className="nav-item">About</span>
          <span className="nav-item">Contact</span>
        </nav>
      </header>

      <section ref={heroRef} className="hero">
        <div className="hero-bg" />
        <div className="hero-text">Welcome</div>
      </section>

      <section ref={galleryRef} className="gallery">
        {[...Array(6)].map((_, i) => (
          <div key={i} className="gallery-item">Item {i + 1}</div>
        ))}
      </section>
    </>
  );
}
```

---

 