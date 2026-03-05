---
name: svg-diagram
description: >-
  Generate technical diagrams, architecture visuals, and layered illustrations as SVG files
  with an HTML export wrapper for PNG download. Use when the user asks to create a diagram,
  architecture chart, flowchart, layered visualization, system overview, or any visual that
  uses boxes, arrows, labels, and colors.
  TRIGGER when: user says "create a diagram", "draw", "visualize", "architecture diagram",
  "layered diagram", "flowchart", "make me an image of", or describes a visual layout.
  DO NOT TRIGGER when: user wants a presentation, a chart from data (use a
  charting library), or photo editing.
---

# SVG Diagram Generator

Generate clean, professional technical diagrams as SVG files with an accompanying HTML page for PNG export.

## Core Philosophy

1. **Pure SVG** -- No external dependencies. One `.svg` file that opens anywhere.
2. **Crisp text** -- SVG text renders sharply at any resolution, unlike rasterized alternatives.
3. **Export-ready** -- Every diagram ships with an HTML wrapper for 1x and 2x PNG download.
4. **Minimal footprint** -- Two files: `<name>.svg` and `<name>-export.html`. No build tools.

---

## Phase 1: Understand the Diagram

Before generating, determine what the user needs. If the request is vague, ask via AskUserQuestion.

If the user provides a screenshot or detailed description, skip questions and generate directly. Use your judgment for layout, colors, and structure -- there are no prescribed patterns. Design each diagram to fit its content.

---

## Phase 2: Generate the SVG

### File naming

Use a descriptive kebab-case name based on the content (e.g. `api-architecture.svg`, `deploy-pipeline.svg`).

### SVG structure

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}" viewBox="0 0 {W} {H}">
  <style>
    text { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif; }
    /* Define reusable text classes */
  </style>

  <!-- Background -->
  <rect width="{W}" height="{H}" fill="#ffffff"/>

  <!-- Diagram elements -->
</svg>
```

---

## What NOT To Do (Critical Anti-Patterns)

These are the SVG pitfalls that ruin diagrams. Avoid all of them.

### DO NOT let text overlap

SVG has no auto-layout. Text elements placed at similar Y coordinates WILL collide. You must manually guarantee separation.

- **Never place a title and description at the same Y coordinate.** Use a two-row layout: title near the top of a box, description below it with a clear gap (40-50px minimum between baselines).
- **Never assume text will fit.** Before placing any text, estimate its pixel width: `character_count x per_char_width`. Common per-char widths:
  - 13px font: ~7px/char
  - 14px font: ~8.5px/char
  - 16-17px font: ~9.5px/char
  - 28px font: ~16px/char
- **Never skip the bounding-box check.** For every text element, confirm: `estimated_text_width < container_width - (2 x padding)`. If it doesn't fit, shrink the font, abbreviate the text, or widen the container.

### DO NOT let text overflow its container

SVG text does NOT clip or wrap automatically. If text is wider than its box, it bleeds out visually.

- **Never put long text in narrow containers without checking width.** A word like "Building" (8 chars) at 14px font is ~68px wide. A 65px sidebar cannot hold it.
- **Never write lines longer than ~85 characters without wrapping.** Split into multiple `<tspan>` elements with `dy` spacing between them.
- **Container width rule:** `width >= (longest_word_in_chars x per_char_width) + 16px padding` at minimum.

### DO NOT use external SVG references in the HTML export

Browsers block `canvas.toDataURL()` on images loaded from `file://` URLs due to CORS. This silently breaks PNG export.

- **Never use `<img src="diagram.svg">` in the export HTML.** Always inline the full SVG markup directly into the HTML page.

### DO NOT forget text rendering basics

- **Never omit `text-anchor` on centered text.** Without `text-anchor="middle"`, text positioned at a center X coordinate will be left-aligned from that point, not centered on it.
- **Never use a single `<text>` element for multi-line content.** SVG `<text>` is single-line only. Use `<tspan>` elements with `dy` attributes for line breaks.
- **Never mix alignment modes at the same Y position.** A left-aligned title and a center-aligned description at the same Y will collide when the description is long.

### DO NOT create cluttered or cramped diagrams

- **Never pack elements edge-to-edge.** Leave minimum 30px padding from the canvas edge and 20-30px gaps between major elements.
- **Never use too many colors.** Pick a small, purposeful palette. Colors should encode meaning (grouping, status, hierarchy), not decorate.
- **Never make the canvas too small for the content.** Size the canvas to fit the content with breathing room. It's better to be slightly too large than to cram things.

### DO NOT forget accessibility and structure

- **Never create a diagram without `<title>` and `<desc>` elements** inside the SVG root. Screen readers need them.
- **Never use color as the only differentiator.** Pair color with labels, borders, or patterns so the diagram is readable without color.

### DO NOT over-engineer the SVG

- **Never add drop shadows, gradients, or filters unless they serve a purpose.** They add complexity and slow rendering. Use flat design by default.
- **Never define unused `<defs>`.** Only add markers, filters, or patterns you actually reference.

---

## Phase 3: Generate the HTML Export Wrapper

For every SVG, create a companion `<name>-export.html` file. The SVG MUST be inlined directly (see anti-pattern above).

```html
<!DOCTYPE html>
<html>
<head>
  <title>{Diagram Title}</title>
  <style>
    body { margin: 40px; font-family: sans-serif; background: #f5f5f5; }
    button { padding: 12px 24px; font-size: 16px; cursor: pointer; background: #333; color: #fff; border: none; border-radius: 6px; margin-bottom: 20px; margin-right: 8px; }
    button:hover { background: #555; }
    .preview { max-width: 100%; border: 1px solid #ddd; border-radius: 8px; overflow: hidden; background: #fff; }
    .preview svg { display: block; width: 100%; height: auto; }
    canvas { display: none; }
  </style>
</head>
<body>
  <button onclick="downloadPNG(1)">Download as PNG</button>
  <button onclick="downloadPNG(2)">Download as PNG (2x)</button>
  <br><br>
  <div class="preview" id="preview">
    <!-- INLINE THE FULL SVG MARKUP HERE (not an <img> tag) -->
    <svg xmlns="http://www.w3.org/2000/svg" width="..." height="..." viewBox="...">
      ...
    </svg>
  </div>
  <canvas id="canvas"></canvas>

  <script>
    function downloadPNG(scale) {
      const svgEl = document.querySelector('#preview svg');
      const svgData = new XMLSerializer().serializeToString(svgEl);
      const blob = new Blob([svgData], { type: 'image/svg+xml;charset=utf-8' });
      const url = URL.createObjectURL(blob);

      const canvas = document.getElementById('canvas');
      const ctx = canvas.getContext('2d');
      const img = new Image();

      img.onload = function() {
        canvas.width = img.naturalWidth * scale;
        canvas.height = img.naturalHeight * scale;
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

        const link = document.createElement('a');
        link.download = '{name}.png';
        link.href = canvas.toDataURL('image/png');
        link.click();

        URL.revokeObjectURL(url);
      };

      img.src = url;
    }
  </script>
</body>
</html>
```

Replace `{name}` with the actual diagram name.

---

## Phase 4: Deliver

1. **Write both files** to the user's working directory
2. **Open the HTML file** in the browser: `open <name>-export.html`
3. **Summarize** what was created and offer to adjust
