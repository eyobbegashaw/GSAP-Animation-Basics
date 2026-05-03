#### Core Concept
**TextPlugin** replaces text content character-by-character inside an element, animating the replacement. **ScrambleTextPlugin** (Club GSAP) takes this further—randomly jumbling characters before resolving to the final text, creating a "decoding" or "hack screen" effect. Both are fundamental to cinematic text reveals.

#### Deep Dive

**TextPlugin** (`gsap/TextPlugin`):
- Replaces the text content of a DOM element smoothly
- Works on `textContent` (not innerHTML — strips tags)
- Automatically preserves spaces (use `{replacement: " "}` for custom)
- Simple API: `text: "new text value"` or `text: { value: "new", newClass: "highlight" }`

**ScrambleTextPlugin** (Club GSAP benefit):
- Randomly scrambles characters, then resolves to target text
- Configurable character sets: `"upperCase"`, `"lowerCase"`, `"upperAndLowerCase"`, `"numbers"`, `"symbols"`, custom string
- `scrambleText: { text: "Final Text", chars: "customSet", revealDelay: 0.5, speed: 0.3 }`
- Tween length controls how fast it resolves; `revealDelay` controls when it STARTS resolving
- New text can be revealed left-to-right, right-to-left, or randomly
- Can scramble individual words, characters, or maintain positions

**Comparison**:
- **TextPlugin**: Clean replacement, no jumble. Best for simple swaps.
- **ScrambleText**: Visual "encryption/decryption" effect. Best for sci-fi, tech, cinematic reveals.
- **SplitText**: (Club GSAP) SPLITS text into chars/words/lines for individual animation. Complements both.

#### ⚠️ Common Pitfalls & Pro Tips

1. **TextPlugin doesn't preserve HTML tags**: Animating `<span>styled</span>` text will strip the span and show the raw tag string. Use SplitText for complex markups.

2. **ScrambleText character set mismatch**: If your target text has characters NOT in the scramble set, they'll appear suddenly at the end (no smooth scramble). Ensure your charset covers all final characters.

3. **newClass property**: You can apply a CSS class mid-reveal — powerful for color changes or style shifts once the text "settles."

4. **Performance with long text**: Scrambling 100+ characters each frame is expensive. For long paragraphs, animate line-by-line or use SplitText groups.

5. **Club GSAP licensing**: ScrambleTextPlugin requires a Club GSAP membership. TextPlugin is free and bundled with GSAP.

#### Example Code

```javascript
// Register plugins
gsap.registerPlugin(TextPlugin);
// ScrambleTextPlugin is auto-detected when loaded (Club GSAP)

// ============================================
// TEXTPLUGIN BASICS
// ============================================

// Simple text replacement
gsap.to(".text-replace", {
  text: "This text has changed!",
  duration: 1,
  ease: "none" // Linear for consistent character swapping
});

// With custom replacement character
gsap.to(".text-replace-custom", {
  text: {
    value: "New content appears",
    // Preserve spaces (default replaces spaces too)
    // Use empty string if you want characters to erase 
    // before the new ones type in
  },
  duration: 0.8,
  ease: "power2.out"
});

// Sequential text changes (like a headline rotator)
const textTimeline = gsap.timeline({ repeat: -1, repeatDelay: 1 });

textTimeline
  .to(".rotating-headline", {
    text: "Design",
    duration: 0.4,
    ease: "steps(6)" // Discrete steps for typewriter feel
  })
  .to(".rotating-headline", {
    text: "", // Clear it
    duration: 0.15,
  })
  .to(".rotating-headline", {
    text: "Develop",
    duration: 0.5,
    ease: "steps(7)"
  })
  .to(".rotating-headline", {
    text: "",
    duration: 0.15,
  })
  .to(".rotating-headline", {
    text: "Deliver",
    duration: 0.5,
    ease: "steps(7)"
  });

// ============================================
// TEXT PLUGIN WITH STAGGER (Multiple elements)
// ============================================
const words = ["Create", "Animate", "Innovate"];
const textElements = gsap.utils.toArray(".word-placeholder");

textElements.forEach((el, i) => {
  gsap.to(el, {
    text: words[i],
    duration: 0.6,
    delay: i * 0.3,
    ease: "power2.out"
  });
});

// ============================================
// SCRAMBLETEXT PLUGIN BASICS (Club GSAP)
// ============================================

// Basic scramble reveal
gsap.to(".scramble-basic", {
  duration: 1.5,
  scrambleText: {
    text: "Decrypted Message",
    chars: "upperCase",      // Only uppercase letters scramble
    revealDelay: 0.5,        // Wait 0.5s before starting to resolve
    speed: 0.6,              // Speed of character resolution
    tweenLength: false       // Use tween's duration (default: true)
  }
});

// Custom character set scramble
gsap.to(".scramble-custom", {
  duration: 2,
  scrambleText: {
    text: "SystemHack.exe",
    chars: "XO01#@%&",      // Only these characters in scramble
    revealDelay: 1,
    speed: 0.2,
    newClass: "text-success" // Class added when each char resolves
  }
});

// Word-by-word scramble (revealing whole words at once)
gsap.to(".scramble-words", {
  duration: 2,
  scrambleText: {
    text: "This sentence will unscramble word by word",
    chars: "lowerCase",
    revealDelay: 0.8,
    speed: 0.3,
    delimiter: " ",          // Treat spaces as word boundaries
    newClass: "revealed",    // CSS class on reveal
  }
});

// Right-to-left reveal (for RTL languages or visual interest)
gsap.to(".scramble-rtl", {
  duration: 1.8,
  scrambleText: {
    text: "Unscrambling backwards",
    chars: "upperCase",
    revealDelay: 0.3,
    speed: 0.5,
    // Animation starting from right side
    rightToLeft: true
  }
});

// ============================================
// ADVANCED SCRAMBLE TEXT PATTERNS
// ============================================

// Countdown / Number scramble
gsap.to(".countdown-number", {
  duration: 3,
  scrambleText: {
    text: "{final}",
    chars: "0123456789",
    revealDelay: 2,
    speed: 0.25,
    onUpdate: function() {
      // As text resolves, update related visuals
      const currentValue = this.targets()[0].textContent;
      if (currentValue.length > 2) {
        gsap.to(".number-circle", { scale: 1.05, duration: 0.1 });
      }
    }
  }
});

// Sequential scramble within a timeline
const revealTimeline = gsap.timeline({ defaults: { ease: "power2.out" } });

revealTimeline
  .to(".line-1", {
    duration: 1,
    scrambleText: { 
      text: "Welcome to the future", 
      chars: "upperCase",
      speed: 0.5
    }
  })
  .to(".line-2", {
    duration: 1.2,
    scrambleText: { 
      text: "Where code meets creativity", 
      chars: "upperAndLowerCase",
      speed: 0.4,
      revealDelay: 0.3
    }
  }, "-=0.3") // Slight overlap
  .to(".line-3", {
    duration: 0.8,
    scrambleText: { 
      text: "© 2026", 
      chars: "XO",
      speed: 0.6,
      revealDelay: 0.2
    }
  });

// ============================================
// PRO PATTERN: Typewriter + Scramble Combo
// ============================================
function createAdvancedTextReveal(element, text, options = {}) {
  const {
    scrambleChars = "upperCase",
    scrambleDuration = 0.5,
    typewriterDuration = 1.5,
    revealDelay = 0,
  } = options;

  const tl = gsap.timeline({
    delay: revealDelay,
  });

  // Phase 1: Scramble quickly (noise phase)
  tl.to(element, {
    duration: scrambleDuration,
    scrambleText: {
      text: text,
      chars: scrambleChars,
      speed: 0.2,
      tweenLength: false
    }
  })
  // Phase 2: Slowly resolve to final text
  .to(element, {
    duration: typewriterDuration,
    scrambleText: {
      text: text,
      chars: scrambleChars,
      speed: 0.1,
      revealDelay: 0,
      newClass: "revealed",
      tweenLength: true // Tween length controls speed
    }
  });

  return tl;
}

// Usage
const cipherElement = document.querySelector(".cipher-text");
createAdvancedTextReveal(cipherElement, "Decrypting secret message...", {
  scrambleChars: "#@$%&*!0123456789ABCDEF",
  scrambleDuration: 0.3,
  typewriterDuration: 2,
  revealDelay: 0.5
});

// ============================================
// TEXT + SPLITTEXT INTEGRATION (Club GSAP)
// ============================================
// When you need character-level transform control AND scramble

// Split the text first
// const split = new SplitText(".split-scramble", { type: "chars" });
// const chars = split.chars;

// // Then scramble the text while animating characters
// gsap.fromTo(chars, 
//   {
//     opacity: 0,
//     y: 20,
//     rotation: 10
//   },
//   {
//     opacity: 1,
//     y: 0,
//     rotation: 0,
//     stagger: 0.05,
//     duration: 0.4,
//     ease: "back.out(1.7)",
//     onStart: () => {
//       // Simultaneously scramble the text content
//       gsap.to(".split-scramble", {
//         duration: 1.5,
//         scrambleText: {
//           text: "Transformed",
//           chars: "upperCase",
//           speed: 0.3
//         }
//       });
//     }
//   }
// );
```

---
