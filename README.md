# Controlling Browser Chrome

How to make Safari's status bar and toolbars take on your page's colors as you scroll, in HTML and
CSS only. No JavaScript anywhere. Open it on an iPhone, a Mac, and inside X's in-app browser; the
three behave differently and this README says how.

**Live demo:** <https://thenotpit.github.io/viewport-fit-demo/>

Three pages, switched from the menu:

- **Viewport Cover** (`index.html`): `viewport-fit=cover`. The page runs edge to edge behind the
  bars, and its background color animates with the scroll, so the bars are always the page's color.
- **Color Strip Hack** (`viewport-auto.html`): the default viewport, so the page stops short of the
  bars. Two faint fixed strips at the screen's edges carry the page's color, and Safari paints the
  bars from them.
- **Variable Strips** (`gradient.html`): the strips again, on a gradient. The top strip follows the
  gradient's color at the top of the screen, the bottom strip its color at the bottom.

## The basics

**`viewport-fit=cover`** lets the page lay out under the notch, the home indicator and the bars,
and makes `env(safe-area-inset-*)` real, so pinned things can stay clear of them.

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

**Put the color on `html` and animate it with the scroll.** Where Safari needs a single color,
behind the status bar and past either end of the page, it reads `html`'s `background-color`. A
scroll-driven animation keeps that color matching whichever end you are at, and the browser runs it
itself, so nothing waits on a script.

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

**Why CSS and not a scroll listener.** On iPhone, scrolling runs on its own thread, and scroll
events reach JavaScript after the page has already moved. A script that sets a color on each event
is always a frame or more behind the content, every event costs a style recalculation and a repaint,
and any hiccup on the main thread shows up as a stutter in the bars. A scroll-driven animation is
tied to the scroll position by the browser itself: no event, no script, nothing to fall behind. In other words, don't use JS for this, CSS only.

**Don't pin a background.** A `position: fixed` layer touching the screen's edges makes Safari fill
the bars with a flat color and the page stops showing through. That includes layers you didn't add:
a popover's `::backdrop` did it here until it was switched off with `display: none`.

## Color strips

Without `viewport-fit=cover`, Safari paints its bars from a fixed element touching the screen's
edge. It reads the element's `background-color` value, not its pixels, and reads it again every time
the element repaints, so a strip animating its color on the scroll gives you live bars. WebKit looks
at one point: the horizontal center of the edge, 4px in. Whatever is topmost there is what it reads.

The strips here are 20px tall, at 10% opacity, with a gradient mask fading the inner edge, and they
run the same keyframes as `html`. What a strip can be, found by changing one thing at a time:

**Safari on iPhone**

- At least 11px tall. WebKit's rule is "more than 10"; 12px is the smallest we tried.
- Can be 10% opacity. Below 0.1 it is skipped.
- Can be blurred or masked.
- Can bleed past the screen's edge.
- Can sit under other elements, with a catch: if something else is on top at the center of the
  edge, Safari uses `html`'s background color for that bar instead of the strip. On a solid page
  the two match and you won't notice. On the gradient page they don't, so the strips sit above the
  header on phones.

**Safari on the Mac and the iPad**

- Reads a fixed edge once, before the first scroll, and holds it. A strip it can see locks the tab
  bar to the first color. WebKit files the iPad with the Mac here, not the iPhone, and an iPad
  confirmed it: `viewport-fit=cover` works, and strips it can see lock the bar.
- Keep the strips under something at the top edge, and the Mac and iPad ignore them and follow
  `html`'s background color instead, live. The strips go above the header only on phones: touch
  devices whose shorter side is under 500px.

**X's in-app browser**

- At least 20px tall. 19px is not read.
- Can be 10% opacity, blurred, or masked.
- Cannot bleed past the screen's edge, however much of it is on screen. That is also why the soft
  edge is a mask and not a blur: a blur fades at the screen's edge too, and fixing that needs
  overhang.
- Updates only once the scroll comes to rest. WebKit sends the new color every frame; the in-app
  browser applies it when scrolling settles.
- A `theme-color` meta tag, even one kept in sync by script, does nothing there.

**Notes**

- Strips on a `viewport-fit=cover` page make Safari fill the bars flat in the strips' color, hiding
  the page behind them. It looks like the default viewport. So cover and strips don't mix: cover
  looks best in Safari and leaves X with a solid bar; strips work everywhere but the page no longer
  shows through the bars.
- How a strip ends up on top or underneath makes no difference. Explicit `z-index` and plain DOM
  order behave the same in all three browsers.
- When testing in X, load the page with a fresh query string each time. It keeps its own cache, and
  a stale copy looks exactly like a failed test.

## Variable strips

A strip painted with the gradient would be sampled once and held, since Safari reads a color value.
So each strip animates its `background-color` through the gradient's stops, placed where its own
edge of the screen meets them during the scroll. The page is 520 screen heights tall in `lvh`, the
same unit as the gradient's first stop, and the scroll covers 420 of them, so a position on the page
divided by 420 is its place in the scroll. The top strip starts on the blue, meets the pink at
23.81%, the night's first color at 92.86%, and ends partway into the last color. The bottom strip
starts one screen down, already on the pink, meets the night's first color at 69.05%, and ends on
the last color. Gradients and animations both interpolate in sRGB, so the strips match the page.
Write the last stop as a plain color: a `color-mix()` stop makes the animation interpolate in oklab
and drift.

Desktop Safari tints from `html`, so on this page `html` runs the top strip's keyframes too.

**Some colors flip the chrome to dark text.** Safari switches a tinted bar to dark text and icons
once white text would have less than 4.5:1 contrast against the tint, as far as we can tell: the
blue passes at 4.50:1, a pink at 4.26:1 flipped it, and the gradient's pink is at 2.17:1. The rule is
Safari's, not WebKit's, so this is inferred from what passed and what didn't. On the gradient page
the pink stop in the keyframes is a custom property: the strips keep the gradient's pink for the
phone, and `html` takes a darker pink, `#964f71` at 5.75:1, for the Mac.
