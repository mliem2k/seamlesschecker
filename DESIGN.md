---
name: Seamless Texture Checker
description: A tiny, cheerful tool for spotting seams in tileable textures
colors:
  primary: "#007bff"
  primary-deep: "#0056b3"
  ink: "#333333"
  white-panel: "#FFFFFFF2"
  scrim-idle: "#0000001A"
  scrim-drag: "#00000033"
  page-bg: "#F0F0F0"
  track-bg: "#DDDDDD"
  inspection-guide: "#FF000059"
typography:
  display:
    fontFamily: "'Comic Sans MS', 'Comic Sans', cursive"
    fontSize: "1.1em"
    fontWeight: 400
    lineHeight: normal
    letterSpacing: normal
  body:
    fontFamily: "Georgia, Calibri, serif"
    fontSize: "1em"
    fontWeight: 400
    lineHeight: normal
rounded:
  sm: "5px"
  lg: "15px"
  full: "50%"
spacing:
  xs: "6px"
  sm: "10px"
  md: "15px"
  lg: "20px"
components:
  panel:
    backgroundColor: "{colors.white-panel}"
    textColor: "{colors.ink}"
    rounded: "{rounded.lg}"
    padding: "{spacing.lg}"
  button-file:
    backgroundColor: "{colors.primary}"
    textColor: "#FFFFFF"
    rounded: "{rounded.lg}"
    padding: "6px 12px"
  button-file-hover:
    backgroundColor: "{colors.primary-deep}"
    textColor: "#FFFFFF"
    rounded: "{rounded.lg}"
    padding: "6px 12px"
  slider-track:
    backgroundColor: "{colors.track-bg}"
    rounded: "{rounded.sm}"
    height: "8px"
---

# Design System: Seamless Texture Checker

## 1. Overview

**Creative North Star: "The Rainbow Workbench"**

A small, cheerful toolbench floats over the real work surface, the tiled texture itself. Everything the user actually judges (does this image repeat cleanly, where's the seam) happens underneath, at full bleed, with nothing constraining or framing it. The workbench on top gets to be playful, colorful, a little toy-like, because it never competes with the image for attention: it's small, it's off to the side of center mass, and its one serious job (the size slider) stays legible even while the title above it is doing a rainbow dance.

This system explicitly rejects generic SaaS utility chrome: no card grids, no dashboard sidebar, no gradient hero panel pretending to be a landing page. It also rejects the opposite failure, a sterile gray "enterprise tool" look, since that would undersell the solo-dev charm that makes this fun to open. The personality lives entirely in the floating panel (title, button, slider); the canvas underneath is deliberately undecorated.

**Key Characteristics:**
- One floating panel, pinned near the top of an otherwise fully-bled canvas
- A rainbow gradient is the system's one recurring motif, used exactly twice (title text, slider fill), never as generic decoration elsewhere
- Playful, slightly bouncy motion throughout (float, pulse, bounce-in), always with a static reduced-motion fallback
- Comic Sans as a deliberate, singular personality signal on the one title, not a general UI font
- The panel is elevated at rest (a real shadow, not flat-by-default), it reads as a physical object sitting on top of the image, gaining more lift on hover/drag

## 2. Colors

A near-neutral, quiet base (white panel, light gray page) so the one recurring rainbow motif reads as intentional decoration rather than noise.

### Primary
- **Workbench Blue** (`#007bff`): the file-picker button and the drop-zone's idle accent tint. The one functional color in the system, everything actionable is this blue.
- **Workbench Blue, Deep** (`#0056b3`): hover state for the file-picker button only.

### Neutral
- **Panel White** (`#FFFFFFF2`, 95% opacity): the floating control panel's background, translucent enough to hint at the image behind it without ever obscuring the panel's own content.
- **Ink** (`#333333`): panel text and the drop-zone's "actively dragging" tint.
- **Page Gray** (`#F0F0F0`): the `<body>` background showing through before any texture is loaded.
- **Track Gray** (`#DDDDDD`): the slider track's resting color, underneath the rainbow fill.
- **Idle Scrim** (`#0000001A`, 10% black) / **Drag Scrim** (`#00000033`, 20% black): the full-bleed overlay tint over the loaded texture, idle vs. actively-dragging-a-file state.

### Utility
- **Inspection Guide Red** (`#FF000059`, 35% opacity): the optional grid-overlay lines marking tile boundaries. Deliberately outside the brand palette; a guide line needs to read against *any* uploaded texture, not just this system's own neutrals, the same reason design tools (Photoshop guides, CAD grids) reach for a saturated red/cyan regardless of their own chrome color.

### Named Rules
**The One Motif Rule.** The rainbow gradient (red through violet) appears in exactly two places: the animated title text and the slider's fill track. It never becomes a general accent, a border, or a background elsewhere. Its rarity is what makes it read as a signature rather than a theme.

## 3. Typography

**Display Font:** Comic Sans MS (with Comic Sans, cursive fallback)
**Body Font:** Georgia (with Calibri, serif fallback)

**Character:** Comic Sans carries the single title and only the title, an unapologetic, personal signature. Georgia is the quiet base font-family declared on `html`/`body`, present but nearly invisible since the interface has almost no body copy, the image is the content.

### Hierarchy
- **Display** (Comic Sans, 400, 1.1em): the one title, "Seamless Texture Checker by Claudio". Rainbow gradient fill via `background-clip: text`, floating and rotating hue continuously.
- **Body** (Georgia, 400, 1em): base document font-family; in practice the interface carries almost no prose, so this is a quiet fallback rather than a visible hierarchy step.

### Named Rules
**The Single Signature Rule.** Comic Sans is used in exactly one place, the title. Every other piece of UI (button label via icon, slider) carries no typeface personality of its own, so the one Comic Sans moment doesn't get diluted by repetition.

## 4. Elevation

Unlike a typical flat product UI, the control panel is elevated at rest, not just on hover, because it's meant to feel like a real object resting on top of the canvas rather than a flat overlay. Depth increases further in response to interaction (dragging a file over the drop zone).

### Shadow Vocabulary
- **Resting Lift** (`box-shadow: 0 4px 15px rgba(0,0,0,0.1)`): the panel's default, always-on shadow.
- **Drag Lift** (`box-shadow: 0 6px 20px rgba(0,0,0,0.15)`), paired with a `scale(1.02)` and `translateX(-50%)`: the panel's response while a file is being dragged over the page.

### Named Rules
**The Always-Floating Rule.** The panel never sits flat. It has a resting shadow and gains more of it on interaction, reinforcing the "physical object on a workbench" metaphor. This is the opposite of a flat-by-default product surface, and that's deliberate here.

## 5. Components

Tactile and toy-like: every interactive element responds to touch with scale, color, or motion, never a static swap.

### Buttons
- **Shape:** `rounded-lg` (15px), fully rounded pill-like label for the file picker
- **Primary (file picker):** Workbench Blue background, white icon, `6px 12px` padding, gentle continuous pulse (`scale(1) → 1.05 → 1`) so it reads as inviting even at rest
- **Hover:** background shifts to Workbench Blue Deep, lifts `-2px` on the Y axis

### Panel (Signature Component)
The floating control panel is the system's one structural container: title + file input on one row, size slider below, all inside a `15px`-radius white panel that never sits flush with the edges of the screen (top: 20px, centered horizontally). It carries the Resting Lift shadow always, gains the Drag Lift + subtle scale on drag-over, and has a subtle continuous `float` animation on its own pseudo-element border layer.

### Inputs (Slider)
- **Style:** custom-rendered range input, `8px` tall, `5px` radius, Track Gray at rest
- **Fill:** the one other rainbow-gradient surface in the system, animated as a slow-scrolling `linear-gradient`, representing how far into the size range the user has dragged
- **Thumb:** `20px` circle, Workbench Blue, scales to `1.1` on hover with a matching blue glow shadow

## 6. Do's and Don'ts

### Do:
- **Do** keep the rainbow gradient to exactly the title and the slider fill; it's a signature, not a general accent.
- **Do** keep the control panel visually elevated at rest (a real shadow), it's meant to feel like an object, not a flat overlay.
- **Do** keep Comic Sans confined to the title only.
- **Do** provide a `prefers-reduced-motion` fallback for every animation (float, pulse, bounce-in, rainbow, rainbow-track): a static end-state, not motion removed with nothing in its place.
- **Do** let the tiled texture underneath stay completely full-bleed and undecorated; the panel is a guest on top of it, not a frame around it.

### Don't:
- **Don't** turn this into a generic SaaS utility page: no card grids, no dashboard sidebar, no gradient hero panel.
- **Don't** make it sterile or corporate-gray either; the playful personality is the point, not a bug to be professionalized away.
- **Don't** add friction (extra clicks, confirmation modals, multi-step flows) between "I have an image" and "I can see it tiled". Drop, paste, and file-pick must all land on the same instant result.
- **Don't** spend the rainbow motif anywhere else in the UI; a rainbow border, a rainbow button, or a rainbow background would dilute the one-motif rule into generic decoration.
