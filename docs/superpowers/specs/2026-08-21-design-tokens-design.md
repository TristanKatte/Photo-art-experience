# Design Tokens & Light/Dark Mode — Design

## Context

The user asked for a CSS file with colors and typography for a light/dark mode
toggle, based on values copied from the old WordPress site's Elementor kit
(`.elementor-kit-10`, blue/green Roboto palette). Investigation of the live
site (`https://photoartexperience.nl/`) showed those Elementor variables are
unused boilerplate — the homepage is actually built with a different theme
("novo") and different plugins, and its real, currently-rendered colors and
fonts are different from what was pasted. This spec is based on values pulled
directly from the live site's compiled CSS, not the Elementor kit.

Real findings from the live site:

- **Accent color:** `#c48f56` (caramel/tan) — confirmed via `novo/style.css`,
  used for nav hover/active states, heading highlights, and buttons. Matches
  the orange "The art of making memories" text visible in the homepage
  screenshot.
- **Accent hover (darker variant):** `#bc8a53`.
- **Body text:** `#161616`.
- **Dark band background** (footer, hero caption bar): `#232323`.
- **Typography:** `Montserrat` for both body (400) and headings (700) — not
  Roboto/Roboto Slab from the unused Elementor kit.
- **"Image protection"** on the old site is CSS-only (`user-select: none`,
  `touch-callout: none` via the NextGEN Gallery Plus plugin) — no real
  right-click/context-menu JS blocking found. Decided: **not** replicated on
  the new site (negligible real protection, meaningful UX cost for
  legitimate visitors).

## Goals

- A global CSS custom-property token system for color and typography, based
  on the real palette above.
- A working light/dark mode toggle: defaults to the visitor's OS preference,
  can be manually overridden, and the override persists across visits with no
  flash of the wrong theme on load.
- Migrate `HeroSlideshow.svelte`'s existing hardcoded colors to the new
  tokens, so the homepage is a working proof that the system works end to
  end.

## Non-goals

- Building out `Header.svelte` or any other stub component beyond placing the
  theme toggle somewhere visible. The toggle is a floating, fixed-position
  button for now; relocating it into a real header is future work.
- Replicating the old site's image-protection CSS.
- A three-way "light / dark / system" UI — the toggle is binary (switches to
  the theme opposite whatever is currently in effect), matching the common
  "system by default, one override" pattern.
- Building a full design-token scale beyond color + typography (no spacing/
  shadow/radius token system) — out of scope for what was asked.

## Token model

Two token groups:

1. **Theme-reactive tokens** — flip between light and dark:
   - `--color-bg`: `#ffffff` (light) / `#121212` (dark)
   - `--color-text`: `#161616` (light) / `#ededed` (dark)
   - `--color-border`: `#e2e2e2` (light) / `#333333` (dark)
2. **Brand-constant tokens** — the old site's "always-dark bar with light
   text" bands (hero caption, footer) are a brand element, not a
   content-polarity element, so these stay constant across both themes:
   - `--color-surface-inverted`: `#232323`
   - `--color-text-inverted`: `#f5f5f5`
3. **Accent** (effectively also brand-constant, same value in both themes):
   - `--color-accent`: `#c48f56`
   - `--color-accent-hover`: `#bc8a53`

Typography tokens:

- `--font-heading`: `'Montserrat', sans-serif`, weight `700`
- `--font-body`: `'Montserrat', sans-serif`, weight `400`

## Component architecture

### `src/lib/styles/tokens.css` (new)

Global stylesheet, imported once from `+layout.svelte`. Contains:

- The reset: `html, body { margin: 0; padding: 0; }` (moved here from
  `HeroSlideshow`-era `+layout.svelte` scoped style — genuinely global, not
  slideshow-specific).
- Base `body` styling: `font-family: var(--font-body)`, `font-weight: 400`,
  `background: var(--color-bg)`, `color: var(--color-text)`.
- Light tokens declared on bare `:root`.
- Dark overrides in two places, per standard practice for "system default,
  explicit override wins":
  - `@media (prefers-color-scheme: dark)` guarded by
    `:root:not([data-theme="light"])` — applies when the OS prefers dark and
    the visitor hasn't explicitly chosen light.
  - `:root[data-theme="dark"]` — applies whenever the visitor has explicitly
    chosen dark, regardless of OS preference.
- Brand-constant and accent tokens declared once on bare `:root` (no
  dark-mode override block needed, since they don't change).

### `src/app.html`

- Adds `<link rel="preconnect" href="https://fonts.googleapis.com">`,
  `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>`, and
  the Montserrat stylesheet link (weights 400/700) to `<head>`.
- Adds a small inline `<script>` right before `%sveltekit.body%` (runs
  synchronously during initial HTML parse, before first paint):
  ```html
  <script>
  	(function () {
  		try {
  			var stored = localStorage.getItem('theme');
  			if (stored === 'light' || stored === 'dark') {
  				document.documentElement.setAttribute('data-theme', stored);
  			}
  		} catch (e) {}
  	})();
  </script>
  ```
  If nothing is stored, no attribute is set and `tokens.css`'s
  `prefers-color-scheme` media query alone determines the initial theme — no
  flash, no JS dependency for first-time visitors.

### `src/lib/components/atoms/ThemeToggle.svelte` (new)

- On mount, determines the currently effective theme: reads
  `document.documentElement.dataset.theme` if set, otherwise falls back to
  `matchMedia('(prefers-color-scheme: dark)').matches`.
- Renders a single button labeled/aria-labeled for the action it performs
  ("Switch to dark mode" / "Switch to light mode" depending on current
  state).
- On click: flips the effective theme, sets
  `document.documentElement.setAttribute('data-theme', next)`, and writes
  `localStorage.setItem('theme', next)`.
- Also listens for `matchMedia('(prefers-color-scheme: dark)')` `change`
  events so the button's label stays in sync if the visitor changes their OS
  setting while no explicit override is stored.

### `src/routes/+layout.svelte`

- Imports `tokens.css` (a plain `import '$lib/styles/tokens.css';` at the top
  of the `<script>` block — SvelteKit treats this as global CSS).
- Removes the scoped `html, body` reset added during the HeroSlideshow
  feature (superseded by `tokens.css`).
- Renders `<ThemeToggle />` as a `position: fixed` element (e.g. top-right
  corner) so it's visible on every page without needing `Header.svelte`
  built out first.

### `src/lib/components/organisms/HeroSlideshow.svelte`

Hardcoded colors replaced with token references:

- `#f5efe6` (light text on dark bands) → `var(--color-text-inverted)`
- `#c98a4b` (eyebrow accent) → `var(--color-accent)`
- `#0d0d0d` (caption band background) → `var(--color-surface-inverted)`
- `#1a1a1a` (image stage background, shown briefly while images load) →
  `var(--color-surface-inverted)`

No structural changes to `HeroSlideshow.svelte` — this is a find-and-replace
of color literals with `var(--token)` references, verified visually in both
light and dark mode afterward.

## Testing

- Manual verification via `npm run dev`: toggle switches themes instantly
  with no flash on reload; reloading after toggling remembers the choice;
  visiting with no stored preference follows the OS light/dark setting;
  changing OS preference live updates the toggle's label when no override is
  stored; HeroSlideshow renders correctly (readable text, correct accent) in
  both themes.
- `svelte-check` / `npm run lint` pass with no new errors.
- No automated tests — this is a CSS/visual feature with no meaningful
  non-visual logic to unit test.
