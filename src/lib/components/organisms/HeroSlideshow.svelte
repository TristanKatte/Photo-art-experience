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
	let hovered = $state(false);
	let focusWithin = $state(false);
	let paused = $derived(hovered || focusWithin);

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
			focusWithin = false;
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
	aria-label="Hero slideshow"
	onmouseenter={() => (hovered = true)}
	onmouseleave={() => (hovered = false)}
	onfocusin={() => (focusWithin = true)}
	onfocusout={handleFocusOut}
>
	<div class="hero-slideshow__stage">
		{#each slides as slide, i (slide.src)}
			<SlideshowSlide src={slide.src} alt={slide.alt} active={i === currentIndex} />
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
