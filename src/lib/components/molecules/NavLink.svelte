<script lang="ts">
	import type { Snippet } from 'svelte';
	import { page } from '$app/state';

	let {
		href,
		variant = 'bar',
		children,
		onclick
	}: {
		href: string;
		variant?: 'bar' | 'menu';
		children: Snippet;
		onclick?: () => void;
	} = $props();

	let isActive = $derived(page.url.pathname === href);
</script>

<a
	{href}
	class="nav-link nav-link--{variant}"
	class:nav-link--active={isActive}
	aria-current={isActive ? 'page' : undefined}
	{onclick}
>
	{@render children()}
</a>

<style>
	.nav-link {
		color: inherit;
		text-decoration: none;
		transition:
			color 0.2s ease,
			border-color 0.2s ease;
	}

	.nav-link--bar {
		font-size: 0.85rem;
		letter-spacing: 0.05em;
		text-transform: uppercase;
		padding-bottom: 0.35rem;
		border-bottom: 2px solid transparent;
	}

	.nav-link--bar:hover,
	.nav-link--bar.nav-link--active {
		color: var(--color-accent);
	}

	.nav-link--bar.nav-link--active {
		border-bottom-color: var(--color-accent);
	}

	.nav-link--menu {
		display: block;
		padding: 0.6rem 1rem;
		font-size: 0.9rem;
		white-space: nowrap;
	}

	.nav-link--menu:hover,
	.nav-link--menu.nav-link--active {
		color: var(--color-accent);
	}
</style>
