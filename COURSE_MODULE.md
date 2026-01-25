# Course Module: Parametric Animation with p5.js

**Duration:** 60–90 minutes (can be split into two sessions)  
**Level:** Intermediate (assumes basic p5.js knowledge)  
**Format:** Hands-on workshop with code exploration

---

## Learning Objectives

By the end of this module, students will be able to:

1. **Explain** how the Chaikin corner-cutting algorithm transforms rough input into smooth curves
2. **Implement** a sliding-window animation model using head/tail indexing
3. **Apply** Perlin noise to create organic, continuous motion in particle systems
4. **Design** resolution-independent graphics using normalized coordinates
5. **Export** creative coding work as SVG vectors and video recordings

---

## Key Concepts

| Concept | What It Means | Why It Matters |
|---------|---------------|----------------|
| **Chaikin Subdivision** | Recursive corner-cutting to smooth polylines | Transforms gesture into design |
| **Head/Tail Animation** | Sliding window over a point array | Creates "drawing itself" effect |
| **Perlin Noise** | Continuous pseudo-random values | Organic motion, not chaos |
| **Normalized Coordinates** | Store positions as 0–1 ratios | Resolution independence |
| **Simulation Space** | Fixed reference resolution for processing | Consistent output across exports |
| **Unit Scaling** | Multiply fixed constants by a scale factor | Same proportions at any size |

---

## The Story of Chaikin

### Who Was George Chaikin?

George Chaikin was a computer scientist who, in **1974**, published a simple yet powerful algorithm for generating smooth curves from rough polygons. At a time when computers had limited processing power, Chaikin's method was elegant: instead of complex spline mathematics, just **cut the corners**.

### How It Works

Given a polyline (a series of connected straight segments):

```
Before:  ●━━━━━━━━━━●━━━━━━━━━━●
         A          B          C
         (sharp corners at B)
```

Chaikin's algorithm:
1. For each segment A→B, place a new point at **25%** of the way
2. Place another new point at **75%** of the way
3. Connect these new points, discarding the original corner
4. Repeat the process on the result

```
After 1 iteration:   ●──●────●──●────●──●
After 2 iterations:  Smoother...
After 3 iterations:  Nearly a curve!
```

### Why It Matters for This Project

When you draw with a mouse, you create **imperfect, jagged input**. The Chaikin algorithm is the bridge between human gesture and smooth visual output. It's a form of **signal processing**—filtering noise while preserving intention.

This connects beautifully to the project's theme: *transformation*. The algorithm doesn't add information; it reveals the curve that was always implied by your gesture.

### A Note on "AI"

The Chaikin algorithm is **not AI**—it's a deterministic geometric process. But it demonstrates a principle that does appear in machine learning: **refinement through iteration**. Each pass of the algorithm brings the shape closer to an ideal. In that sense, it's a poetic precursor to the iterative optimization that powers modern AI systems.

---

## Teaching Plan

### Part 1: Introduction (10 min)

**Show the finished animation** running in both formats (landscape/portrait).

Discussion prompts:
- "What do you notice about how the line appears?"
- "Where does the organic, fluid feeling come from?"
- "How might you export this for use in a video?"

**Introduce the project structure:**
- Single HTML file with embedded p5.js sketch
- Control panel for parameters
- Export capabilities (SVG, video)

---

### Part 2: The Drawing Pipeline (15 min)

**Walk through the code flow** (use diagrams on whiteboard):

```
Mouse Input → pointsNorm (normalized) → Chaikin Smoothing → Cache → Animation → Render
```

**Live demo:** 
1. Draw on the canvas
2. Show `pointsNorm` in console: values between 0 and 1
3. Show `preparedCacheSim.length` growing after smoothing

**Key code to highlight:**

```javascript
// Storing normalized points (mousePressed/mouseDragged)
let nx = mouseX / width;
let ny = mouseY / height;
pointsNorm.push({ x: nx, y: ny });

// Rebuilding cache in simulation space
let pointsSim = denormalizePointsForSize(pointsNorm, sim.w, sim.h);
preparedCacheSim = chaikinReduce(pointsSim, seg_limit_sq);
```

**Discussion:** Why store as 0–1 instead of pixels?  
Answer: The same drawing works at any resolution.

---

### Part 3: The Chaikin Algorithm (20 min)

**Whiteboard exercise:** Draw a triangle manually, then apply one iteration of Chaikin by hand.

**Code exploration:** Find `chaikinReduce()` in the source:

```javascript
function chaikinReduce(pts, segLimitSq) {
    // Subdivision logic...
    // For each segment, create points at 25% and 75%
}
```

**Experiment:** 
- Change `seg_limit_sq` to a larger value → fewer subdivisions, more angular
- Change to a smaller value → more subdivisions, smoother

**Connect to the visual:** Show how the same gesture looks with different subdivision levels.

---

### Part 4: Head/Tail Animation (15 min)

**Concept introduction:**

```
Points:  [0] [1] [2] [3] [4] [5] [6] [7] [8] [9] ...
              └─────── visible ───────┘
              tail                    head
```

- `head` advances each frame (controlled by `animSpeed`)
- `tail` follows, keeping `max_show` points visible
- `tailSmooth` uses `lerp()` for gradual following

**Code walkthrough:**

```javascript
head += animSpeed;
let targetTail = max(0, head - max_show);
tailSmooth = lerp(tailSmooth, targetTail, 0.1);  // Smooth follow
let tail = floor(tailSmooth);
let reduced = preparedCacheSim.slice(tail, head);
```

**Interactive demo:**
- Pause animation (`0` key)
- Manually increment head in console
- Observe the slice changing

**Experiment:** Change the lerp factor (0.1) to 0.5 or 0.01. What happens?

---

### Part 5: Perlin Noise Splashes (15 min)

**Review Perlin noise basics:**
- Unlike `random()`, `noise()` returns continuous values
- Nearby inputs → similar outputs
- Adding a time dimension creates smooth evolution

**Code exploration:** Find `drawSplashToContext()`:

```javascript
let n = noise(x + noiseSpace, y + noiseSpace, noise_time);
let dx = map(n, 0, 1, -r_x, r_x);
let dy = map(n, 0, 1, -r_y, r_y);
let r = map(noise(...), 0, 1, r_min, r_max);
ellipse(point.x + dx, point.y + dy, r * 2, r * 2);
```

**Experiment:**
- Change `noise_delta_time` to speed up or slow down the motion
- Increase `r_x` and `r_y` for wilder movement
- Change the color of `drops1`, `drops2`, `drops3`

---

### Part 6: Export Deep Dive (10 min)

**SVG Export:**
- Uses p5.js-svg to create a true vector graphics buffer
- `createGraphics(w, h, SVG)` creates an offscreen SVG canvas
- Same `renderScene()` function draws to it
- Serialize with `XMLSerializer` and download as `.svg`

**Video Recording:**
- `canvas.captureStream(fps)` creates a video stream from the canvas
- `MediaRecorder` records the stream to chunks
- On stop, combine chunks into a Blob and download

**Key insight:** Both exports reuse the same rendering code via `renderScene(target, w, h, ...)`. This is **DRY principle** in action.

---

### Part 7: Wrap-Up & Extensions (5–10 min)

**Review the pipeline** one more time:

> Input → Normalize → Smooth (Chaikin) → Cache → Animate (head/tail) → Map to target → Render

**Discussion prompts:**
- "What other gestures could drive this system?" (audio, sensors, data)
- "How might you change the end card for a different brand?"
- "What happens if you combine multiple Chaikin-smoothed paths?"

---

## Exercises

### Exercise 1: Tail Length Experiment (Easy)
Change `max_show` from 2500 to 500, then to 5000. Observe how this affects:
- The perceived speed of the animation
- The visual density of the lines
- The "memory" of the path

### Exercise 2: Custom Color Palette (Easy)
Modify these color definitions to create your own palette:
```javascript
linesStart = color(164, 232, 207);  // Start of gradient
linesEnd = color(210, 163, 169);    // End of gradient
drops1 = color(119, 58, 239);       // Splash color 1
drops2 = color(159, 98, 255);       // Splash color 2
drops3 = color(79, 18, 199);        // Splash color 3
```

### Exercise 3: Easing Functions (Medium)
The circle transition uses `easeOutCubic`. Implement and try:
- `easeInOutQuad`
- `easeOutElastic`
- `easeOutBounce`

Find easing functions at [easings.net](https://easings.net/).

### Exercise 4: Alternative End Card (Medium)
Instead of a circle, create:
- A rectangle that grows from the center
- Multiple small circles that converge
- A shape that matches your brand

### Exercise 5: Audio-Reactive (Advanced)
Use `p5.AudioIn` to:
- Control animation speed based on volume
- Change splash size based on frequency
- Trigger the end card when audio stops

### Exercise 6: Multi-Path Composition (Advanced)
Modify the code to support multiple independent paths:
- Store an array of `pointsNorm` arrays
- Each path has its own color
- Paths can animate at different speeds

---

## Discussion Prompts

1. **Gesture vs. Algorithm**  
   "The Chaikin algorithm smooths your drawing. Is the result more 'you' or more 'algorithm'? Where does authorship lie?"

2. **Smoothing as Aesthetic Choice**  
   "What if we didn't smooth at all? What does the raw gesture communicate differently?"

3. **Resolution Independence**  
   "Why is it valuable that the same drawing works at 720p and 4K? What other creative tools do this?"

4. **Export as Distribution**  
   "This project can output SVG and video. How does the export format affect how people experience and share your work?"

5. **Iteration and Refinement**  
   "Chaikin's algorithm refines a shape through repeated passes. Where else do we see 'iterative refinement' in creative practice?"

---

## Assessment Ideas

### Formative (During Session)
- Can students predict what happens when `max_show` changes?
- Can students trace a point from mouse input to rendered output?
- Can students modify a color and see the result?

### Summative (After Session)
- Create a personalized version with a new color palette and end-card text
- Record a 10-second video in both landscape and portrait formats
- Write a short paragraph explaining the head/tail animation model

---

## Resources

- **p5.js Reference:** [p5js.org/reference](https://p5js.org/reference/)
- **Chaikin's Algorithm Explained:** [Understanding Chaikin's Algorithm](https://www.bit-101.com/blog/2021/08/chaikins-algorithm/)
- **Perlin Noise Deep Dive:** [The Nature of Code, Chapter 1](https://natureofcode.com/book/introduction/)
- **p5.js-svg Documentation:** [github.com/zenozeng/p5.js-svg](https://github.com/zenozeng/p5.js-svg)
- **Easing Functions:** [easings.net](https://easings.net/)

---

## Instructor Notes

### Common Student Struggles

1. **Normalized coordinates confusion**  
   Students may not immediately understand why we divide by width/height. Use a concrete example: "If your canvas is 1920 wide and you click at x=960, that's 0.5—exactly halfway. Now if we resize to 3840, we multiply 0.5 × 3840 = 1920, still halfway."

2. **Chaikin subdivision count**  
   The code doesn't use a fixed iteration count; it uses a distance threshold. Explain that we keep subdividing until segments are short enough.

3. **Why sim-space?**  
   This is the trickiest concept. Emphasize: "We want 1000 points in the cache whether the preview is 800px or the recording is 1920px. The cache density should be constant."

### Timing Adjustments

- **Shorter session (45 min):** Skip Part 6 (exports) and Exercise 5-6
- **Longer session (2 hours):** Add live coding to create a new end-card animation from scratch
- **Multi-day course:** Day 1 = Parts 1-4, Day 2 = Parts 5-6 + exercises

### Hardware Considerations

- Video recording works best in Chrome
- Safari users should focus on SVG export
- If students have slow machines, suggest reducing `numLines` to 5
