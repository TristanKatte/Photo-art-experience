<script lang="ts">
	import { page } from '$app/state';
	import Icon from '$lib/components/atoms/Icon.svelte';
	import NavLink from '$lib/components/molecules/NavLink.svelte';
	import ThemeToggle from '$lib/components/atoms/ThemeToggle.svelte';

	// TODO: swap the typographic wordmark below for the real logo asset once it's ready.
	const portfolioLinks = [
		{ href: '/portfolio/fantasie', label: 'Fantasie' },
		{ href: '/portfolio/painterly-portret', label: 'Painterly portret' },
		{ href: '/portfolio/kinderen', label: 'Kinderen' },
		{ href: '/portfolio/locatie-en-urbex', label: 'Locatie en Urbex' },
		{ href: '/editing', label: 'Voor & na' }
	];

	let mobileOpen = $state(false);
	let portfolioOpen = $state(false);
	let dropdownEl: HTMLDivElement | undefined;

	function closeMenus() {
		mobileOpen = false;
		portfolioOpen = false;
	}

	function toggleMobile() {
		mobileOpen = !mobileOpen;
		portfolioOpen = false;
	}

	function togglePortfolio() {
		portfolioOpen = !portfolioOpen;
	}

	function handleDocumentClick(event: MouseEvent) {
		if (dropdownEl && !dropdownEl.contains(event.target as Node)) {
			portfolioOpen = false;
		}
	}

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Escape') closeMenus();
	}

	$effect(() => {
		// close any open menu whenever the route changes
		page.url.pathname;
		closeMenus();
	});
</script>

<svelte:window onclick={handleDocumentClick} onkeydown={handleKeydown} />

<header class="site-header">
	<div class="site-header__bar">
		<a href="/" class="site-header__logo" onclick={closeMenus}>Photo &amp; Art Experience</a>

		<button
			type="button"
			class="site-header__menu-toggle"
			aria-expanded={mobileOpen}
			aria-controls="primary-nav"
			aria-label={mobileOpen ? 'Sluit menu' : 'Open menu'}
			onclick={toggleMobile}
		>
			<Icon name={mobileOpen ? 'close' : 'menu'} size="1.3rem" />
		</button>

		<nav
			id="primary-nav"
			class="site-header__nav"
			class:site-header__nav--open={mobileOpen}
			aria-label="Primary"
		>
			<NavLink href="/" onclick={closeMenus}>Home</NavLink>

			<div class="site-header__dropdown" bind:this={dropdownEl}>
				<button
					type="button"
					class="site-header__dropdown-toggle"
					aria-haspopup="true"
					aria-expanded={portfolioOpen}
					onclick={togglePortfolio}
				>
					Portfolio
					<Icon name="chevron-down" size="0.9rem" />
				</button>

				{#if portfolioOpen}
					<ul class="site-header__dropdown-menu" role="menu">
						{#each portfolioLinks as link (link.href)}
							<li role="none">
								<NavLink href={link.href} variant="menu" onclick={closeMenus}>
									{link.label}
								</NavLink>
							</li>
						{/each}
					</ul>
				{/if}
			</div>

			<NavLink href="/booking" onclick={closeMenus}>Boek een fotoshoot</NavLink>
			<NavLink href="/about" onclick={closeMenus}>Over Andrea</NavLink>
			<NavLink href="/contact" onclick={closeMenus}>Contact</NavLink>

			<ThemeToggle />
		</nav>
	</div>
</header>

<style>
	.site-header {
		position: fixed;
		inset: 0 0 auto 0;
		z-index: 100;
		/* no static rgba fallback here: the underlying color itself changes with
		   the light/dark toggle, so a single hardcoded fallback can't be correct
		   for both themes. color-mix() is supported by all current browsers. */
		background: color-mix(in srgb, var(--color-bg) 80%, transparent);
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		color: var(--color-text);
		border-bottom: 1px solid var(--color-border);
	}

	.site-header__bar {
		height: var(--header-height);
		max-width: 1440px;
		margin: 0 auto;
		padding: 0 1.5rem;
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1.5rem;
	}

	.site-header__logo {
		color: inherit;
		text-decoration: none;
		font-family: var(--font-heading);
		font-weight: var(--font-heading-weight);
		font-size: 1.1rem;
		letter-spacing: 0.02em;
		white-space: nowrap;
	}

	.site-header__menu-toggle {
		display: none;
		background: transparent;
		border: none;
		color: inherit;
		cursor: pointer;
		padding: 0.25rem;
	}

	.site-header__nav {
		display: flex;
		align-items: center;
		gap: 2rem;
	}

	.site-header__dropdown {
		position: relative;
	}

	.site-header__dropdown-toggle {
		display: inline-flex;
		align-items: center;
		gap: 0.35rem;
		background: transparent;
		border: none;
		color: inherit;
		font: inherit;
		font-size: 0.85rem;
		letter-spacing: 0.05em;
		text-transform: uppercase;
		cursor: pointer;
		padding: 0.35rem 0;
	}

	.site-header__dropdown-toggle:hover {
		color: var(--color-accent);
	}

	.site-header__dropdown-menu {
		position: absolute;
		top: calc(100% + 0.75rem);
		left: 0;
		min-width: 12rem;
		margin: 0;
		padding: 0.5rem 0;
		list-style: none;
		background: var(--color-bg);
		border: 1px solid var(--color-border);
		border-radius: 0.25rem;
		box-shadow: 0 0.75rem 1.5rem rgba(0, 0, 0, 0.3);
	}

	@media (max-width: 860px) {
		.site-header__menu-toggle {
			display: block;
		}

		.site-header__nav {
			display: none;
			position: fixed;
			/* .site-header's backdrop-filter makes it the containing block for this
			   fixed descendant, so top/height are sized off the header's own box
			   (top: 100% = its bottom edge) rather than the viewport. */
			top: 100%;
			left: 0;
			right: 0;
			height: calc(100vh - var(--header-height));
			box-sizing: border-box;
			flex-direction: column;
			align-items: stretch;
			gap: 0;
			padding: 1rem 1.5rem 2rem;
			background: var(--color-bg);
			overflow-y: auto;
		}

		.site-header__nav--open {
			display: flex;
		}

		.site-header__dropdown-menu {
			position: static;
			border: none;
			box-shadow: none;
			background: transparent;
			padding-left: 1rem;
		}

		.site-header__nav :global(.nav-link--bar) {
			padding: 0.75rem 0;
			border-bottom: 1px solid var(--color-border);
		}
	}
</style>
