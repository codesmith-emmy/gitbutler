<script lang="ts">
	import Icon from '$lib/components/Icon.svelte';
	import { getContext } from 'svelte';
	import type iconsJson from '$lib/icons/icons.json';
	import type { ContextMenuContext } from './contextMenu';

	export let icon: keyof typeof iconsJson | undefined = undefined;
	export let id: string | undefined = undefined;
	export let label: string;
	export let selected = false;
	export let disabled = false;

	const context = getContext<ContextMenuContext>('ContextMenu');
	const selection$ = context.selection$;

	$: if (selected && id) selection$.next({ id, label });
</script>

<button
	class="menu-item"
	class:disabled
	{disabled}
	{id}
	on:mousedown
	on:click
	on:click={(e) => {
		e.stopPropagation();
		if (id && !disabled) selection$.next({ id, label });
	}}
>
	{#if icon}
		<Icon name={icon} />
	{/if}

	<span class="label text-base-12">
		{label}
	</span>
	<slot name="control" />
</button>

<style lang="postcss">
	.menu-item {
		cursor: pointer;
		display: flex;
		text-align: left;
		align-items: center;
		color: var(--clr-theme-scale-ntrl-0);
		height: var(--space-24);
		padding: var(--space-4) var(--space-6);
		border-radius: var(--radius-s);
		gap: var(--space-12);
		transition: background-color var(--transition-fast);
		&:not(.disabled):hover {
			transition: none;
			background-color: color-mix(in srgb, transparent, var(--darken-tint-light));
		}
	}
	.label {
		user-select: none;
		flex-grow: 1;
		white-space: nowrap;
	}
	.disabled {
		color: var(--clr-theme-scale-ntrl-50);
	}
</style>
