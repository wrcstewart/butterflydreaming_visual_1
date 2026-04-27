# ButterflyDreaming Visual Module — Specification v0.1

## Overview

This document is the source of truth for the first exemplar visual media module
for the ButterflyDreaming platform. It describes two HTML files — a harness and a
visual module — that together demonstrate the BD media module protocol for graphics.
Claude Code should read this document in full before writing any code and refer
back to it throughout.

This spec is a companion to MUSIC_MODULE_SPEC.md. The BD protocol (postMessage,
%%bd_ directives, parseBD parser) is identical. Only the rendering domain differs.

---

## Context

ButterflyDreaming is a dyadic encounter platform documented at butterflydreaming.info.
This visual module renders kolam — Tamil sacred geometry — using L-systems. Kolam
is chosen for its deep resonance with the platform's philosophy: the mandatory
closed-loop constraint mirrors the dyadic encounter that always resolves into a
published child node; the threshold drawing practice mirrors the platform as a
threshold between self and other; the Tamil origin adds a South Indian sacred
geometry tradition alongside the Chinese (Zhuangzi), Western biological
(symbiogenesis) and mathematical (Lorenz) foundations of the platform.

---

## Project Structure

```
butterflydreaming_visual_1/
  VISUAL_MODULE_SPEC.md       ← this file
  index.html                  ← harness (to be created)
  visual_module.html          ← the visual media module (to be created)
```

No sample files are required — all rendering is done in JavaScript on the canvas.

---

## Libraries

Both HTML files load all libraries from CDN — no npm, no build step.

| Library       | Purpose                          | CDN |
|---------------|----------------------------------|-----|
| lindenmayer   | L-System rewriting engine        | https://cdn.jsdelivr.net/npm/lindenmayer/dist/lindenmayer.browser.js |

No other libraries are required. All canvas drawing is vanilla JavaScript.

---

## The BD Directive Syntax

This module uses the BD directive syntax as defined in the BD Node Format
Specification. For reference:

- Single-line directive: terminated by newline
- Multi-line directive: opened by [ on same line, closed by %%bd_] on its own line
- %%bd_module appears once at the top
- Modules interpret directives they understand and silently ignore the rest

---

## The %%bd_ Directives for This Module

### Required
```
%%bd_module visual_module.html
```

### Rendering Directives

| Directive          | Type    | Default    | Description |
|--------------------|---------|------------|-------------|
| %%bd_symmetry      | integer | 8          | N-fold rotational symmetry (1–16) |
| %%bd_depth         | integer | 4          | L-system rewriting iterations |
| %%bd_step          | number  | 40         | Turtle step length in pixels |
| %%bd_angle         | number  | 90         | Turtle turn angle in degrees |
| %%bd_stroke        | color   | #4a9b8e    | Line colour (hex) |
| %%bd_background    | color   | #0a0a0f    | Canvas background colour (hex) |
| %%bd_weight        | number  | 1.5        | Stroke line weight in pixels |

### The Score Directive

The L-system rules are carried in a multi-line %%bd_score block:

```
%%bd_score [
axiom: FBFBFBFB
A: AFBFA
B: AFBFBFBFA
%%bd_]
```

The score block contains:
- One `axiom:` line — the starting string
- One or more production rules in the form `Symbol: replacement`
- Lines beginning with # are comments and are ignored

The symbol alphabet is:
| Symbol | Meaning |
|--------|---------|
| F      | Move forward drawing a curved line segment |
| f      | Move forward without drawing |
| +      | Turn left by %%bd_angle degrees |
| -      | Turn right by %%bd_angle degrees |
| [      | Push current position and angle to stack |
| ]      | Pop position and angle from stack |
| \|     | Turn 180 degrees (reverse direction) |

---

## Default L-System Score

The default score for this module is an 8-fold kolam-inspired pattern that
produces an interlocking closed loop across the canvas.

```
%%bd_module visual_module.html
%%bd_symmetry 8
%%bd_depth 4
%%bd_step 40
%%bd_angle 45
%%bd_stroke #4a9b8e
%%bd_background #0a0a0f
%%bd_weight 1.5
%%bd_score [
axiom: F+F+F+F+F+F+F+F
F: F-F+F+F-F
%%bd_]
```

The angle of 45 degrees and the eightfold rotational symmetry create a more
authentic kolam-like interlocking pattern than a simple 90-degree grid grammar.

---

## The postMessage Protocol

Identical to the music module. Reproduced here for completeness.

### Harness → Module
```javascript
// Load and render a node
{ type: "BD_INIT", payload: { text: "...full node text..." } }

// Stop rendering (clear canvas)
{ type: "BD_STOP" }
```

### Module → Harness
```javascript
// Module loaded and canvas ready
{ type: "BD_READY" }

// User edited text and pressed Send Back
{ type: "BD_UPDATE", payload: { text: "...updated node text..." } }

// Rendering error
{ type: "BD_ERROR", payload: { message: "description" } }
```

During development use origin "*". Note in code that production should
use "https://butterflydreaming.info".

---

## File 1: index.html

### Purpose
Simulates the main ButterflyDreaming platform for development and testing.
Not part of the final platform.

### Explanatory Header
Add an explanatory section at the top of the page, in a calm understated
style, smaller than the main UI elements, off-white at reduced opacity (0.6),
separated from the controls by a thin horizontal rule. Text:

---

SIMPLE EXAMPLE OF TEXT TO MEDIA — ButterflyDreaming Platform

This page demonstrates how a text node from the ButterflyDreaming graph can
drive a visual media module. The L-system rules and %%bd_ directives shown
below would be found in a graph node, discovered and edited collaboratively
by two anonymous users during a dyadic encounter.

The %%bd_ directives are a shared platform language — each directive is
available to all media modules, which interpret them in their own way or
ignore them silently. A directive that controls reverb in the music module
might influence colour or motion in a visual module.

The kolam tradition of Tamil Nadu draws closed loops around a grid of dots
at the threshold of the home at dawn. This module renders kolam using
L-system rewriting — the same mathematical framework as plant growth and
fractal geometry. In ButterflyDreaming, the mandatory closed-loop constraint
of kolam mirrors the dyadic encounter: every path must return to its origin.

---

### Layout
Two-column layout:

**Left column — Controls:**
- Label: EXTENDED L-SYSTEM NOTATION
- Textarea (id: ls-input) pre-populated with the default score above
- Button: "Send to Player"
- Status area showing last message received from module

**Right column — Player:**
- iframe (id: visual-module) pointing to visual_module.html
- Suggest 600x600px — square to suit the radially symmetric output

### Behaviour
1. On page load the textarea is pre-populated with the default score
2. On page load, after a short delay (500ms) to allow the iframe to initialise,
   send BD_INIT automatically so the kolam renders immediately on opening
3. When "Send to Player" is clicked: read textarea, post BD_INIT to iframe
4. Listen for postMessage events:
   - BD_READY → status: "Module ready"
   - BD_UPDATE → update textarea with payload.text, status: "Received update"
   - BD_ERROR → status: "Error: [message]"

---

## File 2: visual_module.html

### Purpose
The visual media module. Receives L-system notation via postMessage, renders
a kolam using L-system rewriting and turtle graphics on a canvas element with
n-fold rotational symmetry.

### Layout
Minimal layout:
- Status line at top (e.g. "Ready", "Rendering...", "Error")
- Canvas element (id: kolam-canvas) filling most of the available space
  — suggest 560x560px, centred
- "Send Back" button at bottom

### Initialisation Sequence
1. On page load, post BD_READY to parent immediately
2. Listen for postMessage events from parent

### BD_INIT Handling

On receiving BD_INIT:

1. Parse the full node text using parseBD() — extract all %%bd_ directives
   and the %%bd_score block
2. Parse the score block to extract axiom and production rules
3. Apply directives to rendering parameters (with defaults for any missing)
4. Run the L-system rewriting using the lindenmayer library
5. Run the turtle interpreter to produce a path for one sector
6. Apply n-fold symmetry stamping to the canvas
7. Check the kolam closed-loop constraint (Option B — warn only)
8. Update status to "Ready" on success

### BD_STOP Handling
Clear the canvas and update status to "Stopped".

---

## The parseBD() Parser

The BD parser is a two-state machine identical to the one specified in the
BD Node Format Specification. It returns a plain object of directive name/value
pairs. The %%bd_score value is the raw multi-line string between the [ and %%bd_].

```javascript
function parseBD(text) {
  const directives = {}
  const lines = text.split('\n')
  let state = 'text'
  let currentDirective = null
  let currentLines = []

  for (const line of lines) {
    if (state === 'text') {
      if (line.startsWith('%%bd_')) {
        const spaceIdx = line.indexOf(' ')
        if (spaceIdx === -1) continue
        const name = line.slice(5, spaceIdx)
        const value = line.slice(spaceIdx + 1).trim()
        if (value === '[') {
          currentDirective = name
          currentLines = []
          state = 'bracket'
        } else {
          directives[name] = value
        }
      }
    } else if (state === 'bracket') {
      if (line.trim() === '%%bd_]') {
        directives[currentDirective] = currentLines.join('\n')
        state = 'text'
      } else {
        currentLines.push(line)
      }
    }
  }
  return directives
}
```

---

## The Score Parser

Parse the %%bd_score value to extract axiom and productions:

```javascript
function parseScore(scoreText) {
  const result = { axiom: '', productions: {} }
  for (const line of scoreText.split('\n')) {
    const trimmed = line.trim()
    if (!trimmed || trimmed.startsWith('#')) continue
    if (trimmed.startsWith('axiom:')) {
      result.axiom = trimmed.slice(6).trim()
    } else {
      const colonIdx = trimmed.indexOf(':')
      if (colonIdx !== -1) {
        const symbol = trimmed.slice(0, colonIdx).trim()
        const replacement = trimmed.slice(colonIdx + 1).trim()
        result.productions[symbol] = replacement
      }
    }
  }
  return result
}
```

---

## The L-System Rewriting

Use the lindenmayer library to rewrite the axiom:

```javascript
const lsystem = new LSystem({
  axiom: parsed.axiom,
  productions: parsed.productions
})
lsystem.iterate(depth)
const result = lsystem.getString()
```

`depth` is the value of %%bd_depth (default 4). At depth 4 the string may be
several thousand characters — this is normal and renders quickly.

Important: depth 6 and above can produce extremely long strings and may cause
browser performance issues. Cap the maximum depth at 6 in the UI and the parser.

---

## The Turtle Interpreter

### Key Design Decision — Curved Lines for Kolam Aesthetic

Standard L-system turtle graphics use straight line segments. Authentic kolam
uses flowing curved lines looping around dots. To achieve the kolam aesthetic,
the turtle renderer must use quadratic bezier curves rather than straight lines
for F moves.

Implementation: at each F move, instead of drawing a straight line from current
position to next position, draw a quadratic bezier curve with a control point
offset perpendicular to the direction of travel. The offset amount controls
the curviness — suggest a value of step * 0.3 as a starting point.

```javascript
function drawCurvedStep(ctx, x, y, angle, step) {
  const rad = (angle * Math.PI) / 180
  const nx = x + Math.cos(rad) * step
  const ny = y + Math.sin(rad) * step
  // control point offset perpendicular to direction
  const perpRad = rad + Math.PI / 2
  const cpOffset = step * 0.3
  const cpx = (x + nx) / 2 + Math.cos(perpRad) * cpOffset
  const cpy = (y + ny) / 2 + Math.sin(perpRad) * cpOffset
  ctx.quadraticCurveTo(cpx, cpy, nx, ny)
  return { x: nx, y: ny }
}
```

### Turtle State
```javascript
let state = {
  x: 0, y: 0,       // current position
  angle: 0,          // current heading in degrees
  stack: []          // for [ and ] branching
}
```

### Symbol Dispatch
```javascript
for (const symbol of lsystemResult) {
  switch (symbol) {
    case 'F': // move forward drawing curved line
    case 'f': // move forward without drawing
    case '+': // turn left
    case '-': // turn right
    case '[': // push state
    case ']': // pop state
    case '|': // reverse (turn 180)
    // all other symbols: ignore silently
  }
}
```

All symbols not in the above list are silently ignored — this is important
for L-system symbols used only as rewriting placeholders (e.g. A, B, X).

---

## The Symmetry Wrapper

The symmetry wrapper is applied at the renderer level, not in the grammar.
Draw the turtle output once into an offscreen canvas, then rotate and stamp
it n times around the centre:

```javascript
function applySymmetry(ctx, offscreen, symmetry, canvasSize) {
  const cx = canvasSize / 2
  const cy = canvasSize / 2
  const sectorAngle = (2 * Math.PI) / symmetry

  ctx.save()
  ctx.translate(cx, cy)
  for (let i = 0; i < symmetry; i++) {
    ctx.rotate(sectorAngle)
    ctx.drawImage(offscreen, -cx, -cy)
  }
  ctx.restore()
}
```

The turtle starts at the centre of the offscreen canvas (or slightly offset
depending on the pattern). Experiment with starting position — for FBFB-type
kolam grammars, starting at the centre works well.

---

## The Kolam Closed-Loop Constraint (Option B — Warn Only)

After turtle interpretation, check whether the final position is approximately
equal to the starting position (within a tolerance of step / 2):

```javascript
function checkClosure(startX, startY, endX, endY, step) {
  const dist = Math.sqrt((endX - startX) ** 2 + (endY - startY) ** 2)
  return dist < step / 2
}
```

If the path does not close, display a subtle warning in the status area:
"Rendered — note: path does not close (kolam constraint not satisfied)"

Do not prevent rendering — this is a creative tool, not a validator.
An unclosed path may still produce interesting and beautiful results.

---

## The Rendering Sequence

The complete rendering sequence on receiving BD_INIT:

1. Parse directives from node text using parseBD()
2. Parse score block using parseScore()
3. Set rendering parameters with defaults for any missing directives
4. Clear canvas, fill with %%bd_background colour
5. Create offscreen canvas same size as main canvas
6. Run L-system iteration using lindenmayer
7. Run turtle interpreter on offscreen canvas, recording start/end positions
8. Apply symmetry wrapper — stamp offscreen canvas n times onto main canvas
9. Check closure constraint, update status accordingly
10. Post BD_READY to parent if this is the first render, otherwise no message

---

## Visual Design

Consistent with the music module aesthetic:

- Background: %%bd_background (default #0a0a0f — near black)
- Canvas border: a subtle 1px teal border (#4a9b8e at 0.3 opacity)
- Status text: off-white (#e8e8e0) at 0.7 opacity, small, below the canvas
- Send Back button: dark background, teal border and text (#4a9b8e)
- Font: Georgia or system-ui serif
- No gradients, no shadows, no animations
- The canvas itself is the visual focus — keep all UI elements minimal

A brief comment at the top of each file:
"ButterflyDreaming Visual Module — [filename] — BD Protocol v0.1"

---

## Error Handling

- If %%bd_score is missing: post BD_ERROR "No score found in node text"
- If axiom is missing from score: post BD_ERROR "No axiom found in score"
- If L-system produces empty string: post BD_ERROR "L-system produced no output"
- If depth > 6: cap at 6 and display a status note "Depth capped at 6"
- If canvas context unavailable: post BD_ERROR "Canvas not supported"
- Wrap all postMessage handlers in try/catch

---

## Testing Checklist

Before considering this exemplar complete, verify:

- [ ] BD_READY posted on page load
- [ ] BD_INIT received → kolam renders on canvas
- [ ] Default 8-fold kolam renders correctly with curved lines
- [ ] Symmetry stamping produces visually correct n-fold pattern
- [ ] Closure check works — console message for open paths
- [ ] BD_STOP clears the canvas
- [ ] Send Back button posts BD_UPDATE with current textarea text
- [ ] Harness textarea updates when BD_UPDATE received
- [ ] Changing axiom or productions and pressing Send to Player re-renders
- [ ] BD_ERROR displayed correctly for malformed score
- [ ] Depth 6 cap works without crashing the browser
- [ ] Renders correctly in Chrome, Firefox and Safari

---

## What This Exemplar Demonstrates

For third-party module developers, this file shows:

1. How to receive BD_INIT and parse the %%bd_ directives and score
2. How to use the parseBD() and parseScore() functions
3. How to use the lindenmayer library from CDN
4. How to implement a turtle graphics renderer on canvas
5. How to apply n-fold symmetry as a renderer wrapper
6. How to post BD_READY, BD_UPDATE and BD_ERROR
7. The expected visual register of a BD visual module
8. That module type (music vs visual) does not affect the BD protocol —
   the postMessage API is identical

---

## Notes for Claude Code

- Do not use any framework — vanilla HTML, CSS and JavaScript only
- Do not use npm or any build tool — CDN only
- Keep each file self-contained — no shared JS files
- Use ES6+ JavaScript throughout
- The lindenmayer library is loaded from jsDelivr CDN as a browser bundle —
  it exposes a global LSystem constructor
- The turtle interpreter must use quadratic bezier curves for F moves,
  not straight lines — this is essential for the kolam aesthetic
- The symmetry wrapper draws to an offscreen canvas first — do not stamp
  directly onto the main canvas in the loop
- All symbols in the L-system result that are not in the turtle alphabet
  (F f + - [ ] |) must be silently ignored
- The default score must render a recognisable 8-fold kolam pattern
- Cap depth at 6 to protect browser performance
- Add clear comments explaining each section, especially the symmetry wrapper
  and the curved line turtle implementation

---

*ButterflyDreaming — a reflective ecosystem — Visual Module Spec v0.1*

### Amendment 1 — Correction to Kolam Philosophical Note and Title

The introductory text in the Context section states that the dyadic encounter
"always resolves into a published child node". This is not accurate — two users
are not required to agree or produce a child node. The encounter may conclude
without a child node being created. The closed-loop constraint of kolam is
therefore an aspiration or metaphor rather than a literal parallel.

The explanatory header text in index.html contains the same statement and
should be softened accordingly. Replace:

"the mandatory closed-loop constraint of kolam mirrors the dyadic encounter:
every path must return to its origin"

with:

"the closed-loop constraint of kolam — every path returning to its origin —
resonates with the aspiration of the dyadic encounter toward resolution
and return"

The title of index.html should be:

BUTTERFLY DREAMING - SIMPLE EXAMPLE OF TEXT TO MEDIA (VISUAL KOLAM)

Respect the capitalisation exactly as shown above.