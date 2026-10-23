<script lang="ts">
	type IconName = 'chevron-down' | 'facebook' | 'instagram' | 'arrow-up' | 'menu' | 'close';

	let { name, size = '1em' }: { name: IconName; size?: string } = $props();
</script>

<svg
	class="icon"
	style:width={size}
	style:height={size}
	viewBox="0 0 24 24"
	fill="none"
	stroke="currentColor"
	stroke-width="2"
	stroke-linecap="round"
	stroke-linejoin="round"
	aria-hidden="true"
	focusable="false"
>
	{#if name === 'chevron-down'}
		<polyline points="6 9 12 15 18 9" />
	{:else if name === 'menu'}
		<line x1="3" y1="6" x2="21" y2="6" />
		<line x1="3" y1="12" x2="21" y2="12" />
		<line x1="3" y1="18" x2="21" y2="18" />
	{:else if name === 'close'}
		<line x1="5" y1="5" x2="19" y2="19" />
		<line x1="19" y1="5" x2="5" y2="19" />
	{:else if name === 'arrow-up'}
		<line x1="12" y1="19" x2="12" y2="5" />
		<polyline points="5 12 12 5 19 12" />
	{:else if name === 'facebook'}
		<path d="M18 2h-3a5 5 0 0 0-5 5v3H7v4h3v8h4v-8h3l1-4h-4V7a1 1 0 0 1 1-1h3z" />
	{:else if name === 'instagram'}
		<rect x="2" y="2" width="20" height="20" rx="5" ry="5" />
		<path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z" />
		<line x1="17.51" y1="6.5" x2="17.5" y2="6.5" />
	{/if}
</svg>

<style>
	.icon {
		display: inline-block;
		vertical-align: middle;
		flex-shrink: 0;
	}
</style>
