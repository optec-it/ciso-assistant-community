<script lang="ts">
	import { page } from '$app/stores';
	import { browser } from '$app/environment';
	import optecMark from '$lib/assets/optec-mark.webp';
	import optecMarkWhite from '$lib/assets/optec-mark-white.png';

	interface Props {
		height?: number;
		width?: number;
		variant?: 'auto' | 'dark' | 'light';
	}

	let { height = 200, width = 200, variant = 'auto' }: Props = $props();

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

	const useDark = $derived(variant === 'dark' || (variant === 'auto' && isDark));

	const branding = $derived($page.data?.branding);
	const src = $derived(
		useDark && branding?.logo_dark_data
			? branding.logo_dark_data
			: useDark
				? (branding?.logo_dark_data ?? optecMarkWhite)
				: (branding?.logo_data ?? optecMark)
	);
</script>

<img class="c" {height} {width} {src} alt={branding?.app_name ?? 'Optec GRC'} data-testid="logo-image" />
