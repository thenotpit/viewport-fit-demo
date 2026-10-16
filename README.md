# viewport-fit=cover demo

One static page showing how to draw a page edge to edge on iPhone, behind Safari's toolbars, with
no scripts sampling or animating the toolbar's colour.

Open it on an iPhone and scroll: the sky runs behind the status bar and the toolbar, from day at the
top of the page to night at the bottom.

## How it works

1. **`viewport-fit=cover`** in the viewport meta tag lets the page draw into the notch and
   home-indicator areas and behind the toolbars, instead of being letterboxed inside them.

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
   ```

2. **The sky is the page's own background**, a gradient painted on `html`. **`lvh`** (the screen
   with the toolbars hidden) sets where day ends, so the first screen is the full day gradient.

   ```css
   html {
     background: linear-gradient(#2f6bff, #e6a3c3 100lvh, #120a2a 75%, #2b1550) no-repeat;
   }
   ```

3. **`env(safe-area-inset-*)`** keeps the pinned header's icons clear of the notch and rounded
   corners, now that the page reaches them.

## Why the background isn't `position: fixed`

Safari tints its toolbars from fixed elements touching the screen's edges. A fixed, full-screen
background layer counts, so Safari turns its bars solid in that layer's colour and the page stops
showing through. That heuristic is also what "colour strips at the top and bottom" tricks lean on.
With the background in the page instead, Safari shows the page itself behind its bars. The pinned
header has no background of its own.
