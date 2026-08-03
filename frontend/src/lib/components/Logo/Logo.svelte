<script lang="ts">
	import { page } from '$app/stores';
	import { browser } from '$app/environment';
	import optecMark from '$lib/assets/optec-mark.webp';

	interface Props {
		height?: number;
		width?: number;
	}

	let { height = 200, width = 200 }: Props = $props();

	let isDark = $state(false);

	$effect(() => {
		if (!browser) return;
		isDark = document.documentElement.classList.contains('dark');
		const observer = new MutationObserver(() => {
			isDark = document.documentElement.classList.contains('dark');
		});
		observer.observe(document.documentElement, { attributes: true, attributeFilter: ['class'] });
		return () => observer.disconnect();
	});

	const branding = $derived($page.data?.branding);
	const src = $derived(
		isDark && branding?.logo_dark_data
			? branding.logo_dark_data
			: branding?.logo_data ?? optecMark
	);
</script>

<img class="c" {height} {width} {src} alt={branding?.app_name ?? 'Optec GRC'} data-testid="logo-image" />
