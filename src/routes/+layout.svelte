<script lang="ts">
	import '$lib/styles/tokens.css';
	import '@fontsource/montserrat/400.css';
	import '@fontsource/montserrat/700.css';
	import favicon from '$lib/assets/favicon.svg';
	import ThemeToggle from '$lib/components/atoms/ThemeToggle.svelte';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<ThemeToggle />

{@render children()}
