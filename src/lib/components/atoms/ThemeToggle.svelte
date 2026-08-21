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
