# School of Tomorrow's AI Vignette Generator

A gesture-driven animation generator for creating intro/outro vignettes with p5.js.

![p5.js](https://img.shields.io/badge/p5.js-1.11.0-ED225D?logo=p5.js)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## TL;DR

```bash
# 1. Run locally
python3 -m http.server 8080
# Open http://localhost:8080

# 2. Draw a path with mouse click+drag

# 3. Choose format: Landscape 16:9 or Portrait 9:16

# 4. Export SVG (press S) or Record Video (panel controls)
```

**Formats:** 1920×1080 (YouTube) or 1080×1920 (Reels/TikTok)  
**Outputs:** `.svg` vector or `.webm` video up to 4K

---

## Quickstart

### Requirements
- A modern web browser (Chrome recommended)
- A local web server (no build step required)

### Run Locally

**Option 1: Python**
```bash
cd SOTA-vignette-generator
python3 -m http.server 8080
```

**Option 2: Node.js**
```bash
npx serve .
# or: npx http-server .
```

**Option 3: VS Code**  
Install "Live Server" extension → Click "Go Live"

Then open `http://localhost:8080` in your browser.

---

## Controls

### Mouse
| Action | Result |
|--------|--------|
| Click + Drag | Draw path on canvas |

### Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `+` / `=` | Increase animation speed |
| `-` / `_` | Decrease animation speed |
| `0` | Pause / Resume |
| `C` | Clear canvas |
| `R` | Reset to default drawing |
| `S` | Export SVG |

### Panel Controls
| Control | Description |
|---------|-------------|
| **Format** | Landscape 16:9 or Portrait 9:16 |
| **Animation Speed** | 0.1× to 10× playback |
| **Max Visible Points** | Tail length (higher = longer trail) |
| **Number of Lines** | Layered line copies (1–30) |
| **Title Text** | Customize Line 1 and Line 2 |
| **Debug** | Show stats overlay |

---

## Exports

### SVG Export
1. Pause or let animation reach desired frame
2. Select export size (Match Format / Preset / Custom)
3. Toggle **Include background** and **Include splashes**
4. Click **Download SVG** or press `S`

**Output:** `vignette_FORMAT_TIMESTAMP.svg`

### Video Recording
1. Set **Recording Resolution** (e.g., 1920×1080)
2. Choose **FPS** (30 or 60) and **Duration**
3. Select **Bitrate** (Medium = 10 Mbps recommended)
4. Enable **Record from start** to capture full animation
5. Click **Start Recording**
6. File auto-downloads when complete

**Output:** `vignette_WxH_FPS_DURATION_BITRATE.webm`

---

## Common Workflows

### 1. YouTube Intro (16:9)
```
Format:              Landscape 16:9
Recording Resolution: 1920×1080 (Full HD)
FPS:                 30
Duration:            10–15 seconds
Bitrate:             Medium (10 Mbps)
Record from start:   ✓ Enabled
```

### 2. Instagram Reels / TikTok (9:16)
```
Format:              Portrait 9:16
Recording Resolution: 1080×1920 (Full HD Portrait)
FPS:                 30
Duration:            10 seconds
Bitrate:             Medium (10 Mbps)
Record from start:   ✓ Enabled
```

### 3. Clean SVG for Print/Web
```
Export Size:         1920×1080 or 1080×1920
Include background:  ✓ (or ✗ for transparent)
Include splashes:    ✗ Disabled (smaller file, cleaner look)
```

### 4. High-Quality Recording (Archive/Master)
```
Recording Resolution: 1920×1080 or higher
FPS:                 30
Duration:            15–20 seconds
Bitrate:             High (20 Mbps) or Ultra (35 Mbps)
Use devicePixelRatio: ✓ Enabled (sharper on HiDPI)
Record from start:   ✓ Enabled
```

---

## Project Structure

```
SOTA-vignette-generator/
├── index.html          # Complete application (single-file prototype)
├── favicon.svg         # Browser tab icon
├── README.md           # This file
├── LICENSE             # MIT License
├── .gitignore          # Git ignore rules
├── COURSE_MODULE.md    # Teaching guide (60–90 min workshop)
└── PROJECT_STORY.md    # Social media / press copy
```

**Why single-file?**  
This is an educational prototype designed for easy sharing and modification. All HTML, CSS, and JavaScript live in `index.html` for simplicity. No build tools or dependencies required.

---

## Parameter Reference

| Parameter | Variable | Default | Description |
|-----------|----------|---------|-------------|
| Animation Speed | `animSpeed` | 1 | Playback multiplier (0 = paused) |
| Max Visible Points | `max_show` | 2500 | Tail length in points |
| Number of Lines | `numLines` | 10 | Layered path copies |
| Segment Limit | `seg_limit_sq` | 100 | Chaikin subdivision threshold |
| Typewriter Speed | `typewriterSpeed` | 80 | ms per character |
| Circle Duration | `circleTransitionDuration` | 1500 | Circle animation (ms) |

---

## How It Works

### Pipeline Overview

```
Mouse Input
    ↓
Normalize to 0–1 coordinates (pointsNorm)
    ↓
Denormalize to simulation space (1920×1080 or 1080×1920)
    ↓
Chaikin corner-cutting smoothing → preparedCacheSim
    ↓
Animation: slice(tail, head) with sliding window
    ↓
Map to render target size
    ↓
Render: lines + splashes + circle + text
```

### Head/Tail Animation

The animation uses a **sliding window** over the smoothed path:

```
Points:  [0]──[100]──[200]──[300]──[400]──[500]──[600]
               └────── visible (max_show) ──────┘
               tail                             head
```

- `head` advances each frame by `animSpeed`
- `tail` follows with `lerp()` smoothing for fluid motion
- `max_show` controls how many points remain visible

### Chaikin Corner-Cutting

Transforms rough gestures into smooth curves by recursively cutting corners at 25%/75% positions:

```
Before:  ●━━━━━━━●━━━━━━━●  (sharp corners)
After:   ●──●──●──●──●──●   (smooth curve)
```

### Perlin Noise Splashes

Organic particle motion using continuous noise:
- Position offset: `noise(x, y, time)` → dx, dy
- Radius variation: `noise(...)` → size
- Creates flowing, non-random motion

### Simulation Space

All processing happens at a fixed resolution (1920×1080 or 1080×1920) regardless of preview size. This ensures:
- Consistent tail length in preview vs. recording
- Same point density across export sizes
- No visual surprises when switching resolutions

---

## Known Issues / Limitations

| Issue | Details | Workaround |
|-------|---------|------------|
| **Safari video recording** | MediaRecorder has limited WebM support | Use SVG export or screen recording software |
| **Large SVG with splashes** | Splashes generate many elements, creating large files | Disable "Include splashes" for cleaner exports |
| **4K recording performance** | May cause frame drops on older hardware | Use 1080p or reduce `numLines` |
| **Long recordings** | Browser memory may grow with 30s+ recordings | Keep recordings under 20s; export multiple clips |
| **WebM compatibility** | Some video editors don't import WebM natively | Convert to MP4 using FFmpeg or Handbrake |

### Converting WebM to MP4
```bash
ffmpeg -i input.webm -c:v libx264 -crf 18 -preset slow output.mp4
```

---

## Troubleshooting

### "CORS error" or blank canvas when opening index.html directly

**Problem:** Opening `index.html` by double-clicking (file:// protocol) blocks loading of external libraries.

**Solution:** Use a local web server:
```bash
python3 -m http.server 8080
# Then open http://localhost:8080
```

### SVG export button disabled / "SVG library not loaded"

**Problem:** The p5.js-svg library failed to load from CDN.

**Solutions:**
1. Check your internet connection
2. Ensure you're using a local server (not file://)
3. Check browser console for specific errors
4. Try a different browser (Chrome recommended)

### Video recording produces empty or corrupt file

**Problem:** MediaRecorder may fail silently on some browsers.

**Solutions:**
1. Use Chrome (best MediaRecorder support)
2. Try a shorter duration (10s instead of 30s)
3. Reduce recording resolution (1080p instead of 4K)
4. Check browser console for errors

### Animation is slow or stuttering

**Problem:** Too many elements being rendered.

**Solutions:**
1. Reduce **Number of Lines** to 5–8
2. Lower **Max Visible Points** to 1000–1500
3. Close other browser tabs
4. Disable browser extensions

### Exported video looks different from preview

**Problem:** Preview size differs from recording resolution.

**Solution:** This is expected behavior. The recording uses the full resolution while the preview fits your screen. The proportions and styling should match—only sharpness differs.

---

## Browser Compatibility

| Feature | Chrome | Firefox | Safari | Edge |
|---------|:------:|:-------:|:------:|:----:|
| Core animation | ✅ | ✅ | ✅ | ✅ |
| SVG export | ✅ | ✅ | ✅ | ✅ |
| Video recording | ✅ | ✅ | ⚠️ | ✅ |

**Recommended:** Chrome for full functionality

---

## Performance Tips

- **Reduce `numLines`** (5–10) for smoother playback on slower devices
- **Lower `max_show`** if animation stutters with complex paths
- **Disable splashes** in SVG export to reduce file size
- **Use 1080p** instead of 4K for recording if you experience frame drops
- Cache rebuilds only when points change (`dirty = true`)

---

## Background: The Story

Every hand-drawn line carries intention, but also imperfection. This project transforms rough gestures into smooth, flowing curves using the **Chaikin corner-cutting algorithm**—a technique developed by George Chaikin in 1974 that recursively "cuts corners" to create organic curves from jagged polylines.

The result is a living ribbon that breathes with Perlin noise, layered with translucent splashes that dance along the path. When the animation completes, a soft circle blooms at the center and the title emerges letter by letter: *School of Tomorrow's AI*.

It's a meditation on transformation: from noise to signal, from gesture to design, from raw input to refined output.

### Visual Sequence
1. **Path reveal** — Smoothed lines trace your drawn gesture
2. **Splash particles** — Perlin noise-driven dots bloom along the curve
3. **Circle transition** — Translucent pink circle moves to center and grows
4. **Title reveal** — Typewriter effect displays the title text

---

## Customization

### Colors
```javascript
linesStart = color(164, 232, 207);  // Gradient start
linesEnd = color(210, 163, 169);    // Gradient end
drops1 = color(119, 58, 239);       // Splash color 1
drops2 = color(159, 98, 255);       // Splash color 2
drops3 = color(79, 18, 199);        // Splash color 3
```

### Circle
```javascript
// In renderScene(), find the circle section:
const circleColor = color(234, 125, 255, opacity * 255);  // Pink
```

### Font
```javascript
textFont('Space Grotesk');  // Change to any Google Font
```

### Easing
Replace `easeOutCubic()` with other functions from [easings.net](https://easings.net/).

---

## Credits

- **[p5.js](https://p5js.org/)** — Creative coding library
- **[p5.js-svg](https://github.com/zenozeng/p5.js-svg)** — SVG renderer by Zeno Zeng
- **George Chaikin** — Corner-cutting algorithm (1974)
- **Processing Foundation** — Creative coding education

---

## License

**MIT License** — Use, modify, and share freely.

Attribution appreciated:
> Built with School of Tomorrow's AI Vignette Generator

---

## Related Files

- **[COURSE_MODULE.md](./COURSE_MODULE.md)** — 60–90 minute teaching guide
- **[PROJECT_STORY.md](./PROJECT_STORY.md)** — Social media copy and press text
