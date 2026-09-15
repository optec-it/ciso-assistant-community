<script lang="ts">
	import type { ActionData, PageData } from './$types';
	import Logo from '$lib/components/Logo/Logo.svelte';
	import FormCard from './FormCard.svelte';
	import { m } from '$paraglide/messages';

	interface Props {
		data: PageData;
		form: ActionData;
	}

	let { data, form }: Props = $props();
</script>

<svelte:head>
	<title>{data.branding?.app_name ?? 'Optec GRC'} | {m.login()}</title>
</svelte:head>

<main class="flex items-center justify-center min-h-screen bg-surface-950">
	<div class="flex flex-col items-center w-full max-w-md px-4">
		<div class="flex justify-center mb-8 max-w-64">
			<Logo variant="dark" />
		</div>
		<h2 class="text-lg font-semibold mb-6">
			<span class="text-surface-400">Sign in to </span>
			<span class="bg-linear-to-r from-blue-400 to-cyan-400 bg-clip-text text-transparent">{data.branding?.app_name ?? 'Optec GRC'}</span>
		</h2>
		<FormCard {data} {form} />
	</div>
</main>
