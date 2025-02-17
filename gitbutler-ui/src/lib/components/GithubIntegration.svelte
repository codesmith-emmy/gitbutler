<script lang="ts">
	import Icon from './Icon.svelte';
	import Link from './Link.svelte';
	import { checkAuthStatus, initDeviceOauth } from '$lib/backend/github';
	import Button from '$lib/components/Button.svelte';
	import Modal from '$lib/components/Modal.svelte';
	import SectionCard from '$lib/components/SectionCard.svelte';
	import { getAuthenticated } from '$lib/github/user';
	import { copyToClipboard } from '$lib/utils/clipboard';
	import * as toasts from '$lib/utils/toasts';
	import type { UserService } from '$lib/stores/user';

	export let userService: UserService;
	export let minimal = false;
	export let disabled = false;

	$: user$ = userService.user$;

	let isCheckingStatus = false;

	let userCode = '';
	let deviceCode = '';

	function gitHubStartOauth() {
		initDeviceOauth().then((verification) => {
			userCode = verification.user_code;
			deviceCode = verification.device_code;
			gitHubOauthModal.show();
		});
	}

	let gitHubOauthModal: Modal;
	function gitHubOauthCheckStatus(deviceCode: string) {
		isCheckingStatus = true;
		let u = $user$;

		checkAuthStatus({ deviceCode })
			.then(async (access_token) => {
				if (u) {
					u.github_access_token = access_token;
					u.github_username = await getAuthenticated(access_token);
					userService.setUser(u);

					toasts.success('GitHub authenticated');

					isCheckingStatus = false;
					gitHubOauthModal.close();
				}
			})
			.catch((err) => {
				console.log(err);
				isCheckingStatus = false;
				gitHubOauthModal.close();
			});
	}

	function forgetGitHub(): void {
		let u = $user$;
		if (u) {
			u.github_access_token = '';
			u.github_username = '';
			userService.setUser(u);
		}
	}
</script>

{#if minimal}
	<Button {disabled} kind="filled" color="primary" on:click={gitHubStartOauth}>Authorize</Button>
{:else}
	<SectionCard orientation="row">
		<svelte:fragment slot="iconSide">
			<div class="icon-wrapper">
				{#if $user$?.github_access_token}
					<div class="icon-wrapper__tick">
						<Icon name="success" color="success" size={18} />
					</div>
				{/if}
				<svg
					width="28"
					height="28"
					viewBox="0 0 28 28"
					fill="var(--clr-theme-scale-ntrl-0)"
					xmlns="http://www.w3.org/2000/svg"
				>
					<path
						fill-rule="evenodd"
						clip-rule="evenodd"
						d="M14.0116 0C6.26354 0 0 6.41664 0 14.3549C0 20.7004 4.01327 26.0717 9.58073 27.9728C10.2768 28.1157 10.5318 27.6639 10.5318 27.2838C10.5318 26.9511 10.5088 25.8104 10.5088 24.6218C6.61115 25.4776 5.79949 22.9106 5.79949 22.9106C5.17311 21.247 4.245 20.8194 4.245 20.8194C2.96929 19.94 4.33793 19.94 4.33793 19.94C5.75303 20.0351 6.49557 21.4135 6.49557 21.4135C7.74804 23.5998 9.76629 22.9821 10.5782 22.6017C10.6941 21.6748 11.0655 21.0332 11.4599 20.6767C8.3512 20.344 5.08047 19.1082 5.08047 13.5942C5.08047 12.0257 5.63687 10.7423 6.51851 9.74425C6.37941 9.38784 5.89213 7.91405 6.6579 5.94152C6.6579 5.94152 7.84097 5.56119 10.5085 7.41501C11.6506 7.10079 12.8284 6.94094 14.0116 6.9396C15.1947 6.9396 16.4007 7.10614 17.5143 7.41501C20.1822 5.56119 21.3653 5.94152 21.3653 5.94152C22.131 7.91405 21.6435 9.38784 21.5044 9.74425C22.4092 10.7423 22.9427 12.0257 22.9427 13.5942C22.9427 19.1082 19.672 20.32 16.5401 20.6767C17.0506 21.1282 17.4911 21.9837 17.4911 23.3385C17.4911 25.2635 17.4682 26.8084 17.4682 27.2836C17.4682 27.6639 17.7234 28.1157 18.4192 27.9731C23.9867 26.0714 27.9999 20.7004 27.9999 14.3549C28.0229 6.41664 21.7364 0 14.0116 0Z"
					/>
				</svg>
			</div>
		</svelte:fragment>
		<svelte:fragment slot="title">GitHub</svelte:fragment>
		<svelte:fragment slot="body">
			Allows you to view and create Pull Requests from GitButler.
		</svelte:fragment>
		{#if $user$?.github_access_token}
			<Button {disabled} kind="outlined" color="neutral" icon="bin-small" on:click={forgetGitHub}
				>Forget</Button
			>
		{:else}
			<Button {disabled} kind="filled" color="primary" on:click={gitHubStartOauth}>Authorize</Button
			>
		{/if}
	</SectionCard>
{/if}

<Modal bind:this={gitHubOauthModal} width="small" title="Authorize with GitHub">
	<div class="wrapper">
		<div class="step-section">
			<div class="step-default" />
			<div class="step-section__content">
				<p class="text-base-body-13">Copy the following verification code:</p>

				<div class="code-wrapper">
					<span class="text-head-20">
						{userCode}
					</span>
					<Button {disabled} icon="copy" on:click={() => copyToClipboard(userCode)}>
						Copy to Clipboard
					</Button>
				</div>
			</div>
		</div>

		<div class="step-section">
			<div class="step-default" />
			<div class="step-section__content">
				<p class="text-base-body-13">
					Go to the <Link href="https://github.com/login/device" target="_blank"
						>GitHub activation page</Link
					>
				</p>
			</div>
		</div>

		<div class="step-section">
			<div class="step-default" />
			<div class="step-section__content">
				<p class="text-base-body-13">
					Paste the code that you copied and follow the on-screen instructions.
				</p>
			</div>
		</div>

		<div class="step-section">
			<div class="step-last" />
			<div class="step-section__content">
				<Button
					kind="filled"
					color="primary"
					loading={isCheckingStatus}
					on:click={async () => {
						gitHubOauthCheckStatus(deviceCode);
					}}
				>
					Check the status and close
				</Button>
			</div>
		</div>
	</div>
</Modal>

<style lang="postcss">
	.wrapper {
		display: flex;
		flex-direction: column;
	}

	.step-section {
		display: flex;
		gap: var(--space-16);
		margin-left: var(--space-8);

		&:last-child {
			& .step-section__content {
				border-bottom: none;
				margin-bottom: 0;
				padding-bottom: var(--space-12);
			}
		}
	}

	.step-section__content {
		display: flex;
		flex-direction: column;
		width: 100%;
		gap: var(--space-12);
		margin-left: var(--space-8);
		padding-bottom: var(--space-20);
		margin-bottom: var(--space-20);
		border-bottom: 1px solid var(--clr-theme-container-outline-light);
	}

	.step-default {
		position: relative;
		width: 1px;
		border-right: 1px dashed var(--clr-theme-scale-ntrl-60);
		margin-top: var(--space-4);

		&::before {
			content: '';
			position: absolute;
			left: 50%;
			transform: translateX(-50%);
			width: var(--space-10);
			height: var(--space-10);
			background-color: var(--clr-theme-scale-ntrl-60);
			border-radius: 100%;
		}
	}

	.step-last {
		position: relative;
		width: 1px;
		margin-top: var(--space-4);

		&::before {
			content: '';
			position: absolute;
			left: 50%;
			transform: translateX(-50%);
			width: var(--space-10);
			height: var(--space-10);
			background-color: var(--clr-theme-scale-ntrl-60);
			border-radius: 100%;
		}
	}

	.icon-wrapper {
		position: relative;
	}

	.icon-wrapper__tick {
		position: absolute;
		display: flex;
		align-items: center;
		justify-content: center;
		bottom: calc(var(--space-4) * -1);
		right: calc(var(--space-4) * -1);
		background-color: var(--clr-theme-scale-ntrl-100);
		border-radius: 50px;
	}

	.code-wrapper {
		display: flex;
		gap: var(--space-10);
		align-items: center;
		align-self: flex-start;
		padding: var(--space-6) var(--space-6) var(--space-6) var(--space-8);
		border-radius: var(--radius-m);
		background-color: var(--clr-theme-container-pale);
		user-select: text;
	}
</style>
