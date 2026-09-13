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
	aria-pressed={isDark}
>
	<span class="theme-toggle__icon theme-toggle__icon--sun" aria-hidden="true">☀️</span>
	<span class="theme-toggle__icon theme-toggle__icon--moon" aria-hidden="true">🌙</span>
</button>

<style>
	.theme-toggle {
		width: 2.25rem;
		height: 2.25rem;
		border-radius: 50%;
		border: 1px solid currentColor;
		background: transparent;
		color: inherit;
		font-size: 1rem;
		cursor: pointer;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
	}

	.theme-toggle__icon {
		display: none;
	}

	.theme-toggle__icon--moon {
		display: block;
	}

	:global(html:not([data-theme='light'])) .theme-toggle__icon--sun {
		display: none;
	}

	@media (prefers-color-scheme: dark) {
		:global(html:not([data-theme='light'])) .theme-toggle__icon--sun {
			display: block;
		}

		:global(html:not([data-theme='light'])) .theme-toggle__icon--moon {
			display: none;
		}
	}

	:global(html[data-theme='dark']) .theme-toggle__icon--sun {
		display: block;
	}

	:global(html[data-theme='dark']) .theme-toggle__icon--moon {
		display: none;
	}
</style>
