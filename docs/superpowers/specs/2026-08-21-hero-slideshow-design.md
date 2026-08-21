# Hero Slideshow — Design

## Context

The homepage on the old WordPress site (Photo & Art Experience) has a full-bleed
image slider at the top: slides crossfade into each other, each image has a slow
"Ken Burns" zoom-out while it's active, and a slide is visible for roughly 5
seconds before the next one takes over. Below the image area sits a caption band
("The art of making memories" / "Photo & Art Experience"), and the image area
itself carries prev/next arrows and a numbered slide indicator (`01–05`) with a
progress line under the active number.

This project is a from-scratch SvelteKit 5 rebuild (atomic-design component
layout under `src/lib/components/{atoms,molecules,organisms,templates}`). Two
relevant files already exist as empty stubs:

- `src/lib/components/organisms/HeroSlideshow.svelte`
- `src/lib/components/molecules/SlideshowSlide.svelte`

This spec defines what goes in them.

## Goals

- Reproduce the full hero section from the screenshot: crossfading, slowly
  zooming-out slides; prev/next arrows; clickable numbered indicator with
  progress line; fixed caption band underneath the image area.
- Use placeholder images for now — real photos are swapped in later.
- No new dependencies (no carousel library).

## Non-goals

- The top navigation bar and the black social/copyright footer strip visible in
  the screenshot belong to `Header.svelte` / `Footer.svelte`, not this
  component. Not built here.
- Per-slide captions — the caption text is fixed regardless of which slide is
  showing.
- Building out `atoms/Image.svelte` — `SlideshowSlide` uses a plain `<img>`.
- Pixel-matching the 3440px screenshot exactly; responsive behavior uses
  relative units and flexbox rather than fixed breakpoints tuned to that one
  screenshot.

## Component architecture

### `HeroSlideshow.svelte` (organism)

Owns all state and behavior:

- `slides`: an array of `{ src, alt }` placeholder objects, defined locally in
  this component for now (swappable for a prop later once real photos exist).
- `currentIndex` (`$state`): index of the active slide.
- `activationCounter` (`$state`): increments every time a slide becomes active
  (including re-activation of the same index on a later loop). Passed to
  `SlideshowSlide` as the animation-restart key.
- `paused` (`$state`): true while hovered or while a control has keyboard
  focus; the autoplay effect checks this before arming the next timeout.
- Autoplay: a Svelte `$effect` arms a `setTimeout` (not `setInterval`) for
  ~5000ms that advances `currentIndex` (wrapping) and bumps
  `activationCounter`; the effect re-arms itself after each advance as long as
  `paused` is false. Manual navigation (arrow click or number click) updates
  `currentIndex`/`activationCounter` directly and the timer effect picks up
  from there, effectively restarting the 5s countdown.
- Renders: the stack of `SlideshowSlide`s, prev/next arrow buttons, the
  numbered indicator row with progress line, and the caption band markup.

### `SlideshowSlide.svelte` (molecule)

Presentational only — no internal state, no timers:

- Props: `src`, `alt`, `active` (boolean), `activationKey` (number).
- Renders one absolutely-positioned, full-bleed `<img>` (`object-fit: cover`).
- `opacity` transitions via CSS based on `active` (crossfade, ~1.2s).
- The Ken Burns zoom is a CSS keyframe animation
  (`scale(1.15) → scale(1)`, ~5s, `ease-out`) applied to the image. It's
  wrapped in `{#key activationKey}` so remounting on each activation restarts
  the keyframe animation from the beginning — no manual reflow hacks.

## Animation & timing details

- Crossfade duration: ~1.2s opacity transition.
- Zoom duration: ~5s, matching the full dwell time, so the zoom completes just
  as the next crossfade begins.
- Only the active and immediately-previous slide need non-zero opacity /
  raised `z-index` at any moment; all others sit at `opacity: 0`.
- Progress line under the active number animates its width/transform over the
  same 5s dwell, keyed the same way as the zoom so it restarts per activation.
  It's purely decorative — not functionally coupled to the timer beyond
  starting together.

## Controls

- **Arrows** (‹ ›): positioned at the vertical center of the left/right edges
  of the image area. Click → previous/next index, wrapping at the ends, resets
  the timer.
- **Numbered indicator** (`01 02 03 04 05`): bottom-right over the image area,
  matching the screenshot. Each number is a clickable button that jumps
  straight to that slide and resets the timer.
- **Pause on hover**: hovering anywhere over the slideshow sets `paused =
  true`; leaving resumes autoplay. Keyboard focus on any control within the
  slideshow behaves the same as hover (pauses autoplay while focused).

## Caption band

A fixed dark bar below the image area (not overlaid on the image, matching the
screenshot) containing the static eyebrow text "The art of making memories"
and title "Photo & Art Experience". Since this content doesn't vary per slide,
it's plain markup in `HeroSlideshow.svelte`, not data-driven.

## Styling & responsiveness

- Image area: full-bleed, roughly full-viewport-height on desktop.
- Flexbox + relative units throughout; arrows and indicator scale down and the
  caption band stacks normally on narrow viewports.
- No pixel-perfect matching to the 3440px screenshot.

## Accessibility

- Arrow buttons and number buttons get `aria-label`s (e.g. "Previous slide",
  "Go to slide 3").
- Autoplay pauses on hover and on keyboard focus within the slideshow's
  controls, resuming when both hover and focus are gone.
- Images get real `alt` text (placeholder alt text for now, matching whatever
  the placeholder images represent).

## Testing

- Manual verification via `npm run dev`: slides autoplay every ~5s, crossfade
  and zoom look right, arrows/numbers navigate and reset the timer, hover
  pauses/resumes, keyboard focus pauses.
- `svelte-check` / `npm run lint` pass with no new errors.
- No dedicated unit tests planned — this is a visual/animation-heavy component
  with no meaningful non-visual logic to unit test beyond the index-wrapping
  math, which is simple enough to verify by inspection and manual testing.
