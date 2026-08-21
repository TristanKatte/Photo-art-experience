# Design Tokens & Light/Dark Mode Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a global CSS design-token system (color + typography) based on the real photoartexperience.nl palette, add a working light/dark mode toggle with system-preference default and persisted override, and migrate `HeroSlideshow.svelte`'s hardcoded colors onto the new tokens.

**Architecture:** A new global stylesheet (`src/lib/styles/tokens.css`) defines CSS custom properties — light values on bare `:root`, dark overrides under both a `prefers-color-scheme` media query and an explicit `data-theme="dark"` attribute. `app.html` loads the Montserrat font and runs a tiny inline script that stamps `data-theme` from `localStorage` before first paint (only when a preference was explicitly stored, so first-time visitors get zero-JS system-preference theming with no flash). A new `ThemeToggle.svelte` atom lets visitors flip the theme, persisting their choice. `HeroSlideshow.svelte`'s hardcoded hex colors are replaced with token references.

**Tech Stack:** SvelteKit 5 (runes mode), plain CSS custom properties, Google Fonts (Montserrat, via `<link>` tags — no npm package), no new dependencies.

## Global Constraints

- Token values (exact, from the live site's compiled CSS):
  - `--color-bg`: `#ffffff` (light) / `#121212` (dark)
  - `--color-text`: `#161616` (light) / `#ededed` (dark)
  - `--color-border`: `#e2e2e2` (light) / `#333333` (dark)
  - `--color-surface-inverted`: `#232323` (constant across both themes)
  - `--color-text-inverted`: `#f5f5f5` (constant across both themes)
  - `--color-accent`: `#c48f56` (constant)
  - `--color-accent-hover`: `#bc8a53` (constant)
  - `--font-heading` / `--font-body`: `'Montserrat', sans-serif`, weight `700` / `400`
- No new npm dependencies.
- No image-protection CSS (`user-select: none` etc.) — explicitly decided against.
- The toggle is binary (light ⇄ dark) — no three-way "system" option in the UI. It reflects system preference until the visitor explicitly overrides it.
- The toggle renders as a `position: fixed` button from `+layout.svelte` — `Header.svelte` stays untouched (still an empty stub, out of scope).
- Formatting: tabs, single quotes, no trailing commas (project `prettier.config.js`) — match this in all new/modified files.
- `npm run lint` (prettier + eslint) and `npm run check` (svelte-check) must pass with 0 errors and 0 warnings after every task.

---

### Task 1: `tokens.css` — token system, reset, and layout wiring

**Files:**

- Create: `src/lib/styles/tokens.css`
- Modify: `src/routes/+layout.svelte`

**Interfaces:**

- Produces: CSS custom properties (`--color-bg`, `--color-text`, `--color-border`, `--color-surface-inverted`, `--color-text-inverted`, `--color-accent`, `--color-accent-hover`, `--font-heading`, `--font-heading-weight`, `--font-body`, `--font-body-weight`) available globally on `:root` from this point on. Task 3 and Task 4 reference these exact custom property names.

- [ ] **Step 1: Write `tokens.css`**

`src/lib/styles/tokens.css`:

```css
:root {
	--color-bg: #ffffff;
	--color-text: #161616;
	--color-border: #e2e2e2;

	--color-surface-inverted: #232323;
	--color-text-inverted: #f5f5f5;

	--color-accent: #c48f56;
	--color-accent-hover: #bc8a53;

	--font-heading: 'Montserrat', sans-serif;
	--font-heading-weight: 700;
	--font-body: 'Montserrat', sans-serif;
	--font-body-weight: 400;
}

@media (prefers-color-scheme: dark) {
	:root:not([data-theme='light']) {
		--color-bg: #121212;
		--color-text: #ededed;
		--color-border: #333333;
	}
}

:root[data-theme='dark'] {
	--color-bg: #121212;
	--color-text: #ededed;
	--color-border: #333333;
}

html,
body {
	margin: 0;
	padding: 0;
}

body {
	background: var(--color-bg);
	color: var(--color-text);
	font-family: var(--font-body);
	font-weight: var(--font-body-weight);
}
```

- [ ] **Step 2: Import it globally and remove the now-redundant scoped reset**

Read the current `src/routes/+layout.svelte` first. Replace its contents with:

```svelte
<script lang="ts">
	import '$lib/styles/tokens.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

{@render children()}
```

(This drops the `<style>` block with the scoped `html, body { margin: 0; padding: 0; }` reset that was added during the HeroSlideshow feature — `tokens.css` now provides that reset globally instead.)

- [ ] **Step 3: Verify visually, including dark-mode override**

Run: `npm run dev`, open `http://localhost:5173/`.

Check:

- No white gutter around the page content (reset still works).
- Open browser DevTools → Rendering tab → "Emulate CSS media feature prefers-color-scheme" → set to "dark". Confirm the page background switches to near-black (`#121212`) and text becomes light — this proves the media-query dark override works, before any toggle UI exists. Set emulation back to "light" (or "No emulation") afterward.
- In the DevTools console, run `getComputedStyle(document.body).fontFamily` — should include `"Montserrat"` in the value even though the font file itself isn't loaded yet (that's Task 2); the CSS property is already wired.

- [ ] **Step 4: Run lint and type checks**

Run: `npm run lint && npm run check`

Expected: 0 errors, 0 warnings.

- [ ] **Step 5: Commit**

```bash
git add src/lib/styles/tokens.css src/routes/+layout.svelte
git commit -m "Add design token system with light/dark CSS custom properties"
```

---

### Task 2: Montserrat font + pre-hydration theme script

**Files:**

- Modify: `src/app.html`

**Interfaces:**

- Consumes: `tokens.css`'s dark-mode `:root[data-theme="dark"]` override (Task 1) — this task's script sets that attribute.
- Produces: the `data-theme` attribute on `<html>`, stamped synchronously before first paint whenever `localStorage.getItem('theme')` is `'light'` or `'dark'`. Task 3's `ThemeToggle.svelte` writes to that same `localStorage` key (`'theme'`) and reads/sets the same `data-theme` attribute.

- [ ] **Step 1: Add font links and the inline theme script**

Read the current `src/app.html` first. Replace its contents with:

```html
<!doctype html>
<html lang="en">
	<head>
		<meta charset="utf-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1" />
		<meta name="text-scale" content="scale" />
		<link rel="preconnect" href="https://fonts.googleapis.com" />
		<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
		<link
			href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap"
			rel="stylesheet"
		/>
		%sveltekit.head%
	</head>
	<body data-sveltekit-preload-data="hover">
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
		<div style="display: contents">%sveltekit.body%</div>
	</body>
</html>
```

- [ ] **Step 2: Verify the font loads**

Run: `npm run dev`, open `http://localhost:5173/`, open DevTools → Network tab, filter by "Font". Reload.

Expected: requests to `fonts.gstatic.com` for Montserrat succeed (status 200). In the DevTools console, run `document.fonts.check("16px Montserrat")` — expect `true` once fonts have loaded (may need to await `document.fonts.ready` first: `await document.fonts.ready; document.fonts.check("16px Montserrat")`).

- [ ] **Step 3: Verify the pre-hydration script (no flash of wrong theme)**

In the DevTools console:

1. Run `localStorage.setItem('theme', 'dark')`, then reload the page. Immediately after reload (check as early as possible — e.g. via "View Page Source" showing the `<script>` runs before `%sveltekit.body%`, or by observing the page paints dark from the very first frame with no light-then-dark flash), confirm `document.documentElement.getAttribute('data-theme') === 'dark'` and the page is rendered in the dark palette from first paint.
2. Run `localStorage.removeItem('theme')`, reload again. Confirm `document.documentElement.getAttribute('data-theme')` is `null`, and the page follows whatever `prefers-color-scheme` emulation is currently set (from Task 1's Step 3 check).

- [ ] **Step 4: Run lint and type checks**

Run: `npm run lint && npm run check`

Expected: 0 errors, 0 warnings.

- [ ] **Step 5: Commit**

```bash
git add src/app.html
git commit -m "Load Montserrat font and add pre-hydration theme script"
```

---

### Task 3: `ThemeToggle` atom + wiring

**Files:**

- Create: `src/lib/components/atoms/ThemeToggle.svelte` (currently empty)
- Modify: `src/routes/+layout.svelte`

**Interfaces:**

- Consumes: the `data-theme` attribute mechanism and `'theme'` `localStorage` key established in Task 2; `--color-bg`, `--color-text`, `--color-border` tokens from Task 1.
- Produces: `ThemeToggle.svelte`, a self-contained component with no props, rendered as `<ThemeToggle />` from `+layout.svelte`.

- [ ] **Step 1: Write `ThemeToggle.svelte`**

`src/lib/components/atoms/ThemeToggle.svelte`:

```svelte
<script lang="ts">
	const STORAGE_KEY = 'theme';

	let isDark = $state(false);

	function resolveIsDark(): boolean {
		const explicit = document.documentElement.getAttribute('data-theme');
		if (explicit === 'dark') return true;
		if (explicit === 'light') return false;
		return window.matchMedia('(prefers-color-scheme: dark)').matches;
	}

	$effect(() => {
		isDark = resolveIsDark();

		const media = window.matchMedia('(prefers-color-scheme: dark)');
		function handleChange() {
			if (!document.documentElement.getAttribute('data-theme')) {
				isDark = media.matches;
			}
		}
		media.addEventListener('change', handleChange);
		return () => media.removeEventListener('change', handleChange);
	});

	function toggle() {
		const next = isDark ? 'light' : 'dark';
		document.documentElement.setAttribute('data-theme', next);
		localStorage.setItem(STORAGE_KEY, next);
		isDark = next === 'dark';
	}
</script>

<button
	type="button"
	class="theme-toggle"
	onclick={toggle}
	aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
>
	{isDark ? '☀️' : '🌙'}
</button>

<style>
	.theme-toggle {
		position: fixed;
		top: 1rem;
		right: 1rem;
		z-index: 1000;
		width: 2.5rem;
		height: 2.5rem;
		border-radius: 50%;
		border: 1px solid var(--color-border);
		background: var(--color-bg);
		color: var(--color-text);
		font-size: 1.1rem;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
```

- [ ] **Step 2: Render it from the layout**

Read the current `src/routes/+layout.svelte` first (from Task 1). Replace its contents with:

```svelte
<script lang="ts">
	import '$lib/styles/tokens.css';
	import favicon from '$lib/assets/favicon.svg';
	import ThemeToggle from '$lib/components/atoms/ThemeToggle.svelte';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<ThemeToggle />

{@render children()}
```

- [ ] **Step 3: Verify the toggle works end-to-end**

Run: `npm run dev`, open `http://localhost:5173/`.

Check:

- The toggle button appears fixed in the top-right corner, showing 🌙 (or ☀️, matching your OS's current light/dark setting).
- Clicking it flips the entire page's background/text colors instantly and the icon changes.
- Reload the page — the choice persists (page loads directly into the theme you last picked, no flash).
- In DevTools console, run `localStorage.removeItem('theme')` and reload — the toggle now reflects your OS preference again (use the Rendering-tab emulation from Task 1 to check both directions).
- `aria-label` on the button updates correctly to describe the _next_ action (e.g. shows "Switch to dark mode" while currently light).

- [ ] **Step 4: Run lint and type checks**

Run: `npm run lint && npm run check`

Expected: 0 errors, 0 warnings.

- [ ] **Step 5: Commit**

```bash
git add src/lib/components/atoms/ThemeToggle.svelte src/routes/+layout.svelte
git commit -m "Add ThemeToggle atom for manual light/dark override"
```

---

### Task 4: Migrate `HeroSlideshow.svelte` to design tokens

**Files:**

- Modify: `src/lib/components/organisms/HeroSlideshow.svelte`

**Interfaces:**

- Consumes: `--color-surface-inverted`, `--color-text-inverted`, `--color-accent` tokens from Task 1's `tokens.css` (already globally available by this point).

- [ ] **Step 1: Replace hardcoded colors with token references**

Read the current `src/lib/components/organisms/HeroSlideshow.svelte` first. Its `<style>` block currently ends with (verify this matches before editing):

```css
.hero-slideshow__stage {
	position: relative;
	width: 100%;
	height: 90vh;
	min-height: 480px;
	overflow: hidden;
	background: #1a1a1a;
}
```

Replace the entire `<style>` block (everything between `<style>` and `</style>`) with:

```css
<style>
	.hero-slideshow {
		display: flex;
		flex-direction: column;
	}

	.hero-slideshow__stage {
		position: relative;
		width: 100%;
		height: 90vh;
		min-height: 480px;
		overflow: hidden;
		background: var(--color-surface-inverted);
	}

	.hero-slideshow__arrow {
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		z-index: 3;
		background: transparent;
		border: none;
		color: var(--color-text-inverted);
		font-size: 3rem;
		line-height: 1;
		padding: 0.5rem 1rem;
		cursor: pointer;
	}

	.hero-slideshow__arrow--prev {
		left: 0.5rem;
	}

	.hero-slideshow__arrow--next {
		right: 0.5rem;
	}

	.hero-slideshow__indicators {
		position: absolute;
		right: 1.5rem;
		bottom: 1.5rem;
		z-index: 3;
		display: flex;
		gap: 1rem;
	}

	.hero-slideshow__indicator {
		background: transparent;
		border: none;
		color: color-mix(in srgb, var(--color-text-inverted) 60%, transparent);
		font-size: 0.85rem;
		letter-spacing: 0.05em;
		cursor: pointer;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.35rem;
		padding: 0;
	}

	.hero-slideshow__indicator--active {
		color: var(--color-text-inverted);
	}

	.hero-slideshow__indicator-progress {
		display: block;
		width: 2rem;
		height: 2px;
		background: color-mix(in srgb, var(--color-text-inverted) 30%, transparent);
	}

	.hero-slideshow__indicator-progress--active {
		background: var(--color-text-inverted);
		animation: hero-slideshow-progress 5000ms linear forwards;
		transform-origin: left;
	}

	@keyframes hero-slideshow-progress {
		from {
			transform: scaleX(0);
		}
		to {
			transform: scaleX(1);
		}
	}

	.hero-slideshow__caption {
		background: var(--color-surface-inverted);
		color: var(--color-text-inverted);
		padding: 2rem 3rem;
	}

	.hero-slideshow__eyebrow {
		margin: 0 0 0.5rem;
		color: var(--color-accent);
		text-transform: uppercase;
		letter-spacing: 0.08em;
		font-size: 0.85rem;
	}

	.hero-slideshow__title {
		margin: 0;
		font-size: 2rem;
		font-weight: 600;
	}

	@media (max-width: 640px) {
		.hero-slideshow__stage {
			height: 70vh;
		}

		.hero-slideshow__caption {
			padding: 1.5rem;
		}

		.hero-slideshow__arrow {
			font-size: 2rem;
		}
	}
</style>
```

Do not change the `<script>` block or the markup (everything above `<style>`) — this task is a CSS-only color substitution. Note two of the original near-black shades (`#1a1a1a` for the stage background, `#0d0d0d` for the caption background) both consolidate onto the single `--color-surface-inverted` token (`#232323`) — an intentional simplification from the design spec's token model, not a mistake.

- [ ] **Step 2: Verify visually in both themes**

Run: `npm run dev`, open `http://localhost:5173/`.

Check:

- The hero slideshow still renders correctly: dark stage background, light arrow/indicator text, caption band with light text and an accent-colored eyebrow line.
- Toggle light/dark mode (via the `ThemeToggle` button from Task 3). The hero's own colors should NOT change between themes (they use the brand-constant `--color-surface-inverted`/`--color-text-inverted`/`--color-accent` tokens) — only the rest of the page (currently just the toggle button itself, since no other content exists yet) should flip.

- [ ] **Step 3: Run lint and type checks**

Run: `npm run lint && npm run check`

Expected: 0 errors, 0 warnings.

- [ ] **Step 4: Commit**

```bash
git add src/lib/components/organisms/HeroSlideshow.svelte
git commit -m "Migrate HeroSlideshow hardcoded colors to design tokens"
```

---

## Self-Review Notes

- **Spec coverage:** Token system with real palette (Task 1), Montserrat typography (Task 2), light/dark toggle with system default + persistence + no-flash (Tasks 1-3), HeroSlideshow migration (Task 4), no image-protection CSS (not implemented anywhere in this plan — correctly absent), toggle as fixed-position element not inside Header.svelte (Task 3) — all spec sections covered.
- **Placeholder scan:** No TBD/TODO markers; all code blocks are complete and runnable as written.
- **Type consistency:** Token custom-property names (`--color-bg`, `--color-text`, `--color-border`, `--color-surface-inverted`, `--color-text-inverted`, `--color-accent`, `--color-accent-hover`, `--font-heading`, `--font-body`) are used identically across Task 1 (defined), Task 3 (`--color-bg`/`--color-text`/`--color-border` consumed by `ThemeToggle`), and Task 4 (`--color-surface-inverted`/`--color-text-inverted`/`--color-accent` consumed by `HeroSlideshow`). The `localStorage` key `'theme'` and the `data-theme` attribute name/values (`'light'`/`'dark'`) are used identically in Task 2's inline script and Task 3's `ThemeToggle.svelte`.
