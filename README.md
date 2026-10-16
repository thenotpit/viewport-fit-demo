# viewport-fit=cover demo

One static page showing how to draw a page edge to edge on iPhone, behind Safari's toolbars, with
no scripts sampling or animating the toolbar's colour.

Open it on an iPhone and scroll: the sky runs behind the status bar and the toolbar, from day at the
top of the page to night at the bottom.

## How it works

1. **`viewport-fit=cover`** in the viewport meta tag lets the page lay out into the notch and
   home-indicator areas, instead of being letterboxed inside them, and makes the
   `env(safe-area-inset-*)` values real.

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
   ```

2. **The sky is a gradient on `html`, over a solid colour.** **`lvh`** (the screen with the toolbars
   hidden) sets where day ends, so the first screen is the full day gradient.

   ```css
   html {
     background: #2f6bff linear-gradient(#2f6bff, #e6a3c3 100lvh, #120a2a 75%, #2b1550) no-repeat;
   }
   ```

3. **The solid colour follows the scroll**, from the blue at the top of the page to the night at the
   bottom, with a CSS scroll-driven animation.

   ```css
   html {
     animation: page-colour linear both;
     animation-timeline: scroll(root);
   }

   @keyframes page-colour {
     from { background-color: #2f6bff; }
     to   { background-color: #2b1550; }
   }
   ```

4. **`env(safe-area-inset-*)`** keeps the pinned header's icons clear of the notch and rounded
   corners, now that the page reaches them.

## Notes

### Don't pin the background

A `position: fixed` background layer, one that stays put while the page scrolls, turns Safari's bars
solid. Safari tints its toolbars from fixed elements touching the screen's edges, and a full-screen
fixed layer touches both. The page then stops showing behind the bars. That same heuristic is what
"colour strips at the top and bottom" tricks rely on.

So the background is part of the page and scrolls with it. Small fixed elements without a background
of their own, like the header here, didn't cause it.

### Safari reads a colour, not the gradient

Where Safari needs one colour (the strip behind the status bar, and the area past either end of the
page when you pull beyond it), it uses the page's solid `background-color`. It ignores the gradient,
and with no solid colour set it falls back to white.

### Move that colour with the scroll

A page has only one background colour, and Safari shows it past both ends. A fixed blue matches the
top of the page but shows blue past the bottom, where the page is night. Animating the colour against
the scroll position keeps it matching whichever end you're at. The browser drives a scroll-driven
animation itself, so no script runs on scroll. Browsers without scroll timelines keep the starting
blue.

### `html` or `body`

The **gradient** works on either. When `html` has no background, browsers paint `body`'s across the
whole page instead, and the two look the same. This demo puts it on `html`, which was tested on an
iPhone.

The **solid colour** is safest on `html`. Once `html` has any background, `body`'s background only
covers the body's own box and is no longer the page's colour. This demo hasn't tested a colour on
`body` alone.

## What we saw on iPhone

| Page setup | Bars and the area past each end |
|---|---|
| Fixed full-screen sky layer | Solid bars in the fixed layer's colour |
| Gradient only, no solid colour (on `html` or `body`) | White |
| Gradient on `html` over solid blue | Blue at both ends |
| Gradient on `html`, solid colour animated from blue to night | Blue at the top, night at the bottom |
