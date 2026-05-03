#### Core Concept
The **MotionPathPlugin** moves elements along a predefined SVG `<path>` or a custom array of coordinate points. Instead of animating X/Y separately (which creates only linear diagonals), you get smooth, curvilinear motion — perfect for orbits, complex trajectories, and organic movement patterns.

#### Deep Dive

**Path Sources**:
- SVG `<path id="myPath">` referenced by `path: "#myPath"`
- Inline SVG path data string: `path: "M 0 0 Q 100 50, 200 100"`
- Array of coordinates: `path: [{x:0, y:0}, {x:100, y:50}, {x:200, y:0}]`
- CSS selector for existing SVG: `path: ".curve-path"`

**Core Configuration**:
- `align`: The element's `path` (rotation follows path tangent) or `"self"` (maintains original rotation)
- `alignOrigin`: Point on element that follows the path `[0.5, 0.5]` = center, `[0, 0]` = top-left
- `autoRotate`: Boolean or number. `true` aligns rotation to path. `-90` offsets alignment by -90deg.
- `start`/`end`: Progress along path (0 to 1). `start: 0.2, end: 0.8` only traverses 60% of path.
- `immediateRender`: Prevent the initial jump to path position (for `from()` tweens).
- `resolution`: Number of points to sample on the path (higher = smoother but more computation).

**Path Command Support**:
- M, L, C, Q, S, T, A (Arc), Z — all standard SVG path commands
- Supports relative (lowercase) and absolute (uppercase) coordinates
- Can parse `<path>`, `<polyline>`, `<polygon>`, `<circle>`, `<ellipse>`, `<rect>`, `<line>`

#### ⚠️ Common Pitfalls & Pro Tips

1. **The path must be in the DOM for SVG selector reference**: If creating paths dynamically, inject into DOM before animating. Alternatively, use inline path string or coordinate array.

2. **`alignOrigin` on non-square elements**: A rectangle following a path with `alignOrigin: [0,0]` will have its top-left corner trace the path. Center alignment `[0.5, 0.5]` is usually more natural.

3. **`autoRotate` on symmetric shapes**: A circle with `autoRotate` shows no visual change since it's rotationally symmetric. Verify on asymmetric shapes.

4. **Performance on complex paths**: A path with 1000+ coordinate points will recalculate extensively. Use `resolution` to reduce sample points or simplify the SVG path.

5. **Nested SVG coordinate spaces**: If animating an element inside an SVG with `viewBox`, MotionPath uses the SVG's coordinate system, not the page's. Ensure consistent coordinate spaces.

#### Example Code

```javascript
gsap.registerPlugin(MotionPathPlugin);

// ============================================
// BASIC MOTION PATH (SVG Reference)
// ============================================
// Assumes: <path id="orbitPath" d="M 100 200 C 300 50, 500 350, 600 200" fill="none" stroke="gray"/>
gsap.to(".orbiter", {
  duration: 3,
  repeat: -1,
  yoyo: true,
  motionPath: {
    path: "#orbitPath",       // SVG path selector
    align: "#orbitPath",      // Align rotation to path
    alignOrigin: [0.5, 0.5],  // Center of element follows path
    autoRotate: true,         // Rotate element along path tangent
    start: 0,
    end: 1,
    // ease: "linear"         // Uncomment for constant speed
  },
  ease: "power1.inOut"
});

// ============================================
// INLINE PATH DATA STRING
// ============================================
gsap.to(".inline-follower", {
  duration: 4,
  motionPath: {
    // M: move to, C: cubic bezier
    path: "M 100 300 C 250 100, 400 500, 600 200 S 800 100, 900 300",
    align: "self",            // Don't rotate along path
    alignOrigin: [0.5, 0.5],
    autoRotate: false,
  },
  repeat: -1,
  ease: "none"               // Linear for constant speed
});

// ============================================
// COORDINATE ARRAY PATH
// ============================================
const customPath = [
  { x: 0, y: 0 },           // Start
  { x: 100, y: -50 },       // Up-right
  { x: 200, y: 50 },        // Down-right
  { x: 300, y: -100 },      // Higher arc
  { x: 400, y: 0 },         // Return to baseline
  { x: 500, y: -150 },      // Peak
  { x: 600, y: 0 },         // Finish
];

gsap.to(".array-follower", {
  duration: 5,
  motionPath: {
    path: customPath,
    curviness: 1.5,          // Smoothness of interpolation between points
    align: "path",
    autoRotate: 90,          // Offset rotation by 90 degrees
  },
  repeat: -1,
  ease: "power2.inOut"
});

// ============================================
// CIRCULAR ORBIT (Using SVG arc or circle)
// ============================================
// Instead of an SVG path, use path shorthand for circle
gsap.to(".planet", {
  duration: 2,
  repeat: -1,
  motionPath: {
    path: [
      { x: 200, y: 200 },   // Simulated circle with curviness
      { x: 200, y: 0 },
      { x: 0, y: -200 },
      { x: -200, y: 0 },
      { x: -200, y: 200 },
      { x: 0, y: 400 }
    ],
    curviness: 1.25,         // High curviness = approximates circle
    align: "path",
    autoRotate: false,
  },
  ease: "none"
});

// ============================================
// PARTIAL PATH TRAVERSAL
// ============================================
gsap.to(".partial-traveler", {
  duration: 1.5,
  motionPath: {
    path: "#longPath",
    align: "#longPath",
    autoRotate: true,
    start: 0.3,               // Start 30% into the path
    end: 0.7,                 // End at 70% (only travels 40% of path)
  },
  ease: "power3.inOut"
});

// ============================================
// MULTIPLE ELEMENTS ON SAME PATH (Offset timing)
// ============================================
const followers = gsap.utils.toArray(".path-follower");
followers.forEach((follower, i) => {
  // Each follower starts at different position AND time
  const startOffset = i * 0.15; // 0, 0.15, 0.3, 0.45...

  gsap.to(follower, {
    duration: 3,
    repeat: -1,
    ease: "none",
    motionPath: {
      path: "#sharedPath",
      align: "#sharedPath",
      autoRotate: true,
      start: startOffset,
      end: 1 + startOffset, // Wrap around (will be clamped to 1)
      immediateRender: true
    },
    delay: (1 - startOffset) * 3 // Staggered start times
  });
});

// ============================================
// MOTIONPATH WITH FROMTO (animate between path positions)
// ============================================
gsap.fromTo(".segment-traveler",
  {
    motionPath: {
      path: "#fullPath",
      start: 0.1,            // From 10% along path
      end: 0.1,
      align: "#fullPath",
      autoRotate: true
    }
  },
  {
    motionPath: {
      path: "#fullPath",
      start: 0.1,            // Still from 10%...
      end: 0.8,              // ...to 80%
      align: "#fullPath",
      autoRotate: true
    },
    duration: 2.5,
    ease: "power2.inOut"
  }
);

// ============================================
// PRO PATTERN: Bezier Curve Generator
// ============================================
function createCurvedPath(startX, startY, endX, endY, controlPoints) {
  // controlPoints: array of {x, y} control handles
  let path = `M ${startX} ${startY} `;
  
  controlPoints.forEach((cp, i) => {
    if (i === 0) {
      // First segment: cubic bezier from start to first control
      path += `C ${cp.x} ${cp.y}, `;
    } else if (i === controlPoints.length - 1) {
      // Last segment: cubic bezier to end point
      path += `${cp.x} ${cp.y}, ${endX} ${endY}`;
    } else {
      // Middle segments: smooth curve through internal points
      path += `${cp.x} ${cp.y}, `;
    }
  });

  return path;
}

// Generate a winding path
const windingPath = createCurvedPath(
  0, 300,                    // Start
  800, 300,                  // End
  [
    { x: 200, y: 100 },     // Control point 1
    { x: 400, y: 500 },     // Control point 2
    { x: 600, y: 100 },     // Control point 3
  ]
);

gsap.to(".curved-traveler", {
  duration: 4,
  repeat: -1,
  motionPath: {
    path: windingPath,
    align: "path",
    autoRotate: true,
    curviness: 1,
  },
  ease: "power1.inOut"
});

// ============================================
// RESPONSIVE MOTIONPATH (Reacts to viewport)
// ============================================
function createResponsivePath() {
  const vw = window.innerWidth;
  const vh = window.innerHeight;

  // Path scales with viewport
  const responsivePathData = [
    { x: vw * 0.1, y: vh * 0.5 },
    { x: vw * 0.3, y: vh * 0.2 },
    { x: vw * 0.6, y: vh * 0.8 },
    { x: vw * 0.9, y: vh * 0.3 },
    { x: vw * 0.7, y: vh * 0.6 },
  ];

  return responsivePathData;
}

let pathAnimation;

function setupPathAnimation() {
  // Kill existing animation
  if (pathAnimation) pathAnimation.kill();

  const pathData = createResponsivePath();

  pathAnimation = gsap.to(".responsive-mover", {
    duration: 3,
    repeat: -1,
    motionPath: {
      path: pathData,
      curviness: 1.2,
      align: "self",
      autoRotate: false,
    },
    ease: "none"
  });
}

// Initial setup
setupPathAnimation();

// Recreate on resize
window.addEventListener("resize", setupPathAnimation);

// ============================================
// MOTIONPATH WITH SCROLLTRIGGER
// ============================================
gsap.to(".scroll-path-follower", {
  motionPath: {
    path: "#scrollPath",
    align: "#scrollPath",
    autoRotate: true,
    start: 0,
    end: 1,
  },
  scrollTrigger: {
    trigger: ".path-section",
    start: "top 80%",
    end: "bottom 20%",
    scrub: 1,               // Smooth scrub along path
    // markers: true,
  }
});
```

---
