#### Core Concept
**FLIP** = First, Last, Invert, Play. The FLIP plugin automates seamless transitions between two states of an element, even if the states have radically different positions, sizes, rotations, or parent containers. It's the magic behind shared element transitions, expanding cards, and morphing layouts.

#### Deep Dive

**The FLIP Algorithm**:
1. **First**: Record the element's current position, size, rotation (the "starting state").
2. **Last**: Change the element to its new position/size/rotation (the "ending state") and record.
3. **Invert**: Calculate the difference and apply inverse transforms so the element APPEARS to be back in the First state (but structurally is in the Last state).
4. **Play**: Animate the inversed transforms to zero, creating a smooth transition.

**`Flip.fit()`**: Resizes/moves one element to perfectly fit another. Useful for transitioning between unrelated elements.

**`Flip.from(state, options)`**: Animates FROM a previously captured state TO the current DOM state.

**`Flip.getState(targets, props)`**: Captures the current state of elements. Store and reuse.

**Key Properties**:
- `targets`: The element(s) to transition (or selector string/element/array)
- `duration`, `ease`, `stagger`, `delay` — Standard tween props
- `absolute: true` — Use position:absolute on targets during animation (for grid/flex transitions)
- `scale: true` — Animate scale changes (default true)
- `fade: true` — Animate opacity changes
- `spin: true` — Animate rotation changes (default true)
- `simple: true` — Skip child element handling for performance
- `props: ["x", "y", "scale", "opacity"]` — Limit what properties FLIP tracks
- `nested: true` — Properly handle nested FLIP animations
- `toggleClass: "active"` — Add class after transition

#### ⚠️ Common Pitfalls & Pro Tips

1. **`absolute: true` on grid parents**: If the target is in a CSS Grid during the transition, toggling `position: absolute` briefly will remove it from the grid flow. Layout may shift. Consider wrapping in a container.

2. **Nested FLIP animations**: Changing a parent layout affects child positions. Use `nested: true` and call `Flip.from()` on children AFTER the parent transition is set up.

3. **Font rendering during FLIP**: Rapid scale changes can cause text to blur (browser re-renders at intermediate sizes). Use `will-change: transform` on text elements pre-transition.

4. **Memory leaks**: `Flip.getState()` returns a big object. Don't hold onto state objects longer than needed. Release references.

5. **`onEnter`/`onLeave` with GSAP context**: If using React, FLIP transitions should be wrapped in `gsap.context()` to properly clean up on unmount.

#### Example Code

```javascript
gsap.registerPlugin(Flip);

// ============================================
// BASIC FLIP: Expanding Card
// ============================================
// HTML: A grid of cards, clicking one "expands" it

document.querySelectorAll(".card").forEach(card => {
  card.addEventListener("click", function() {
    // FIRST: Capture current state
    const state = Flip.getState(this, {
      props: "transform,opacity,borderRadius" // What to track
    });

    // LAST: Change the DOM state
    this.classList.toggle("card--expanded");

    // INVERT + PLAY: Animate from captured state to new state
    Flip.from(state, {
      duration: 0.6,
      ease: "power3.inOut",
      scale: true,
      fade: true,
      absolute: true,  // Use position absolute during transition
      onComplete: () => {
        // Clean up any inline styles after transition
        Flip.fit(null, this); // Optional: remove FLIP styles
      }
    });
  });
});

// ============================================
// FLIP GETSTATE: Reusable state capture
// ============================================
// Capture state BEFORE any DOM changes
const preTransitionState = Flip.getState(".transitioning-elements", {
  props: "transform,opacity,backgroundColor,boxShadow"
});

// Make your DOM changes (reorder, add classes, move elements)
document.querySelector(".layout").classList.add("reordered");

// Animate from captured state to new state
Flip.from(preTransitionState, {
  duration: 0.8,
  stagger: 0.1,         // Stagger element transitions
  ease: "power2.inOut",
  absolute: true,
  scale: true,
  spin: true,
  fade: true,
  onComplete: () => {
    // Post-transition cleanup
    document.querySelectorAll(".transitioning-elements").forEach(el => {
      el.style.transform = "";
      el.style.opacity = "";
      el.style.width = "";
    });
  }
});

// ============================================
// FLIP BETWEEN CONTAINERS (Cross-parent transition)
// ============================================
// Moving an element from one parent to another
const movingElement = document.querySelector(".draggable");

// Capture state in current parent
const moveState = Flip.getState(movingElement);

// Move element to new container
document.querySelector(".new-container").appendChild(movingElement);

// Animate from old position to new position
Flip.from(moveState, {
  duration: 0.5,
  ease: "back.out(1.2)",
  absolute: true,      // Essential for cross-parent transitions
  scale: true,
  onComplete: () => {
    movingElement.style.position = ""; // Clean up
  }
});

// ============================================
// FLIP.FIT(): Making two different elements morph
// ============================================
// Useful for modal openings, image zooms, etc.
const thumbnail = document.querySelector(".thumbnail");
const fullImage = document.querySelector(".full-image");

// Make fullImage appear exactly where thumbnail was
Flip.fit(fullImage, thumbnail, {
  duration: 0,         // Instant: just position it
  scale: true,
});

// Then animate to its natural size
gsap.to(fullImage, {
  // These properties will animate from the "fitted" state
  scale: 1,
  x: 0,
  y: 0,
  duration: 0.6,
  ease: "power3.out"
});

// ============================================
// FULL FLIP.FIT() WITH ANIMATION
// ============================================
document.querySelector(".open-modal").addEventListener("click", () => {
  const modal = document.querySelector(".modal");
  const button = document.querySelector(".open-modal");

  // First: make modal fit the button
  Flip.fit(modal, button, { duration: 0, scale: true });

  // Show the modal (it's positioned over the button now)
  modal.style.visibility = "visible";

  // Now animate from button-size to full-screen modal
  Flip.from(Flip.getState(modal), {
    duration: 0.5,
    ease: "power3.inOut",
    scale: true,
    // Start from the fitted state, end at natural modal size
    onComplete: () => {
      modal.classList.add("modal-open");
    }
  });
});

// ============================================
// NESTED FLIP (Parent AND children transition)
// ============================================
const parentCard = document.querySelector(".parent-card");
const childElements = parentCard.querySelectorAll(".child");

// Capture ALL states before change
const parentState = Flip.getState(parentCard);
const childStates = Flip.getState(childElements);

// Make changes (e.g., expand the card, and children rearrange)
parentCard.classList.toggle("card-expanded");

// Animate parent first
Flip.from(parentState, {
  duration: 0.7,
  ease: "power3.out",
  absolute: true,
  nested: true,          // Tell GSAP children also transition
});

// Then separately animate children (with their own feel)
Flip.from(childStates, {
  duration: 0.5,
  stagger: 0.05,
  ease: "back.out(1.2)",
  delay: 0.15,           // Start slightly after parent
  absolute: true,
  scale: true,
});

// ============================================
// PRO PATTERN: Search/Filter Layout Transition
// ============================================
function filterItems(filterFn) {
  const container = document.querySelector(".filter-grid");
  const items = container.querySelectorAll(".filter-item");

  // Capture current state of all items
  const state = Flip.getState(items);

  // Apply filter (items that don't match get hidden)
  items.forEach(item => {
    if (filterFn(item)) {
      item.style.display = "block";
    } else {
      item.style.display = "none";
    }
  });

  // Animate the remaining items to their new positions
  Flip.from(state, {
    duration: 0.5,
    stagger: 0.05,
    ease: "power2.inOut",
    absolute: true,
    scale: true,
    fade: true,
    onEnter: (elements) => {
      // Items that APPEAR (were hidden, now shown)
      return gsap.fromTo(elements, 
        { opacity: 0, scale: 0.3 },
        { opacity: 1, scale: 1, duration: 0.4 }
      );
    },
    onLeave: (elements) => {
      // Items that DISAPPEAR (were shown, now hidden)
      return gsap.to(elements, { 
        opacity: 0, 
        scale: 0.3, 
        duration: 0.3 
      });
    }
  });
}

// Usage on search input
document.querySelector(".filter-input").addEventListener("input", (e) => {
  const searchTerm = e.target.value.toLowerCase();
  filterItems((item) => {
    return item.textContent.toLowerCase().includes(searchTerm);
  });
});

// ============================================
// FLIP WITH SCROLLTRIGGER (Layout change on scroll)
// ============================================
gsap.timeline({
  scrollTrigger: {
    trigger: ".sticky-header-trigger",
    start: "top top",
    end: "+=500",
    scrub: true,
  }
})
.add(() => {
  // Capture state before header change
  const headerState = Flip.getState(".header-logo, .header-nav");

  // Toggle compact class
  document.querySelector(".header").classList.add("header-compact");

  // Animate transition
  Flip.from(headerState, {
    duration: 0.4,
    ease: "power2.inOut",
    scale: true,
  });
});

// ============================================
// DYNAMIC LIST REORDERING (Drag-and-drop style)
// ============================================
function moveItem(fromIndex, toIndex) {
  const list = document.querySelector(".sortable-list");
  const items = list.querySelectorAll(".list-item");

  // Capture all item states
  const state = Flip.getState(items);

  // Reorder DOM
  if (toIndex >= items.length) {
    list.appendChild(items[fromIndex]);
  } else if (fromIndex < toIndex) {
    list.insertBefore(items[fromIndex], items[toIndex].nextSibling);
  } else {
    list.insertBefore(items[fromIndex], items[toIndex]);
  }

  // Animate the transition
  Flip.from(state, {
    duration: 0.4,
    stagger: {
      each: 0.02,
      from: Math.min(fromIndex, toIndex), // Stagger outward from change
    },
    ease: "power2.inOut",
    absolute: true,
    scale: false,         // Optional: don't scale, just move
    onComplete: () => {
      // Clean up styles
      items.forEach(item => {
        item.style.transform = "";
      });
    }
  });
}
```

---

