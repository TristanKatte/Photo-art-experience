<script lang="ts">
	import Icon from '$lib/components/atoms/Icon.svelte';

	// TODO: swap in the real Facebook/Instagram URLs once they're available.
	const year = new Date().getFullYear();

	function scrollToTop() {
		window.scrollTo({ top: 0, behavior: 'smooth' });
	}
</script>

<footer class="site-footer">
	<div class="site-footer__social">
		<a
			href="https://www.facebook.com"
			target="_blank"
			rel="noreferrer"
			class="site-footer__social-link"
		>
			<Icon name="facebook" size="1.1rem" />
			<span>Facebook</span>
		</a>
		<span class="site-footer__divider" aria-hidden="true">&bull;</span>
		<a
			href="https://www.instagram.com"
			target="_blank"
			rel="noreferrer"
			class="site-footer__social-link"
		>
			<Icon name="instagram" size="1.1rem" />
			<span>Instagram</span>
		</a>
	</div>

	<div class="site-footer__bottom">
		<small>&copy; {year} Photo &amp; Art Experience. All Rights Reserved.</small>
		<button
			type="button"
			class="site-footer__to-top"
			onclick={scrollToTop}
			aria-label="Terug naar boven"
		>
			<Icon name="arrow-up" size="1rem" />
		</button>
	</div>
</footer>

<style>
	.site-footer {
		background: var(--color-bg);
		color: var(--color-text);
		border-top: 1px solid var(--color-border);
	}

	.site-footer__social {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 1rem;
		padding: 1.5rem;
		font-size: 0.9rem;
	}

	.site-footer__social-link {
		display: inline-flex;
		align-items: center;
		gap: 0.4rem;
		color: inherit;
		text-decoration: none;
	}

	.site-footer__social-link:hover {
		color: var(--color-accent);
	}

	.site-footer__divider {
		color: var(--color-border);
	}

	.site-footer__bottom {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
		padding: 1rem 1.5rem;
		border-top: 1px solid var(--color-border);
		font-size: 0.8rem;
	}

	.site-footer__to-top {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		width: 2.25rem;
		height: 2.25rem;
		border-radius: 50%;
		border: 1px solid currentColor;
		background: transparent;
		color: inherit;
		cursor: pointer;
		flex-shrink: 0;
	}

	.site-footer__to-top:hover {
		color: var(--color-accent);
		border-color: var(--color-accent);
	}

	@media (max-width: 480px) {
		.site-footer__bottom {
			flex-direction: column;
			text-align: center;
		}
	}
</style>
