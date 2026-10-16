# viewport-fit=cover demo

One static page showing how to draw a page edge to edge on iPhone, behind Safari's toolbars, with
no scripts sampling or animating the toolbar's colour.

Open it on an iPhone and scroll: the sky runs behind the toolbars, and fades from day to night as
you go.

## How it works

1. **`viewport-fit=cover`** in the viewport meta tag lets the page draw into the notch and
   home-indicator areas and behind the toolbars, instead of being letterboxed inside them.

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
   ```

2. **`lvh`** (large viewport height: the screen with the toolbars hidden) sizes the fixed sky, so it
   reaches the bottom of the screen whatever the toolbars are doing. A small overscan past `lvh`
   carries it on under iOS's floating bottom bar.

   ```css
   .sky {
     position: fixed;
     top: 0;
     height: calc(100lvh + 6rem);
   }
   ```

3. **`env(safe-area-inset-*)`** keeps the header's icons clear of the notch and rounded corners,
   now that the page reaches them.

The day-to-night fade is a CSS scroll-driven animation (`animation-timeline: scroll(root)`), so the
browser runs it alongside scrolling. Browsers without scroll timelines get the same fade from a few
lines of script.
