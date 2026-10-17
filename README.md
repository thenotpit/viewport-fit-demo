# viewport-fit=cover demo

One static page showing how to draw a page edge to edge on iPhone, behind Safari's toolbars, in
HTML and CSS alone, with no scripts sampling or animating the toolbar's color.

Open it on an iPhone and scroll: the sky runs behind the status bar and the toolbar, from day at the
top of the page to night at the bottom.

There are three pages, switched from the menu:

- **Viewport Cover - No Strips** (`index.html`): `viewport-fit=cover`. The page draws edge to edge behind
  Safari's bars, one solid color fading from blue to night as you scroll, so the page and the bars
  are always exactly the same color.
- **Viewport Auto - Strips** (`viewport-auto.html`): the same page with the default viewport. It is
  letterboxed inside the status bar and the toolbars, and Safari paints them from two faint fixed
  strips at the screen's edges, which carry the same color.
- **Gradient** (`gradient.html`): `viewport-fit=cover` with a day gradient on the page, over the
  solid color that follows the scroll.

Everything they share is in `styles.css`; each page sets only its own background on `html`.

## How it works

1. **`viewport-fit=cover`** in the viewport meta tag lets the page lay out into the notch and
   home-indicator areas, instead of being letterboxed inside them, and makes the
   `env(safe-area-inset-*)` values real.

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
   ```

2. **The sky is a gradient on `html`, over a solid color.** **`lvh`** (the screen with the toolbars
   hidden) sets where day ends, so the first screen is the full day gradient.

   ```css
   html {
     background: #2f6bff linear-gradient(#2f6bff, #ff8ac2 100lvh, #120a2a 75%, #2b1550) no-repeat;
   }
   ```

3. **The solid color follows the scroll**, from the blue at the top of the page to the night at the
   bottom, with a CSS scroll-driven animation.

   ```css
   html {
     animation: page-color linear both;
     animation-timeline: scroll(root);
   }

   @keyframes page-color {
     from { background-color: #2f6bff; }
     to   { background-color: #2b1550; }
   }
   ```

   The gradient page runs a different set of keyframes on `html`, the same ones as its top strip:
   the gradient's colors, placed where the top of the screen meets them during the scroll. Desktop
   Safari tints its tab bar from `html`'s color rather than from the strips, and the shared
   blue-to-night keyframes would skip the pink. The one cost is past the bottom of the page, where
   Safari paints `html`'s color while rubber-banding: the top edge's final color, a shade lighter
   than the gradient's last color, since the top of the screen never reaches the bottom of the page.

4. **`env(safe-area-inset-*)`** keeps the pinned header's icons clear of the notch and rounded
   corners, now that the page reaches them.

## Notes

### Don't pin the background

A `position: fixed` background layer, one that stays put while the page scrolls, turns Safari's bars
solid. Safari tints its toolbars from fixed elements touching the screen's edges, and a full-screen
fixed layer touches both. The page then stops showing behind the bars. That same heuristic is what
"color strips at the top and bottom" tricks rely on.

So the background is part of the page and scrolls with it. Small fixed elements without a background
of their own, like the header here, didn't cause it.

Watch for full-screen layers you didn't add yourself. A popover comes with a `::backdrop`, a
transparent layer pinned over the whole screen, and opening the menu panel turned the bottom bar
solid until the backdrop was switched off with `display: none`. A scrim drawn as a huge shadow on the
panel instead avoided the solid bars but couldn't dim the strip behind the status bar, which is
Safari's own color, so the panel opens without one.

### Safari reads a color, not the gradient

Where Safari needs one color (the strip behind the status bar, and the area past either end of the
page when you pull beyond it), it uses the page's solid `background-color`. It ignores the gradient,
and with no solid color set it falls back to white.

### Move that color with the scroll

A page has only one background color, and Safari shows it past both ends. A fixed blue matches the
top of the page but shows blue past the bottom, where the page is night. Animating the color against
the scroll position keeps it matching whichever end you're at. The browser drives a scroll-driven
animation itself, so no script runs on scroll. Browsers without scroll timelines keep the starting
blue.

### `html` or `body`

The **gradient** works on either. When `html` has no background, browsers paint `body`'s across the
whole page instead, and the two look the same. This demo puts it on `html`, which was tested on an
iPhone.

The **solid color** is safest on `html`. Once `html` has any background, `body`'s background only
covers the body's own box and is no longer the page's color. This demo hasn't tested a color on
`body` alone.

### Why it's smoother than the strip hack

It comes down to what has to happen on each frame while you scroll.

**The strip hack** runs a chain for every scroll update:

1. Safari scrolls the page, then tells JavaScript that a scroll happened.
2. The script works out a color and changes the strips' style.
3. The browser recalculates styles and repaints the strips.
4. Safari notices the strips changed and re-tints its toolbars, which is separate interface drawing
   outside the page.

On iPhone, scrolling happens on its own fast path, and scroll events reach JavaScript slightly after
the page has already moved. So the strips are always a frame or more behind the content, and any
hiccup on the main thread shows up as stutter.

**This page does far less per frame:**

- **The gradient** is part of the page. Scrolling moves pixels that are already drawn.
- **The hexagon** is fixed, so the browser keeps it on its own layer and leaves it in place while the
  page moves under it.
- **The background color** is the only thing that changes with the scroll, and a scroll-driven
  animation ties it straight to the scroll position. No script runs, so it can't fall behind.

The color isn't completely free: changing a background color still means repainting, though it's a
cheap flat fill. The real gain is that nothing waits on JavaScript and nothing fights Safari's
toolbar logic. The strip hack fakes the effect from outside the page; this is the page.

### No JavaScript at all

The page has no scripts. The menu button opens its panel with the HTML popover API
(`popovertarget`), which also handles closing it on an outside tap or Escape, and the panel's
animation is a CSS transition with `@starting-style`.

### The gradient page's strips

A strip is read as a color value, so a strip carrying the gradient itself would be sampled once and
held. Instead each strip on the gradient page animates its `background-color` through the
gradient's stops, placed where its own edge of the screen meets them. The page is 520 screen
heights tall, in `lvh` like the gradient's first stop, and the scroll covers 420 of them. The top
edge starts on the blue, meets the pink at one screen height, 23.81% of the scroll, the night's top
at 75% of the page, 92.86%, and ends 23.08% of the way from there to the night's bottom. The bottom
edge starts one screen down, already on the pink, meets the night's top at 69.05%, and ends on the
night's bottom. Gradients and animations both interpolate linearly in sRGB, so the strips match the
page within rounding, with one caveat: the last stop is written as a plain color, because a
`color-mix()` stop makes the animation interpolate in oklab and drift off the gradient. On a phone
the bottom edge sits a toolbar's height higher while the toolbar is showing, so the bottom strip
runs slightly ahead until the toolbar collapses.

### Pink and the bars' text

Safari switches a tinted bar to dark text and icons once the tint is bright enough. The rule is in
Safari rather than WebKit, and a simple brightness cut doesn't fit what we saw: a pink with a
greyscale value of 0.48 flipped the bar while the blue at 0.47 did not. White text's contrast
against the tint does fit, with 4.5:1 as the line: the blue gives 4.51, that pink 4.25, the
gradient's pink 2.17. On the gradient page the pink stop in `html`'s keyframes is therefore a
custom property: the top strip keeps the gradient's pink, and `html`, which is what desktop
Safari tints from, takes `#964f71`, a contrast of 5.7. The phone's bars come from the strips
and keep the brighter pink.

### The edge strips

Without `viewport-fit=cover`, the page stops short of the bars and Safari paints them itself. It
reads a fixed element touching a screen edge, and reads it again whenever that element repaints,
so the strips on the auto page run the same scroll-driven keyframes as `html` and the bars follow
the scroll live. It reads the element's color value, not its pixels: the strips are at 10%
opacity, the floor for a candidate, and a gradient mask fades their inner edge.

What a strip can be, measured on an iPhone one change at a time. Safari's bars and X's in-app
browser read the strips by the same mechanism but not by the same rules:

| | Safari | X's in-app browser |
|---|---|---|
| Height | 12px read; the WebKit source says more than 10px | 20px read, 19px not |
| 10% opacity | Read | Read |
| Blur | Read | Read |
| Gradient mask | Read | Read |
| Box past the screen's edge | Read | Not read, however much is on screen |
| Under the header | Not read: the top bar falls back to `html`'s color | Not read: the bar falls back to `html`'s color |
| Viewport | Read under cover and auto | Read under cover and auto |

Safari reads whatever is topmost at the center of the edge. The header covers that point at the
top, and a header with no color makes Safari fall back to `html`'s background color for the top
bar. On a solid page that is the strip's color anyway, which is why strips under the header looked
fine there; the gradient page, where the top strip and `html` differ, showed the top bar shadowing
`html` until the strips went above the header. Nothing covers the bottom center, so the bottom
strip is read either way.

Desktop Safari runs the other way. On the Mac a fixed edge's color is read once, before the first
scroll, and then held, so when the strips are detected there they lock the tab tint to the first
blue. Under the header the Mac falls back to `html`'s color and follows the scroll. So the strips
sit above the header on touch devices only, through a `hover: none` media query, and stay under it
on the Mac. How an element ends up on top makes no difference: explicit z-index and plain DOM
order behave the same in all three browsers.

In-app browsers draw an opaque bar of their own above the page. That bar reads a color once at
load and then stops following `html`'s background. In X's in-app browser a `theme-color` meta tag
kept equal to `html`'s color by a script had no effect, and the strips do move the bar, but only
once the scroll comes to rest: WebKit delivers the strip's new color on every frame, and the
in-app browser applies it when scrolling settles. When testing there, load the page with a fresh
query string each time. The in-app browser keeps its own cache, and a stale copy looks exactly
like a failed test.

## What we saw on iPhone

| Page setup | Bars and the area past each end |
|---|---|
| Fixed full-screen sky layer | Solid bars in the fixed layer's color |
| Gradient only, no solid color (on `html` or `body`) | White |
| Gradient on `html` over solid blue | Blue at both ends |
| Gradient on `html`, solid color animated from blue to night | Blue at the top, night at the bottom |
| Cover page plus fixed strips at each edge | Bars filled flat in the strips' color, hiding the page behind them |
