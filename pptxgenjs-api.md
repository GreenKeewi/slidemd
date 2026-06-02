# PptxGenJS API Reference

This file is loaded by the slides skill when writing the build script.

**Install:** `npm install -g pptxgenjs`
**Run:** `node build.js`
**Output:** `.pptx` file

---

## Boilerplate Setup

```javascript
const PptxGenJS = require("pptxgenjs");
const pptx = new PptxGenJS();

// Slide dimensions (default 10" x 7.5" widescreen)
pptx.layout = "LAYOUT_WIDE"; // 13.33" x 7.5"

// Default font
pptx.theme = { headFontFace: "Arial Black", bodyFontFace: "Calibri" };
```

---

## Adding Slides

```javascript
const slide = pptx.addSlide();
```

---

## Text

```javascript
// Simple text
slide.addText("Hello World", {
  x: 0.5, y: 0.5, w: 9, h: 1,
  fontSize: 36,
  bold: true,
  color: "1E2761",
  align: "left",
  fontFace: "Arial Black",
});

// Multi-run text (mixed styles in one box)
slide.addText([
  { text: "Bold intro ", options: { bold: true, fontSize: 18 } },
  { text: "then regular text continues.", options: { fontSize: 18 } },
], { x: 0.5, y: 2, w: 9, h: 1 });

// Bullet list
slide.addText([
  { text: "First point", options: { bullet: true } },
  { text: "Second point", options: { bullet: true } },
  { text: "Third point", options: { bullet: true } },
], {
  x: 0.5, y: 2, w: 5.5, h: 3,
  fontSize: 16,
  color: "333333",
  lineSpacingMultiple: 1.4,
});

// Centered large stat
slide.addText("87%", {
  x: 6, y: 2, w: 3, h: 2,
  fontSize: 72,
  bold: true,
  color: "F96167",
  align: "center",
  valign: "middle",
});

// Caption / label text
slide.addText("Source: user-provided data", {
  x: 0.5, y: 6.8, w: 9, h: 0.4,
  fontSize: 10,
  color: "999999",
  italic: true,
});
```

---

## Shapes & Backgrounds

```javascript
// Full-slide dark background
slide.addShape(pptx.shapes.RECTANGLE, {
  x: 0, y: 0, w: "100%", h: "100%",
  fill: { color: "1E2761" },
  line: { type: "none" },
});

// Accent card / content block
slide.addShape(pptx.shapes.ROUNDED_RECTANGLE, {
  x: 0.4, y: 1.8, w: 5.5, h: 3.5,
  fill: { color: "F5F5F5" },
  line: { color: "CADCFC", width: 1 },
  rectRadius: 0.1,
});

// Thick left border accent (motif)
slide.addShape(pptx.shapes.RECTANGLE, {
  x: 0.4, y: 1.5, w: 0.08, h: 3.5,
  fill: { color: "F96167" },
  line: { type: "none" },
});

// Colored circle for icon background
slide.addShape(pptx.shapes.ELLIPSE, {
  x: 1, y: 2, w: 0.7, h: 0.7,
  fill: { color: "028090" },
  line: { type: "none" },
});

// Divider line
slide.addShape(pptx.shapes.LINE, {
  x: 0.5, y: 3.8, w: 9.3, h: 0,
  line: { color: "DDDDDD", width: 1 },
});
```

---

## Images

```javascript
// Local image file
slide.addImage({
  path: "/home/claude/slides-output/images/slide-2.png",
  x: 5.5, y: 1.2, w: 4.5, h: 4.5,
});

// Image with rounded corners effect (use shape clip)
slide.addImage({
  path: "/home/claude/slides-output/images/slide-3.png",
  x: 5.5, y: 1.2, w: 4.5, h: 4.5,
  rounding: true,
});

// Full-bleed background image
slide.addImage({
  path: "/home/claude/slides-output/images/hero.png",
  x: 0, y: 0, w: "100%", h: "100%",
});

// Semi-transparent overlay on top of image
slide.addShape(pptx.shapes.RECTANGLE, {
  x: 0, y: 0, w: "100%", h: "100%",
  fill: { color: "000000", transparency: 45 },
  line: { type: "none" },
});
```

---

## Charts

Always use real, user-provided data. Never invent numbers.

```javascript
// Bar chart
slide.addChart(pptx.charts.BAR, [
  {
    name: "Category",
    labels: ["Q1", "Q2", "Q3", "Q4"],
    values: [42, 58, 71, 95],
  },
], {
  x: 0.5, y: 1.5, w: 9, h: 4.5,
  barDir: "col",                   // "col" = vertical, "bar" = horizontal
  chartColors: ["028090"],
  showLegend: false,
  showTitle: false,
  valAxisLabelFontSize: 11,
  catAxisLabelFontSize: 11,
  dataLabelFontSize: 11,
  showValue: true,
});

// Line chart
slide.addChart(pptx.charts.LINE, [
  {
    name: "Revenue",
    labels: ["2020", "2021", "2022", "2023", "2024"],
    values: [12000, 18500, 24000, 31000, 45000],
  },
], {
  x: 0.5, y: 1.5, w: 8.5, h: 4.5,
  lineSize: 3,
  chartColors: ["F96167"],
  showLegend: true,
  legendPos: "b",
  showValue: false,
});

// Pie chart
slide.addChart(pptx.charts.PIE, [
  {
    name: "Breakdown",
    labels: ["Segment A", "Segment B", "Segment C"],
    values: [45, 30, 25],
  },
], {
  x: 1, y: 1.5, w: 8, h: 5,
  chartColors: ["028090", "F96167", "F9E795"],
  showLegend: true,
  legendPos: "r",
  showPercent: true,
  dataLabelFontSize: 13,
});

// Donut chart
slide.addChart(pptx.charts.DOUGHNUT, [
  {
    name: "Share",
    labels: ["Used", "Remaining"],
    values: [67, 33],
  },
], {
  x: 2, y: 1.5, w: 6, h: 5,
  chartColors: ["1E2761", "CADCFC"],
  holeSize: 55,
  showLegend: false,
  dataLabelFontSize: 14,
});
```

---

## Speaker Notes

```javascript
// Add notes to a slide
slide.addNotes("Key point: This data represents Q4 results.\n\nTransition: Now let's look at what drove this growth.");
```

---

## Slide Background Color

```javascript
slide.background = { color: "1E2761" };   // Dark background
slide.background = { color: "FFFFFF" };   // White
slide.background = { color: "F5F5F5" };   // Off-white / light gray
```

---

## Complete Example: Title Slide

```javascript
const titleSlide = pptx.addSlide();
titleSlide.background = { color: "1E2761" };

// Background image with overlay (optional)
titleSlide.addImage({
  path: "/home/claude/slides-output/images/hero.png",
  x: 0, y: 0, w: "100%", h: "100%",
});
titleSlide.addShape(pptx.shapes.RECTANGLE, {
  x: 0, y: 0, w: "100%", h: "100%",
  fill: { color: "1E2761", transparency: 35 },
  line: { type: "none" },
});

// Title
titleSlide.addText("Presentation Title", {
  x: 1, y: 2, w: 11, h: 1.8,
  fontSize: 48,
  bold: true,
  color: "FFFFFF",
  align: "center",
  fontFace: "Arial Black",
});

// Subtitle
titleSlide.addText("Subtitle or Tagline Here", {
  x: 1, y: 3.9, w: 11, h: 0.8,
  fontSize: 20,
  color: "CADCFC",
  align: "center",
  fontFace: "Calibri",
});

titleSlide.addNotes("Welcome the audience. Briefly introduce yourself and the topic.");
```

---

## Complete Example: Two-Column Content Slide

```javascript
const slide2 = pptx.addSlide();
slide2.background = { color: "FFFFFF" };

// Slide title
slide2.addText("Section Title", {
  x: 0.5, y: 0.3, w: 12, h: 0.8,
  fontSize: 36,
  bold: true,
  color: "1E2761",
  fontFace: "Arial Black",
});

// Left border accent
slide2.addShape(pptx.shapes.RECTANGLE, {
  x: 0.5, y: 1.3, w: 0.07, h: 4.5,
  fill: { color: "028090" },
  line: { type: "none" },
});

// Left column — bullet points
slide2.addText([
  { text: "Key point number one", options: { bullet: true, fontSize: 15, color: "333333" } },
  { text: "Key point number two with more detail", options: { bullet: true, fontSize: 15, color: "333333" } },
  { text: "Key point number three", options: { bullet: true, fontSize: 15, color: "333333" } },
], {
  x: 0.75, y: 1.4, w: 5.5, h: 4.5,
  lineSpacingMultiple: 1.5,
  paraSpaceAfter: 8,
});

// Right column — image
slide2.addImage({
  path: "/home/claude/slides-output/images/slide-2.png",
  x: 7, y: 1.2, w: 5.5, h: 4.8,
  rounding: true,
});

slide2.addNotes("Expand on each bullet point here. Don't just read the slide.");
```

---

## Complete Example: Stat Callout Slide

```javascript
const statSlide = pptx.addSlide();
statSlide.background = { color: "F5F5F5" };

statSlide.addText("By the Numbers", {
  x: 0.5, y: 0.3, w: 12, h: 0.7,
  fontSize: 36, bold: true, color: "1E2761", fontFace: "Arial Black",
});

// 3 stat cards
const stats = [
  { value: "94%", label: "User Satisfaction", x: 0.5 },
  { value: "2.4M", label: "Active Users", x: 4.7 },
  { value: "$18B", label: "Market Size", x: 8.9 },
];

stats.forEach(stat => {
  // Card background
  statSlide.addShape(pptx.shapes.ROUNDED_RECTANGLE, {
    x: stat.x, y: 1.5, w: 3.7, h: 3.8,
    fill: { color: "FFFFFF" },
    line: { color: "CADCFC", width: 1.5 },
    shadow: { type: "outer", color: "000000", blur: 6, offset: 2, angle: 45, opacity: 0.1 },
    rectRadius: 0.12,
  });
  // Big number
  statSlide.addText(stat.value, {
    x: stat.x, y: 1.9, w: 3.7, h: 1.8,
    fontSize: 60, bold: true, color: "028090", align: "center", fontFace: "Arial Black",
  });
  // Label
  statSlide.addText(stat.label, {
    x: stat.x, y: 3.7, w: 3.7, h: 0.8,
    fontSize: 14, color: "666666", align: "center",
  });
});
```

---

## Save & Output

```javascript
// Save to outputs directory
await pptx.writeFile({ fileName: "/mnt/user-data/outputs/presentation.pptx" });
console.log("✅ Presentation saved.");
```

---

## Common Mistakes to Avoid

- `w` and `h` are in inches, not pixels — `w: 9` means 9 inches
- Coordinates `x: 0, y: 0` = top-left corner
- `color` values are hex WITHOUT `#` — `"1E2761"` not `"#1E2761"`
- `transparency` is 0–100 (0 = opaque, 100 = invisible)
- Always `await pptx.writeFile()` — it's async
- Image paths must be absolute or resolvable from the script's working directory
- `bullet: true` goes in the per-run `options`, not the top-level text options
