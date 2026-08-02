<script lang="ts">
	import { page } from '$app/stores';
	import optecMark from '$lib/assets/optec-mark.webp';

	interface Props {
		height?: number;
		width?: number;
	}

	let { height = 200, width = 200 }: Props = $props();

	const branding = $derived($page.data?.branding);
	const src = $derived(branding?.logo_data ?? optecMark);
</script>

<img class="c" {height} {width} {src} alt={branding?.app_name ?? 'Optec GRC'} data-testid="logo-image" />
