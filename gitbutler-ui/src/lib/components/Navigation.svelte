<script lang="ts">
	import BaseBranchCard from './BaseBranchCard.svelte';
	import Branches from './Branches.svelte';
	import DomainButton from './DomainButton.svelte';
	import Footer from './Footer.svelte';
	import ProjectSelector from './ProjectSelector.svelte';
	import Resizer from './Resizer.svelte';
	import { persisted } from '$lib/persisted/persisted';
	import { SETTINGS_CONTEXT, type SettingsStore } from '$lib/settings/userSettings';
	import * as hotkeys from '$lib/utils/hotkeys';
	import { unsubscribe } from '$lib/utils/random';
	import { platform } from '@tauri-apps/api/os';
	import { from } from 'rxjs';
	import { onMount } from 'svelte';
	import { getContext } from 'svelte';
	import type { User } from '$lib/backend/cloud';
	import type { Project, ProjectService } from '$lib/backend/projects';
	import type { BranchService } from '$lib/branches/service';
	import type { GitHubService } from '$lib/github/service';
	import type { BranchController } from '$lib/vbranches/branchController';
	import type { BaseBranchService } from '$lib/vbranches/branchStoresCache';

	export let branchService: BranchService;
	export let baseBranchService: BaseBranchService;
	export let branchController: BranchController;
	export let project: Project;
	export let user: User | undefined;
	export let githubService: GitHubService;
	export let projectService: ProjectService;

	const minResizerWidth = 280;
	const minResizerRatio = 150;
	const platformName = from(platform());
	const userSettings = getContext<SettingsStore>(SETTINGS_CONTEXT);
	const defaultTrayWidthRem = persisted<number | undefined>(
		undefined,
		'defaulTrayWidth_ ' + project.id
	);

	let viewport: HTMLDivElement;
	let isResizerHovered = false;
	let isResizerDragging = false;

	$: isNavCollapsed = persisted<boolean>(false, 'projectNavCollapsed_' + project.id);

	function toggleNavCollapse() {
		$isNavCollapsed = !$isNavCollapsed;
	}

	onMount(() =>
		unsubscribe(
			hotkeys.on('Meta+/', () => {
				toggleNavCollapse();
			})
		)
	);
</script>

<aside class="navigation-wrapper">
	<div class="resizer-wrapper" tabindex="0" role="button">
		<button
			class="folding-button"
			class:resizer-hovered={isResizerHovered || isResizerDragging}
			on:mousedown={toggleNavCollapse}
			class:folding-button_folded={$isNavCollapsed}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				xmlns:xlink="http://www.w3.org/1999/xlink"
				viewBox="0 0 8 12"
				fill="none"
				><path
					d="M6,0L0,6l6,6"
					transform="translate(1 0)"
					stroke-width="1.5"
					stroke-linejoin="round"
				/></svg
			>
		</button>
		<Resizer
			{viewport}
			direction="right"
			minWidth={minResizerWidth}
			defaultLineColor="var(--clr-theme-container-outline-light)"
			on:click={() => $isNavCollapsed && toggleNavCollapse()}
			on:dblclick={() => !$isNavCollapsed && toggleNavCollapse()}
			on:width={(e) => {
				$defaultTrayWidthRem = e.detail / (16 * $userSettings.zoom);
			}}
			on:hover={(e) => {
				isResizerHovered = e.detail;
			}}
			on:resizing={(e) => {
				isResizerDragging = e.detail;
			}}
			on:overflowValue={(e) => {
				const overflowValue = e.detail;

				if (!$isNavCollapsed && overflowValue > minResizerRatio) {
					$isNavCollapsed = true;
				}

				if ($isNavCollapsed && overflowValue < minResizerRatio) {
					$isNavCollapsed = false;
				}
			}}
		/>
	</div>

	<div
		class="navigation"
		class:collapsed={$isNavCollapsed}
		style:width={$defaultTrayWidthRem && !$isNavCollapsed ? $defaultTrayWidthRem + 'rem' : null}
		bind:this={viewport}
		role="menu"
		tabindex="0"
	>
		<!-- condition prevents split second UI shift -->
		{#if $platformName}
			<div class="navigation-top">
				{#if $platformName == 'darwin'}
					<div class="drag-region" data-tauri-drag-region />
				{/if}
				<ProjectSelector {project} {projectService} isNavCollapsed={$isNavCollapsed} />
				<div class="domains">
					<BaseBranchCard
						{project}
						{baseBranchService}
						{githubService}
						isNavCollapsed={$isNavCollapsed}
					/>
					<DomainButton
						href={`/${project.id}/board`}
						domain="workspace"
						label="Workspace"
						iconSrc="/images/domain-icons/working-branches.svg"
						{branchController}
						{baseBranchService}
						isNavCollapsed={$isNavCollapsed}
					/>
				</div>
			</div>
			{#if !$isNavCollapsed}
				<Branches projectId={project.id} {branchService} {githubService} />
			{/if}
			<Footer {user} projectId={project.id} isNavCollapsed={$isNavCollapsed} />
		{/if}
	</div>
</aside>

<style lang="postcss">
	.navigation-wrapper {
		display: flex;
		position: relative;

		&:hover {
			& .folding-button {
				opacity: 1;
				right: calc(var(--space-6) * -1);
			}
		}
	}

	.navigation {
		width: 17.5rem;
		display: flex;
		flex-direction: column;
		position: relative;
		background: var(--clr-theme-container-light);
		max-height: 100%;
		user-select: none;
	}
	.drag-region {
		flex-shrink: 0;
		height: var(--space-20);
	}
	.navigation-top {
		display: flex;
		flex-direction: column;
		padding-bottom: var(--space-24);
		padding-left: var(--space-14);
		padding-right: var(--space-14);
	}
	.domains {
		display: flex;
		flex-direction: column;
		gap: var(--space-4);
	}

	.resizer-wrapper {
		position: absolute;
		top: 0;
		right: 0;
		height: 100%;
		width: var(--space-4);
	}

	.folding-button {
		z-index: 42;
		position: absolute;
		right: calc(var(--space-2) * -1);
		top: 50%;
		transform: translateY(-50%);
		width: var(--space-16);
		height: var(--space-36);
		padding: var(--space-4);
		background: var(--clr-theme-container-light);
		border-radius: var(--radius-m);
		border: 1px solid var(--clr-theme-container-outline-light);
		opacity: 0;
		transition:
			background-color var(--transition-fast),
			border-color var(--transition-fast),
			opacity var(--transition-medium),
			right var(--transition-fast);

		& svg {
			stroke: var(--clr-theme-scale-ntrl-50);
			transition: stroke var(--transition-fast);
		}

		&:hover {
			background-color: color-mix(
				in srgb,
				var(--clr-theme-container-light),
				var(--darken-tint-extralight)
			);
			border-color: color-mix(
				in srgb,
				var(--clr-theme-container-outline-light),
				var(--darken-tint-dark)
			);
		}
	}

	.folding-button_folded {
		& svg {
			transform: rotate(180deg) translateX(-0.0625rem);
		}
	}

	/* MODIFIERS */

	.navigation.collapsed {
		width: auto;
		justify-content: space-between;
		padding-bottom: var(--space-16);
	}

	.resizer-hovered {
		background-color: var(--resizer-color);
		border: 1px solid var(--resizer-color);
		transition-delay: 0.1s;

		& svg {
			stroke: var(--clr-theme-scale-ntrl-100);
			transition-delay: 0.1s;
		}
	}
</style>
