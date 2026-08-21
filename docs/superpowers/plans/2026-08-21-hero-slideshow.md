# Hero Slideshow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a crossfading, Ken-Burns-zooming hero image slideshow on the homepage, matching the old WordPress site's behavior (placeholder images for now, real photos later).

**Architecture:** `HeroSlideshow.svelte` (organism) owns all state — current slide index, an activation counter used to restart per-slide CSS animations, autoplay timer, hover/focus pause — and renders a stack of `SlideshowSlide.svelte` (molecule, presentational-only) plus arrows, a numbered indicator with progress line, and a fixed caption band. `src/routes/+page.svelte` renders `<HeroSlideshow />`.

**Tech Stack:** SvelteKit 5 (runes mode: `$state`, `$props`, `$effect`), plain CSS (scoped `<style>` blocks, no framework), no new npm dependencies.

## Global Constraints

- No new dependencies — no carousel library (spec: Non-goals).
- Slide dwell time: ~5000ms per slide (spec: Animation & timing details).
- Crossfade duration: ~1.2s opacity transition (spec: Animation & timing details).
- Zoom: CSS keyframe `scale(1.15) → scale(1)`, ~5s, `ease-out` (spec: Animation & timing details).
- Caption text is fixed for all slides — "The art of making memories" (eyebrow) / "Photo & Art Experience" (title) — not data-driven (spec: Caption band).
- Do not build out `atoms/Image.svelte` — use a plain `<img>` in `SlideshowSlide` (spec: Non-goals).
- Do not touch `Header.svelte` / `Footer.svelte` — the top nav and social/copyright footer are out of scope (spec: Non-goals).
- Formatting: tabs, single quotes, no trailing commas (project `prettier.config.js`) — match this in all new files.
- `npm run lint` (prettier + eslint) and `npm run check` (svelte-check) must pass with no new errors after every task.

---

### Task 1: Placeholder hero images

**Files:**
- Create: `static/images/hero/placeholder-1.svg`
- Create: `static/images/hero/placeholder-2.svg`
- Create: `static/images/hero/placeholder-3.svg`
- Create: `static/images/hero/placeholder-4.svg`
- Create: `static/images/hero/placeholder-5.svg`

**Interfaces:**
- Produces: 5 static image files served by SvelteKit at `/images/hero/placeholder-1.svg` … `/images/hero/placeholder-5.svg` (anything under `static/` is served from the site root). Task 2 and Task 3 reference these exact URLs.

- [ ] **Step 1: Create the 5 placeholder SVG files**

`static/images/hero/placeholder-1.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1600 900" role="img" aria-label="Placeholder photo 1">
	<rect width="1600" height="900" fill="#3f3a2f" />
	<text x="50%" y="50%" fill="#f5efe6" font-family="Georgia, 'Times New Roman', serif" font-size="64" text-anchor="middle" dominant-baseline="middle">Placeholder 1</text>
</svg>
```

`static/images/hero/placeholder-2.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1600 900" role="img" aria-label="Placeholder photo 2">
	<rect width="1600" height="900" fill="#55483a" />
	<text x="50%" y="50%" fill="#f5efe6" font-family="Georgia, 'Times New Roman', serif" font-size="64" text-anchor="middle" dominant-baseline="middle">Placeholder 2</text>
</svg>
```

`static/images/hero/placeholder-3.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1600 900" role="img" aria-label="Placeholder photo 3">
	<rect width="1600" height="900" fill="#2e3b2f" />
	<text x="50%" y="50%" fill="#f5efe6" font-family="Georgia, 'Times New Roman', serif" font-size="64" text-anchor="middle" dominant-baseline="middle">Placeholder 3</text>
</svg>
```

`static/images/hero/placeholder-4.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1600 900" role="img" aria-label="Placeholder photo 4">
	<rect width="1600" height="900" fill="#4a3f35" />
	<text x="50%" y="50%" fill="#f5efe6" font-family="Georgia, 'Times New Roman', serif" font-size="64" text-anchor="middle" dominant-baseline="middle">Placeholder 4</text>
</svg>
```

`static/images/hero/placeholder-5.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1600 900" role="img" aria-label="Placeholder photo 5">
	<rect width="1600" height="900" fill="#33302a" />
	<text x="50%" y="50%" fill="#f5efe6" font-family="Georgia, 'Times New Roman', serif" font-size="64" text-anchor="middle" dominant-baseline="middle">Placeholder 5</text>
</svg>
```

- [ ] **Step 2: Verify the files are served correctly**

Run: `npm run dev`

Then in a browser (or `curl`), open each of:
- `http://localhost:5173/images/hero/placeholder-1.svg`
- `http://localhost:5173/images/hero/placeholder-2.svg`
- `http://localhost:5173/images/hero/placeholder-3.svg`
- `http://localhost:5173/images/hero/placeholder-4.svg`
- `http://localhost:5173/images/hero/placeholder-5.svg`

Expected: each URL renders a solid-color rectangle with centered "Placeholder N" text, no 404s.

- [ ] **Step 3: Commit**

```bash
git add static/images/hero
git commit -m "Add placeholder hero slideshow images"
```

---

### Task 2: `SlideshowSlide` molecule (crossfade + Ken Burns zoom)

**Files:**
- Modify: `src/lib/components/molecules/SlideshowSlide.svelte` (currently empty)
- Modify: `src/routes/+page.svelte` (temporary visual harness — replaced in Task 3)

**Interfaces:**
- Consumes: image URLs produced by Task 1 (`/images/hero/placeholder-1.svg`).
- Produces: `SlideshowSlide.svelte` component with props `{ src: string; alt: string; active: boolean; activationKey: number }`. Task 3's `HeroSlideshow.svelte` renders one `SlideshowSlide` per slide with these exact prop names and types.

- [ ] **Step 1: Write `SlideshowSlide.svelte`**

`src/lib/components/molecules/SlideshowSlide.svelte`:

```svelte
<script lang="ts">
	let {
		src,
		alt,
		active,
		activationKey
	}: {
		src: string;
		alt: string;
		active: boolean;
		activationKey: number;
	} = $props();
</script>

{#key activationKey}
	<img {src} {alt} class="slideshow-slide" class:slideshow-slide--active={active} />
{/key}

<style>
	.slideshow-slide {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		object-fit: cover;
		opacity: 0;
		z-index: 1;
		transition: opacity 1.2s ease;
		animation: slideshow-slide-zoom 5s ease-out forwards;
	}

	.slideshow-slide--active {
		opacity: 1;
		z-index: 2;
	}

	@keyframes slideshow-slide-zoom {
		from {
			transform: scale(1.15);
		}
		to {
			transform: scale(1);
		}
	}
</style>
```

- [ ] **Step 2: Temporarily render it in the homepage to verify visually**

Replace the contents of `src/routes/+page.svelte` with:

```svelte
<script lang="ts">
	import SlideshowSlide from '$lib/components/molecules/SlideshowSlide.svelte';
</script>

<div style="position: relative; width: 100%; height: 100vh;">
	<SlideshowSlide
		src="/images/hero/placeholder-1.svg"
		alt="Placeholder photo 1"
		active={true}
		activationKey={0}
	/>
</div>
```

Run: `npm run dev`, open `http://localhost:5173/`.

Expected: the placeholder image fills the viewport, fades in, and slowly zooms from slightly-enlarged down to its normal size over about 5 seconds (then holds at normal size — the timer that would advance to the next slide is built in Task 3, so it should NOT loop or change on its own yet).

- [ ] **Step 3: Run lint and type checks**

Run: `npm run lint && npm run check`

Expected: no errors.

- [ ] **Step 4: Commit**

```bash
git add src/lib/components/molecules/SlideshowSlide.svelte src/routes/+page.svelte
git commit -m "Add SlideshowSlide molecule with crossfade + zoom animation"
```

---

### Task 3: `HeroSlideshow` organism (state, timer, controls, caption) + homepage wiring

**Files:**
- Modify: `src/lib/components/organisms/HeroSlideshow.svelte` (currently empty)
- Modify: `src/routes/+page.svelte` (replace Task 2's temporary harness with the real component)

**Interfaces:**
- Consumes: `SlideshowSlide` from Task 2 with props `{ src, alt, active, activationKey }`; placeholder image URLs from Task 1.
- Produces: `HeroSlideshow.svelte`, a self-contained component with no props, rendered as `<HeroSlideshow />` from `+page.svelte`.

- [ ] **Step 1: Write `HeroSlideshow.svelte`**

`src/lib/components/organisms/HeroSlideshow.svelte`:

```svelte
<script lang="ts">
	import SlideshowSlide from '$lib/components/molecules/SlideshowSlide.svelte';

	const SLIDE_DURATION_MS = 5000;

	const slides = [
		{ src: '/images/hero/placeholder-1.svg', alt: 'Placeholder photo 1' },
		{ src: '/images/hero/placeholder-2.svg', alt: 'Placeholder photo 2' },
		{ src: '/images/hero/placeholder-3.svg', alt: 'Placeholder photo 3' },
		{ src: '/images/hero/placeholder-4.svg', alt: 'Placeholder photo 4' },
		{ src: '/images/hero/placeholder-5.svg', alt: 'Placeholder photo 5' }
	];

	let currentIndex = $state(0);
	let activationCounter = $state(0);
	let paused = $state(false);

	function goTo(rawIndex: number) {
		const newIndex = ((rawIndex % slides.length) + slides.length) % slides.length;
		if (newIndex === currentIndex) return;
		currentIndex = newIndex;
		activationCounter += 1;
	}

	function next() {
		goTo(currentIndex + 1);
	}

	function prev() {
		goTo(currentIndex - 1);
	}

	function handleFocusOut(event: FocusEvent) {
		const container = event.currentTarget as HTMLElement;
		const nextFocused = event.relatedTarget as Node | null;
		if (!nextFocused || !container.contains(nextFocused)) {
			paused = false;
		}
	}

	$effect(() => {
		if (paused) return;
		const slideAtArmTime = currentIndex;
		const timeoutId = setTimeout(() => {
			goTo(slideAtArmTime + 1);
		}, SLIDE_DURATION_MS);
		return () => clearTimeout(timeoutId);
	});
</script>

<section
	class="hero-slideshow"
	onmouseenter={() => (paused = true)}
	onmouseleave={() => (paused = false)}
	onfocusin={() => (paused = true)}
	onfocusout={handleFocusOut}
>
	<div class="hero-slideshow__stage">
		{#each slides as slide, i (slide.src)}
			<SlideshowSlide
				src={slide.src}
				alt={slide.alt}
				active={i === currentIndex}
				activationKey={activationCounter}
			/>
		{/each}

		<button
			type="button"
			class="hero-slideshow__arrow hero-slideshow__arrow--prev"
			onclick={prev}
			aria-label="Previous slide"
		>
			‹
		</button>
		<button
			type="button"
			class="hero-slideshow__arrow hero-slideshow__arrow--next"
			onclick={next}
			aria-label="Next slide"
		>
			›
		</button>

		<div class="hero-slideshow__indicators">
			{#each slides as slide, i (slide.src)}
				<button
					type="button"
					class="hero-slideshow__indicator"
					class:hero-slideshow__indicator--active={i === currentIndex}
					onclick={() => goTo(i)}
					aria-label={`Go to slide ${i + 1}`}
				>
					<span class="hero-slideshow__indicator-number">{String(i + 1).padStart(2, '0')}</span>
					{#key activationCounter}
						<span
							class="hero-slideshow__indicator-progress"
							class:hero-slideshow__indicator-progress--active={i === currentIndex}
						></span>
					{/key}
				</button>
			{/each}
		</div>
	</div>

	<div class="hero-slideshow__caption">
		<p class="hero-slideshow__eyebrow">The art of making memories</p>
		<h1 class="hero-slideshow__title">Photo &amp; Art Experience</h1>
	</div>
</section>

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
		background: #1a1a1a;
	}

	.hero-slideshow__arrow {
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		z-index: 3;
		background: transparent;
		border: none;
		color: #f5efe6;
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
		color: rgba(245, 239, 230, 0.6);
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
		color: #f5efe6;
	}

	.hero-slideshow__indicator-progress {
		display: block;
		width: 2rem;
		height: 2px;
		background: rgba(245, 239, 230, 0.3);
	}

	.hero-slideshow__indicator-progress--active {
		background: #f5efe6;
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
		background: #0d0d0d;
		color: #f5efe6;
		padding: 2rem 3rem;
	}

	.hero-slideshow__eyebrow {
		margin: 0 0 0.5rem;
		color: #c98a4b;
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

- [ ] **Step 2: Wire it into the homepage**

Replace the contents of `src/routes/+page.svelte` with:

```svelte
<script lang="ts">
	import HeroSlideshow from '$lib/components/organisms/HeroSlideshow.svelte';
</script>

<HeroSlideshow />
```

- [ ] **Step 3: Manually verify full behavior**

Run: `npm run dev`, open `http://localhost:5173/`.

Check each of the following:
- Slides autoplay, advancing roughly every 5 seconds, with a crossfade and Ken-Burns zoom-out on each new slide.
- Clicking the `‹` / `›` arrows immediately moves to the previous/next slide and restarts the 5s timer.
- Clicking a number (`01`–`05`) jumps straight to that slide, highlights it, and its progress line restarts from empty.
- Hovering over the slideshow pauses autoplay; moving the mouse away resumes it.
- Tabbing focus onto an arrow or number pauses autoplay; tabbing focus away resumes it.
- The caption band below the image reads "The art of making memories" / "Photo & Art Experience" and does not change between slides.

- [ ] **Step 4: Run lint and type checks**

Run: `npm run lint && npm run check`

Expected: no errors.

- [ ] **Step 5: Commit**

```bash
git add src/lib/components/organisms/HeroSlideshow.svelte src/routes/+page.svelte
git commit -m "Add HeroSlideshow organism with autoplay, controls, and caption band"
```

---

## Self-Review Notes

- **Spec coverage:** Crossfade + Ken Burns zoom (Task 2/3), 5s dwell timer (Task 3), prev/next arrows (Task 3), clickable numbered indicator + progress line (Task 3), pause on hover/focus (Task 3), fixed caption band (Task 3), placeholder images (Task 1), no new dependencies (all tasks use only built-in Svelte/CSS), `atoms/Image.svelte` left untouched (Task 2 uses plain `<img>`), Header/Footer untouched (no task modifies them) — all spec sections are covered.
- **Placeholder scan:** No TBD/TODO markers; all code blocks are complete and runnable as written.
- **Type consistency:** `SlideshowSlide` props (`src: string`, `alt: string`, `active: boolean`, `activationKey: number`) match exactly between its Task 2 definition and Task 3's usage.
