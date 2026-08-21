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
