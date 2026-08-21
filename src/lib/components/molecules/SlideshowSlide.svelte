<script lang="ts">
	let {
		src,
		alt,
		active
	}: {
		src: string;
		alt: string;
		active: boolean;
	} = $props();
</script>

<div class="slideshow-slide" class:slideshow-slide--active={active} aria-hidden={!active}>
	<img {src} {alt} class="slideshow-slide__image" />
</div>

<style>
	.slideshow-slide {
		position: absolute;
		inset: 0;
		opacity: 0;
		z-index: 1;
		transition: opacity 1.2s ease;
	}

	.slideshow-slide--active {
		opacity: 1;
		z-index: 2;
	}

	.slideshow-slide__image {
		display: block;
		width: 100%;
		height: 100%;
		object-fit: cover;
	}

	.slideshow-slide--active .slideshow-slide__image {
		animation: slideshow-slide-zoom 5s ease-out forwards;
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
