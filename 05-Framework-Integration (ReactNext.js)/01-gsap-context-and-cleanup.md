#### Core Concept
In React, components mount and unmount unpredictably. Traditional GSAP code creates memory leaks because animations keep references to unmounted DOM elements. **`gsap.context()`** solves this by creating a scoped animation environment where all GSAP instances (tweens, timelines, ScrollTriggers) are automatically tracked and can be killed with a single `revert()` call — ensuring zero memory leaks.

#### Deep Dive

**What `gsap.context()` Does**:
- Creates a context (like a box) where all GSAP animations created inside are registered
- Returns a cleanup function that kills EVERYTHING in that context — tweens, timelines, ScrollTriggers, Draggable, SplitText, etc.
- Automatically passes a safe selector function `(selector) => context.selector(selector)` that scopes queries to the context's DOM element
- Can accept a DOM element reference as its scope: `gsap.context(callback, scopeElement)`
- On `revert()`, it not only kills animations but also reverts inline styles to their pre-animation state

**The Pattern**:
```javascript
useEffect(() => {
  const ctx = gsap.context(() => {
    // All GSAP code here
  }, componentRef); // scope

  return () => ctx.revert(); // Cleanup on unmount
}, []);
```

**Scoped Selectors**: `ctx.selector(".my-class")` ensures you only target elements within the component, avoiding cross-component interference.

**`ctx.add()`**: Manually add a tween, timeline, or callback to the context for tracking.

#### ⚠️ Common Pitfalls & Pro Tips

1. **Always return the cleanup function**: Forgetting `return () => ctx.revert()` is the #1 cause of GSAP memory leaks in React.

2. **Don't use `gsap.context()` OUTSIDE useEffect**: Creating contexts in render functions creates infinite loops. Context belongs in `useEffect` or lifecycle methods.

3. **Selectors can be null initially**: If animating refs, ensure `componentRef.current` is not null before passing to `gsap.context()`.

4. **ScrollTrigger cleanup is CRITICAL**: ScrollTriggers persist globally. Without `revert()`, they accumulate on every hot reload and route change, causing phantom animations.

5. **`ctx.revert()` also removes inline styles**: Great for resetting, but if you need CSS styles to persist after unmount, apply those styles via CSS classes, not GSAP inline styles.

#### Example Code

```jsx
// ============================================
// REACT BASIC: gsap.context() Pattern
// ============================================
import { useEffect, useRef } from 'react';
import gsap from 'gsap';

function AnimatedComponent() {
  const sectionRef = useRef(null);

  useEffect(() => {
    // Create context scoped to this component
    const ctx = gsap.context(() => {
      // Safe selector: only targets elements inside sectionRef
      // If sectionRef is null, selector returns empty array (no errors)
      
      // Entrance animation
      gsap.from(".title", {
        y: 50,
        opacity: 0,
        duration: 0.8,
        delay: 0.2,
        ease: "power3.out"
      });

      gsap.from(".description", {
        y: 30,
        opacity: 0,
        duration: 0.6,
        delay: 0.5,
        ease: "power2.out"
      });

      // Hover animations for cards
      const cards = gsap.utils.toArray(".card");
      cards.forEach(card => {
        const tl = gsap.timeline({ paused: true });
        tl.to(card, { scale: 1.05, duration: 0.3, ease: "power2.out" });
        
        card.addEventListener("mouseenter", () => tl.play());
        card.addEventListener("mouseleave", () => tl.reverse());
      });

    }, sectionRef); // Scope to component root element

    // CRITICAL: Cleanup function
    return () => ctx.revert();

  }, []); // Empty dependency = run once on mount

  return (
    <section ref={sectionRef} className="animated-section">
      <h1 className="title">Welcome</h1>
      <p className="description">This is animated safely</p>
      <div className="card">Card 1</div>
      <div className="card">Card 2</div>
      <div className="card">Card 3</div>
    </section>
  );
}
```

```jsx
// ============================================
// REACT: ScrollTrigger with Cleanup
// ============================================
import { useEffect, useRef } from 'react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

function ScrollAnimatedSection() {
  const sectionRef = useRef(null);

  useEffect(() => {
    const ctx = gsap.context(() => {
      
      // Scroll-triggered timeline
      const tl = gsap.timeline({
        scrollTrigger: {
          trigger: sectionRef.current,
          start: "top 80%",
          end: "bottom 20%",
          scrub: 1,
          // markers: true,
        }
      });

      tl.from(".scroll-title", { y: 100, opacity: 0 })
        .from(".scroll-card", { 
          x: -200, 
          opacity: 0,
          stagger: 0.2 
        }, "-=0.2")
        .from(".scroll-image", { 
          scale: 0.8, 
          opacity: 0 
        }, "<");

      // Parallax effect
      gsap.to(".parallax-bg", {
        y: "-20%",
        ease: "none",
        scrollTrigger: {
          trigger: ".parallax-wrapper",
          start: "top bottom",
          end: "bottom top",
          scrub: true,
        }
      });

    }, sectionRef);

    // On unmount: kill all ScrollTriggers and tweens
    return () => ctx.revert();

  }, []);

  return (
    <section ref={sectionRef}>
      <div className="parallax-wrapper">
        <div className="parallax-bg" />
      </div>
      <h2 className="scroll-title">Scroll Animated</h2>
      <div className="scroll-card">Card 1</div>
      <div className="scroll-card">Card 2</div>
      <div className="scroll-card">Card 3</div>
      <img className="scroll-image" src="/image.jpg" alt="" />
    </section>
  );
}
```

```jsx
// ============================================
// REACT: Dynamic Content + FLIP Animation
// ============================================
import { useState, useEffect, useRef } from 'react';
import gsap from 'gsap';
import { Flip } from 'gsap/Flip';

gsap.registerPlugin(Flip);

function ExpandableGrid() {
  const [expandedId, setExpandedId] = useState(null);
  const gridRef = useRef(null);

  const handleCardClick = (id) => {
    const ctx = gsap.context(() => {
      const cards = gsap.utils.toArray('.grid-card');
      
      // Capture state before any change
      const state = Flip.getState(cards);

      // Update state (triggers React re-render)
      setExpandedId(prev => prev === id ? null : id);

      // Since React hasn't re-rendered yet, we need to
      // wait for the DOM update before flipping
    }, gridRef);

    // Cleanup when component unmounts
    // (This won't run immediately — it's for unmount)
    // We manually revert after animation completes
  };

  useEffect(() => {
    // This effect runs AFTER React re-renders with new state
    if (!gridRef.current) return;

    const ctx = gsap.context(() => {
      const cards = gsap.utils.toArray('.grid-card');
      
      // Since state changed, capture that we want to flip
      // But we need the PREVIOUS state...
      // Better approach: store state before setState
      
      // Simplified: just animate any card that became expanded
      if (expandedId) {
        const expandedCard = cards.find(card => 
          card.dataset.id === String(expandedId)
        );
        
        if (expandedCard) {
          gsap.from(expandedCard, {
            scale: 0.9,
            opacity: 0.5,
            duration: 0.5,
            ease: "power2.out"
          });
        }
      }
    }, gridRef);

    return () => ctx.revert();
  }, [expandedId]);

  return (
    <div ref={gridRef} className="grid-container">
      {items.map(item => (
        <div
          key={item.id}
          data-id={item.id}
          className={`grid-card ${expandedId === item.id ? 'expanded' : ''}`}
          onClick={() => handleCardClick(item.id)}
        >
          {item.content}
        </div>
      ))}
    </div>
  );
}
```

```jsx
// ============================================
// REACT: MANUAL CONTEXT ADDITION
// ============================================
import { useEffect, useRef } from 'react';
import gsap from 'gsap';

function ComplexAnimation() {
  const containerRef = useRef(null);
  
  useEffect(() => {
    const ctx = gsap.context(() => {
      // Standard animations get auto-added
    }, containerRef);

    // Manually add external tweens/timelines
    const externalTween = gsap.to(".external-element", {
      x: 100,
      duration: 1
    });
    
    // Track it for cleanup
    ctx.add(externalTween);

    // Track a callback-based cleanup
    const resizeHandler = () => {
      // Some resize logic
    };
    window.addEventListener("resize", resizeHandler);
    
    ctx.add(() => {
      // Cleanup function for non-GSAP resources
      window.removeEventListener("resize", resizeHandler);
    });

    return () => ctx.revert();
  }, []);

  return <div ref={containerRef}>...</div>;
}
```

---
