---
name: slides
description: >
  Use this skill any time the user wants to create a beautiful, detailed, informative slide presentation (.pptx).
  Triggers on: "make me a presentation", "build a slide deck", "create slides for", "I need a deck on", "presentation for my class", "pitch deck", "slideshow", "make slides about", "school presentation", "startup pitch", "business presentation", "can you make slides", "slides for", or any request to turn content/notes/research into a visual presentation.
  Always use this skill — do not try to build slides without it. This skill handles the full workflow: upfront design interview, outline approval, Gemini image generation, chart generation, QA, and speaker notes export. It is the single source of truth for creating high-quality, designed, non-hallucinated presentations.
---

# Slides Skill

A full-workflow skill for building **premium, designed, informative .pptx presentations** — for school projects, business pitches, startup decks, and everything in between.

**This skill has two reference files. Read them when needed:**
- [`references/design-system.md`](references/design-system.md) — Color palettes, typography rules, layout patterns, anti-AI-slop checklist
- [`references/pptxgenjs-api.md`](references/pptxgenjs-api.md) — Full PptxGenJS code patterns, charts, image embedding, speaker notes

---

## Phase 1 — The Design Interview

**Before writing a single line of code, run the full interview.** Do NOT skip questions or infer answers. Every answer shapes the output.

Ask ALL of the following. You may group them into a flowing conversation — don't fire them as a numbered list — but cover every item:

### Block A — Content & Purpose
1. **What is this presentation about?** (Topic, subject, thesis)
2. **What is the goal?** (Inform, persuade, pitch, teach, present findings, tell a story?)
3. **Who is the audience?** (Teacher, investors, classmates, clients, general public?)
4. **What do you already know / have?** (Notes, research, bullet points, data, URLs, an outline?) — Ask them to paste it in. More input = better output.
5. **Are there specific facts, stats, or claims you want included?** List them. The skill will ONLY use information the user provides or facts Claude already knows with certainty — it will never fabricate data.

### Block B — Structure
6. **How many slides roughly?** (If unsure, suggest a range based on topic complexity.)
7. **Should there be an intro slide? Agenda? Conclusion? Call to action?**
8. **Any specific sections or slide topics you know you want?**

### Block C — Design & Tone
9. **What tone?** (Formal & professional / Academic & clean / Bold & energetic / Casual & friendly / Creative & expressive — or describe your own)
10. **Do you have brand colors or a color scheme in mind?** (Hex codes, or describe a vibe — e.g., "dark navy and gold", "earthy greens", "bright and playful")
    - If no colors: offer 3 curated palette options from the design system and let the user pick. Read [`references/design-system.md`](references/design-system.md) for palettes.
11. **Any font preferences?** (If none, Claude will choose a pairing that fits the tone.)
12. **What image style?** (Realistic photo / Flat illustration / Abstract / Minimalist / None — describe a vibe if you want)

### Block D — Features
13. **Do you want speaker notes?** (Yes → exported as a separate `speaker-notes.md` file in addition to embedded in .pptx)
14. **Any data/stats that need charts or graphs?** (Bar, line, pie, donut, etc. — paste the raw numbers)
15. **Any specific images you want?** (Describe them, or say "Claude's choice" to let the skill generate them via Gemini)

---

## Phase 2 — Design Brief Summary

After the interview, print a **Design Brief** for the user to approve before building. Format it like this:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DESIGN BRIEF
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Topic:        [topic]
Goal:         [goal]
Audience:     [audience]
Slide count:  [N slides]
Tone:         [tone]
Colors:       [primary hex] / [secondary hex] / [accent hex]
Fonts:        [header font] + [body font]
Image style:  [style]
Charts:       [list or "none"]
Speaker notes:[yes/no]

Content source: [summary of what the user provided]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Ask: **"Does this look right? Any changes before I build the outline?"**

---

## Phase 3 — Slide Outline Approval

Generate a slide-by-slide outline and show it to the user **before writing any code**. Format:

```
Slide 1 — [Title Slide]
  Layout: Full-bleed hero image, centered title + subtitle
  Content: [Title], [Subtitle/tagline]
  Image: [Gemini prompt or "none"]

Slide 2 — [Agenda / Overview]
  Layout: Icon row (3-4 items)
  Content: [list of sections]
  Image: none / icons

Slide 3 — [Section header or content slide title]
  Layout: Two-column (text left, image right)
  Content: [bullet points or key claim]
  Image: [Gemini prompt]
  Chart: [type + data if applicable]
...
```

Ask: **"Here's the outline — approve it or tell me what to change before I start building."**

Do NOT proceed to code until the user explicitly approves.

---

## Phase 4 — Image Generation with Gemini CLI

For every slide that needs an image, run Gemini CLI to generate it **before building the deck**.

```bash
# Check Gemini CLI is available
gemini --version 2>/dev/null || echo "Gemini CLI not found"

# Generate an image for a slide
gemini generate-image \
  --prompt "[your detailed image prompt]" \
  --output /home/claude/slides-output/images/slide-N.png \
  --aspect-ratio 16:9
```

**Writing good Gemini prompts:**
- Be specific: style, lighting, subject, mood, color palette
- Match the user's requested image style from the interview
- Include color palette hints so images don't clash with slide design
- Example: `"Flat illustration of a person planting a tree in an urban park, earthy greens and warm terracotta palette, soft lighting, minimal background, modern infographic style"`

**Fallback — if Gemini CLI is unavailable or image generation fails:**
1. Note which slides need images
2. Use PptxGenJS shapes, icons, and color blocks as placeholders
3. Print a list of image prompts the user can manually generate and drop in
4. Never leave a slide image-less without a designed visual placeholder

---

## Phase 5 — Build the Deck

Read [`references/pptxgenjs-api.md`](references/pptxgenjs-api.md) for all code patterns.

### Setup

```bash
npm install -g pptxgenjs
mkdir -p /home/claude/slides-output/images
```

### Anti-Hallucination Rules (CRITICAL)

> **Only include information the user explicitly provided, or facts Claude knows with certainty from training.** If a stat, name, date, or claim wasn't in the user's input and you're not 100% sure of it, either leave it out or flag it with `[USER TO VERIFY]`. Never invent research, statistics, quotes, names, or outcomes.

Specifically:
- If the user said "add some stats about X" without providing them → ask for them, don't invent
- If a chart is requested without data → ask for the numbers, don't use placeholder data as if real
- If a quote is requested → use only real, verified quotes; flag uncertain ones

### Build Workflow

1. **Install dependencies** — `npm install -g pptxgenjs`
2. **Organize images** — move Gemini-generated images to `/home/claude/slides-output/images/`
3. **Write the build script** — one JS file per presentation at `/home/claude/slides-output/build.js`
4. **Run it** — `node /home/claude/slides-output/build.js`
5. **Output to** `/mnt/user-data/outputs/[presentation-name].pptx`

### Slide Structure Rules

Read [`references/design-system.md`](references/design-system.md) for full layout and visual guidelines. Key rules:

- **Every slide needs a visual element** — image, chart, icon, or designed shape. No text-only slides.
- **Vary layouts** across slides — don't use the same template for every slide
- **Title slide + conclusion slide** get the boldest treatment (dark background, large type)
- **Content slides** get a lighter/white background for readability
- **Font sizes**: titles 36–44pt, section headers 20–24pt, body 14–16pt, captions 10–12pt
- **Commit to ONE visual motif** and carry it across every slide
- **NEVER use accent lines under titles** — AI slop tell. Use whitespace instead.

---

## Phase 6 — Speaker Notes

If the user requested speaker notes:

1. **Embed notes in each slide** via PptxGenJS `slide.addNotes()`
2. **Also export to a separate file**: `/mnt/user-data/outputs/[name]-speaker-notes.md`

Speaker notes format per slide:
```markdown
## Slide N — [Slide Title]

**What to say:**
[2-4 sentences expanding on the slide content. Not a script — talking points. Conversational, not robotic.]

**Key point to land:**
[The one thing the audience should take away from this slide]

**Transition:**
[How to move to the next slide naturally]

---
```

Notes should:
- Expand on bullet points with context not shown on the slide
- Sound like a human talking, not reading a document
- Be calibrated to the audience (simpler for a class, sharper for investors)
- Never introduce new facts not on the slide or in the user's source material

---

## Phase 7 — QA

### Content Check
```bash
# Extract all text from the finished deck
extract-text /mnt/user-data/outputs/presentation.pptx
```
- Verify no placeholder text remains (`lorem`, `TODO`, `[insert]`, `XXX`)
- Verify all user-provided facts appear correctly
- Verify no slide is missing its title

### Visual Check
```bash
# Convert to images for visual inspection
python scripts/office/soffice.py --headless --convert-to pdf /mnt/user-data/outputs/presentation.pptx
rm -f /home/claude/slides-output/slide-*.jpg
pdftoppm -jpeg -r 150 /mnt/user-data/outputs/presentation.pdf /home/claude/slides-output/slide
ls -1 "$PWD"/home/claude/slides-output/slide-*.jpg
```

Inspect each slide for:
- Text overflow / cut-off content
- Overlapping elements
- Low contrast (text vs background)
- Cramped spacing (< 0.3" gaps)
- Missing images (blank placeholder)
- Inconsistent margins

Fix issues, then re-render. **Stop after one fix-and-verify cycle** unless a new user-visible defect appears.

---

## Phase 8 — Deliver

Present outputs to the user:

```
✅ Presentation complete!

📊 Slides:        [name].pptx  (N slides)
📝 Speaker notes: [name]-speaker-notes.md
🖼️  Images used:   N generated via Gemini CLI
```

If any images failed to generate, list their prompts so the user can generate and swap them in manually.

If any facts were flagged `[USER TO VERIFY]`, list them explicitly and ask the user to confirm or correct them before using the deck.

---

## Quick Reference

| Phase | What happens |
|-------|-------------|
| 1 — Interview | Ask all questions about content, design, tone, features |
| 2 — Design Brief | Print summary, get approval |
| 3 — Outline | Show slide-by-slide plan, get approval |
| 4 — Images | Generate all images via Gemini CLI (fallback to shapes) |
| 5 — Build | Write + run PptxGenJS build script |
| 6 — Speaker Notes | Embed in .pptx + export `speaker-notes.md` |
| 7 — QA | Content check + visual inspection + fixes |
| 8 — Deliver | Present files, flag any unresolved items |
