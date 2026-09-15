<script lang="ts">
	import { run } from 'svelte/legacy';
	import { onMount } from 'svelte';

	import SuperForm from '$lib/components/Forms/Form.svelte';
	import TextField from '$lib/components/Forms/TextField.svelte';
	import { loginSchema } from '$lib/utils/schemas';

	import { page } from '$app/state';
	import { redirectToProvider } from '$lib/allauth.js';
	import { getSecureRedirect } from '$lib/utils/helpers';
	import { zod4 as zod } from 'sveltekit-superforms/adapters';
	import MfaAuthenticateModal from './mfa/components/MFAAuthenticateModal.svelte';
	import { m } from '$paraglide/messages';
	import {
		getModalStore,
		type ModalComponent,
		type ModalSettings,
		type ModalStore
	} from '$lib/components/Modals/stores';

	interface Props {
		data: any;
		form: any;
	}

	let { data, form }: Props = $props();

	const modalStore: ModalStore = getModalStore();

	function modalMFAAuthenticate(): void {
		const mfaTypes: string[] = form?.mfaFlow?.types ?? ['totp'];
		const modalComponent: ModalComponent = {
			ref: MfaAuthenticateModal,
			props: {
				_form: data.mfaAuthenticateForm,
				formAction: '?/mfaAuthenticate',
				mfaTypes
			}
		};
		const modal: ModalSettings = {
			type: 'component',
			component: modalComponent,
			title: m.mfaAuthenticateTitle(),
			body: m.enterCodeGeneratedByApp()
		};
		modalStore.trigger(modal);
	}

	run(() => {
		form && form.mfaFlow ? modalMFAAuthenticate() : null;
	});

	function getSSOCallbackURL(callbackURL: string): string {
		const url = new URL(callbackURL);
		const next = getSecureRedirect(page.url.searchParams.get('next')) || '/';

		url.pathname = '/sso/authenticate';
		url.search = '';
		url.searchParams.set('next', next);
		return url.toString();
	}

	function triggerSSO(): void {
		redirectToProvider(
			data.SSOInfo.sp_entity_id,
			getSSOCallbackURL(data.SSOInfo.callback_url),
			'login'
		);
	}

	const autoSSO = $derived(data.SSOInfo?.is_enabled && page.url.searchParams.has('sso'));

	onMount(() => {
		if (autoSSO) triggerSSO();
	});
</script>

<div
	class="flex flex-col w-full p-8 rounded-2xl shadow-2xl bg-surface-900 border border-surface-800"
>
	<div data-testid="login" class="flex flex-col w-full items-center space-y-4">
		{#if autoSSO}
			<div class="bg-secondary-500/20 text-secondary-400 px-6 py-5 rounded-full text-3xl">
				<i class="fa-solid fa-circle-notch fa-spin"></i>
			</div>
			<p class="text-center text-surface-400 text-sm">{m.loginSSO()}…</p>
		{:else}
			<div class="w-full">
				<SuperForm
					class="flex flex-col space-y-4"
					data={data?.form}
					dataType="form"
					validators={zod(loginSchema)}
					action="?/login&next={page.url.searchParams.get('next') || '/'}"
				>
					{#snippet children({ form })}
						<TextField type="email" {form} field="username" label={m.email()} />
						<TextField type="password" {form} field="password" label={m.password()} />
						<p class="">
							<button
								class="btn w-full font-semibold text-surface-950 rounded-lg py-3"
								style="background-color: var(--color-secondary-500);"
								data-testid="login-btn"
								type="submit">{m.login()}</button
							>
						</p>
					{/snippet}
				</SuperForm>
			</div>
			{#if data.SSOInfo.is_enabled}
				<div class="flex items-center justify-center w-full space-x-3 py-1">
					<hr class="flex-1 border-surface-700" />
					<span class="text-surface-500 text-sm">{m.or()}</span>
					<hr class="flex-1 border-surface-700" />
				</div>
				<button
					class="btn bg-surface-800 hover:bg-surface-700 text-surface-200 font-semibold w-full border border-surface-700 rounded-lg py-3"
					onclick={triggerSSO}
				>
					<i class="fa-brands fa-microsoft mr-2"></i>
					{m.loginSSO()}
				</button>
			{/if}
			<p class="text-xs text-surface-500 text-center pt-2">
				Local accounts skip multi-factor authentication. Sign-ins are recorded in the audit log.
			</p>
		{/if}
	</div>
</div>
