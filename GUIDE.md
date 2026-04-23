# Lecture Creation Guide

Comprehensive reference for creating new lectures in the Python OOP course at oop.szturo.online.

## Project Overview

Static HTML presentations for a Python OOP course at VILNIUS TECH / Visma.
- Each lecture = one standalone HTML file (`NN-topic.html`)
- Shared `shared.css` + `shared.js` for navigation, theme, quiz reveal
- All images in `assets/`
- Hosted on Vercel at `oop.szturo.online`
- Each lecture is 70-90 minutes (~55-65 slides)

## Existing Lectures

| # | Title | File |
|---|-------|------|
| 01 | Classes & Objects | `01-classes-objects.html` |
| 02 | Encapsulation & Access Control | `02-encapsulation.html` |
| 03 | Inheritance | `03-inheritance.html` |
| 04 | Polymorphism & PEP 8 | `04-polymorphism.html` |
| 05 | Composition & Aggregation | `05-composition.html` |
| 06 | Exception Handling & Logging | `06-exception-handling.html` |
| 07 | SOLID Principles | `07-solid-principles.html` |
| 08 | UML & Design Patterns | `08-uml-design-patterns.html` |
| 09 | Design Patterns (Part 2) | `09-uml-design-patterns.html` (not yet deployed) |

## Standard Slide Structure

Each lecture follows this exact pattern:

1. **Title slide** (1) — Gradient heading, SVG preview diagram, "Course N" subtitle
2. **Curriculum** (2) — All 12 topics listed, current one highlighted in red
3-8. **Recap quizzes** (3 quizzes + 3 explanation slides) — from the **LAST 3 LECTURES**
9. **Agenda** — bullet list with emojis for each topic
10+. **Content sections** — real-world analogy → problem → solution → code → pros/cons
N-1. **Key Takeaways** — summary with "Next up: [next lecture]"

### Recap Quiz Rules

- Always recap the **last 3 lectures** (never repeat questions across lectures!)
- Check existing quizzes in previous HTML files before writing new ones
- Quiz format: 4 options (A/B/C/D), correct answer highlighted on 2nd click
- Prefer **code-based questions** over memorized definitions — show code, ask "what violates X?"
- Each quiz slide is followed by an explanation slide with recap content

## HTML Structure Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Topic Name: Course N — Python OOP</title>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600&family=Outfit:wght@300;400;600;700;800&display=swap" rel="stylesheet">
<link rel="icon" href="vgtu.jpg">
<link rel="stylesheet" href="shared.css">
<style>
  /* Any lecture-specific animations */
  .reveal-slide .reveal-blocks .reveal-block { opacity: 0; transform: translateY(8px); transition: opacity 0.4s, transform 0.4s; }
  .reveal-slide.step-1 .reveal-blocks .reveal-block:nth-child(1) { opacity: 1; transform: translateY(0); }
  .reveal-slide.step-2 .reveal-blocks .reveal-block:nth-child(-n+2) { opacity: 1; transform: translateY(0); }
  .reveal-slide.step-3 .reveal-blocks .reveal-block { opacity: 1; transform: translateY(0); }
  /* Overflow fixes */
  pre { overflow-x: auto; max-width: 100%; }
  .two-col, .three-col { overflow: hidden; }
  svg { max-width: 100%; height: auto; }
  .col { overflow: hidden; word-wrap: break-word; }
</style>
</head>
<body>

<div class="mobile-warning"><!-- standard mobile warning block --></div>
<div class="progress" id="progress"></div>
<div class="visma-logo"><!-- Visma + VGTU logos --></div>

<!-- SLIDES -->

<div class="nav">
  <button id="skip-quiz" class="skip-quiz" onclick="skipQuiz()">Skip Quiz &nbsp;&#9654;</button>
  <button id="theme-toggle" onclick="toggleTheme()" title="Toggle light/dark mode" style="font-size: 1.2rem;">&#9788;</button>
  <button id="prev" onclick="navigate(-1)">&lsaquo;</button>
  <span class="counter" id="counter"></span>
  <button id="next" onclick="navigate(1)">&rsaquo;</button>
</div>

<script src="shared.js"></script>
<script>
(function() {
  /* Standard reveal-slide extension + URL hash navigation */
})();
</script>
</body>
</html>
```

**Copy the full boilerplate from `07-solid-principles.html` or `08-uml-design-patterns.html` as a starting point.** They have the complete mobile-warning, logos, nav, hash navigation script.

## Design System

### Colors (in `shared.css`)

```css
--accent: #ef4444          /* Red — primary accent */
--green: #34d399           /* Success / correct */
--cyan: #22d3ee            /* Code class names */
--orange: #fb923c          /* Warnings */
--purple: #a78bfa          /* Secondary highlights */
--pink: #f472b6            /* Tertiary */
--blue: #60a5fa            /* Links, info */
--heading: #fff            /* Pure white headings */
--muted: #8b8fa4           /* Secondary text */
--text: #e2e4ea            /* Body text */
--bg: #0c0e14              /* Page background */
--surface: #161923         /* Card/slide backgrounds */
--code-bg: #111320         /* Code blocks */
```

### Section Accent Colors (for SOLID / SOLID-like sections)

- `#818cf8` (S / Singleton / section 1)
- `#a78bfa` (O / Builder / section 2)
- `#f472b6` (L / section 3)
- `#fbbf24` (I / section 4)
- `#34d399` (D / section 5)

### Typography

- **Outfit** — headings, body text (300-800 weights)
- **JetBrains Mono** — code, class names, diagram labels
- h1: 3.4rem, 800 weight, gradient from white to accent
- h2: 2.1rem, 700 weight
- h3: 1.2rem, 600 weight, accent color
- Body: 1.15rem, 1.7 line-height
- Code blocks: 0.72-0.85rem (use 0.68rem for two-column code)

### Slide Layouts

**Full-width content:**
```html
<div class="slide">
  <h2>Heading</h2>
  <div style="max-width: 700px;">...</div>
</div>
```

**Two-column (text + visual):**
```html
<div class="slide">
  <h2>Heading</h2>
  <div class="layout">
    <div class="text">...</div>
    <div class="visual">
      <img src="assets/foo.png" style="max-height: 400px; border-radius: 8px;">
    </div>
  </div>
</div>
```

**Three-column cards (for categorization):**
```html
<div class="three-col" style="margin-top: 1.5rem;">
  <div class="col" style="border-color: #818cf8; text-align: center;">
    <p style="font-size: 2.5rem; margin-bottom: 0.3rem;">&#127981;</p>
    <h3 style="color: #818cf8;">Title</h3>
    <p style="font-size: 0.9rem;">Description</p>
  </div>
  <!-- 2 more cols -->
</div>
```

**Centered section divider:**
```html
<div class="slide center" style="background: var(--surface);">
  <h2>Section Title</h2>
  <p style="color: var(--muted); font-style: italic;">Quote or intro text</p>
</div>
```

**Centered image showcase:**
```html
<div class="slide center">
  <h2>Topic</h2>
  <div style="display: flex; justify-content: center;">
    <img src="assets/foo.png" style="max-height: 500px; border-radius: 8px;">
  </div>
</div>
```

## Key Interactive Mechanisms

### Quiz Slides (Two-Step Reveal)

```html
<div class="slide quiz-slide">
  <h2>Quiz Time! <span style="color: var(--muted); font-size: 1rem;">(Lecture NN)</span></h2>
  <p style="color: var(--accent); font-size: 1rem; font-weight: 600;">Question text?</p>
  <!-- Optional code block for code-based questions -->
  <pre style="font-size: 0.72rem;">...</pre>
  <div class="two-col quiz-answers" style="margin-top: 1rem; max-width: 700px;">
    <div class="col quiz-answer"><p>A) Option</p></div>
    <div class="col quiz-answer correct"><p><strong>B) Correct option</strong></p></div>
    <div class="col quiz-answer"><p>C) Option</p></div>
    <div class="col quiz-answer"><p>D) Option</p></div>
  </div>
  <p class="quiz-explanation">Why B is correct...</p>
</div>
```

First click reveals options, second reveals correct answer + explanation.

### Progressive Code Reveal (3 Blocks)

```html
<div class="slide reveal-slide">
  <h2>Heading</h2>
  <div class="reveal-blocks">
    <div class="reveal-block"><p>Label 1:</p><pre>code...</pre></div>
    <div class="reveal-block"><p>Label 2:</p><pre>code...</pre></div>
    <div class="reveal-block"><p>Label 3:</p><pre>code...</pre></div>
  </div>
</div>
```

Each click reveals the next block. After 3, navigation moves to next slide.

### URL Hash Navigation

Include the custom script (copy from lecture 07) that:
- Adds `#N` to URL when navigating
- Jumps to slide N on page load if hash present
- Responds to `hashchange` events

## Content Patterns for Each Concept

For each major concept/pattern/principle, use this flow:

1. **Section title slide** — divider with `background: var(--surface)` + quote
2. **Real-world analogy** — 2-3 non-programming examples in a `.three-col` grid with emoji icons
3. **Problem** — what breaks without this concept
4. **Solution** — how the concept solves it (with UML diagram if applicable)
5. **Code example** — in reveal-slide (violation → fix)
6. **Pros/cons** — two-column list
7. **When to use / when NOT to use**

## Images & Assets

### Asset folder structure

```
assets/
  <topic>-<name>.png         e.g. uml-classes-methods.png
  dp-<pattern>.png           e.g. dp-singleton.png, dp-builder.png
  dp-icon-<pattern>.png      small icons from refactoring.guru
  solid-<principle>-<name>.png
```

### Standard image prompt for generation

```
Flat vector illustration, clean minimal style, soft rounded shapes, muted
pastel colors with one vibrant accent. Solid bright green (#00FF00)
background. No text, no labels, no watermarks, no human hands or body
parts. The main object must be centered with generous padding (at least
15% empty space on all sides) so it can be easily cut out. Style
consistent with tech presentation slides — modern, friendly, slightly
playful but professional. Thick outlines, simple shadows, 2D flat
design.

[SPECIFIC SUBJECT]
```

Then remove green background with this Python script:

```python
from PIL import Image
import numpy as np
img = Image.open("path.png").convert("RGBA")
data = np.array(img, dtype=np.float64)
r, g, b = data[:,:,0], data[:,:,1], data[:,:,2]
green_strength = g - np.maximum(r, b)
fully_green = green_strength > 80
data[fully_green] = [0, 0, 0, 0]
semi_green = (green_strength > 30) & (green_strength <= 80)
alpha_factor = 1.0 - ((green_strength[semi_green] - 30) / 50.0)
data[semi_green, 3] = (alpha_factor * 255).clip(0, 255)
data[semi_green, 1] = (data[semi_green, 1] * alpha_factor).clip(0, 255)
Image.fromarray(data.astype(np.uint8)).save("out.png")
```

### Emoji Icons (preferred over images for section markers)

Use HTML entities in bullet lists and headings:
- &#127981; Factory / building
- &#129513; Puzzle piece
- &#128172; Speech bubble
- &#128161; Lightbulb
- &#9989; Checkmark
- &#10060; X / cross
- &#128270; Magnifier / search
- &#128296; Construction / building
- &#128736;&#65039; Tools / wrench
- &#128683; No entry
- &#128214; Book
- &#127987; Flag (for singleton analogies)
- &#128203; Clipboard
- &#127979; School
- &#127828; Burger
- &#127829; Pizza
- &#128722; Shopping bag (IKEA)
- &#128690; Bicycle
- &#9889; Lightning

## Writing Style

From `CLAUDE.md`:
- **No comments in code** (no `# this does X` — code speaks for itself)
- **Don't add functionality not asked for**
- **Small methods, SOLID principles**
- **Don't agree with every sentence, push back when there's a better approach**
- Keep tone casual and educational
- Use `**bold**` for key terms, italic quotes for definitions

## UML & Diagrams

For this course, **use real UML notation** (since students learn UML in lecture 08):
- Class boxes with 3 sections: name / attributes / methods
- Horizontal lines between sections
- Visibility markers: `+` public, `-` private, `#` protected, `~` package
- Inheritance: hollow triangle arrow pointing to parent
- Association: plain line
- Aggregation: empty diamond
- Composition: filled diamond
- Dependency: dashed arrow
- Multiplicity: `1`, `0..1`, `1..*`, `*`, `n..m`

**Always inline SVGs** for diagrams — they inherit the theme colors via `var(--code-bg)`, `var(--border)`, etc.

## Live Coding Exercise

Create `NN-live-coding-<topic>.md` alongside the HTML:
- Step-by-step (Steps 1-N, grouped in Parts A-E)
- Python code blocks (no comments)
- Output blocks showing expected output
- Discussion callouts after key steps
- Comparison table at the end
- 5+ discussion questions

**Format**: Match `06-live-coding-exceptions.md` or `07-live-coding-solid.md` exactly.

## Parallel Agent Workflow

For large lectures (50+ slides), dispatch agents in parallel:

1. **Create team**: `TeamCreate({ team_name: "course-NN", description: "Build lecture NN" })`
2. **HTML agent**: Full PDF content + structural outline + reference to existing lecture
3. **Live coding agent**: Exercise file based on existing format
4. **Playwright verifier** (after HTML done): Visual QA of all slides

### Agent prompt template

```
Create /Users/tomek/dev/oop-slides/NN-topic.html — a lecture slide deck
for "Topic Name, Course N" in the Python OOP course.

FIRST: Read /Users/tomek/dev/oop-slides/08-uml-design-patterns.html to
understand the EXACT HTML structure, styling patterns, boilerplate.
Copy the same structure exactly.

IMPORTANT RULES from CLAUDE.md:
- Don't add comments to code
- Use emojis/icons on bullet-heavy slides
- Use reveal-slide progressive disclosure for code examples
- Include URL hash navigation (same script as lecture 08)
- Font sizes: 0.75rem for code in reveal slides, 0.68rem for two-column

SLIDE STRUCTURE (~55-60 slides for 70-90 min):
[detailed outline of every slide]

Make sure NO quiz questions duplicate those in lectures N-1, N-2, N-3.
Read those files first to check.
```

## Common Pitfalls (From Past QA)

1. **Right-edge overflow** — code blocks extend beyond viewport. Fix: reduce font size, add `overflow-x: auto`, constrain `max-width: 100%`
2. **Bottom overflow on reveal slides** — when all 3 blocks shown. Fix: reduce font size or content
3. **Hardcoded dark gradients** — break in light mode. Use `var(--surface)` instead
4. **SVG text truncation** — text extends beyond viewBox. Fix: widen viewBox (use negative x if needed)
5. **Arrows not connecting** — line endpoints don't match box edges. Compute endpoint math carefully
6. **Duplicate quiz questions** across lectures — ALWAYS check previous lectures' quizzes first
7. **Too much theory without real-world analogies** — add emoji-illustrated analogy slides before code
8. **Sparse slides** — 3 empty boxes with one word each. Add lists of examples, icons, descriptions

## Playwright Verification

Dispatch an agent with this prompt template:

```
Verify the visual quality of http://localhost:8788/NN-topic.html using
Playwright browser tools.

Navigate through ALL N slides and check:
1. Title slide, curriculum (item N highlighted)
2. Quiz two-step reveal works
3. Code blocks: syntax highlighted, no overflow
4. Reveal-slide progressive disclosure works
5. All sections present
6. Pros/Cons two-column layouts
7. Key Takeaways at end
8. Theme toggle (dark/light)
9. Navigation buttons, progress bar, counter "N / total"
10. Visma + VGTU logos

Report ALL issues found with slide numbers.
Focus on: code fitting viewport, reveal-slide overflow, SVG connections.
```

Start dev server first: `python3 -m http.server 8788 --directory /Users/tomek/dev/oop-slides &`

## Git Workflow

1. Commit files individually with meaningful messages
2. **Don't commit Playwright screenshots** (`slide-*.png` in root)
3. **Don't commit source PDFs**
4. Push to `tsturo/oop-slides` on GitHub
5. Deploy with `vercel --prod --yes` (alias: `oop.szturo.online`)
6. **Don't auto-deploy** — only when user explicitly asks

## Index Page

Update `index.html` to add the new lecture:

```html
<a class="topic" href="NN-topic.html">
  <span class="num">NN</span>
  <span class="title">Topic Name</span>
  <span class="badge ready">ready</span>
</a>
```

Or keep as coming soon:
```html
<div class="topic disabled">
  <span class="num">NN</span>
  <span class="title">Topic Name</span>
  <span class="badge soon">coming soon</span>
</div>
```

---

## Prompt to Use After `/clear`

Copy-paste this prompt to start working on a new lecture:

```
Read /Users/tomek/dev/oop-slides/GUIDE.md — it's a comprehensive guide
for creating lectures in this Python OOP course. Then read the source
PDF for lecture NN at "/Users/tomek/dev/oop-slides/NN - Topic Name.pdf"
to understand the content.

Build lecture NN following the guide exactly:
1. Create NN-topic.html (use 08-uml-design-patterns.html as the HTML
   structure reference — copy the boilerplate)
2. Create NN-live-coding-topic.md (use 07-live-coding-solid.md as format
   reference)
3. Target 55-65 slides for 70-90 minutes of lecture time
4. Include recap quizzes from lectures N-1, N-2, N-3 (check their files
   first to avoid duplicate questions)
5. Follow the content pattern: section intro → real-world analogy →
   problem → solution → code (reveal-slide) → pros/cons
6. Use emojis liberally on bullet lists
7. Inline SVGs for UML diagrams using the theme variables
8. Update index.html to link the new lecture

Use parallel agents (create a team) to work on HTML + live coding +
Playwright verification simultaneously. Ask me for images if you need
real-world photos that can't be drawn with SVG.

Do NOT commit or push until I explicitly ask.
Do NOT deploy to Vercel automatically.
```

Replace `NN` and `Topic Name` with the actual lecture number and title.
