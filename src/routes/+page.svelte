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
