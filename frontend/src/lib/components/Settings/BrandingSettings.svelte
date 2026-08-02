<script lang="ts">
	import { m } from '$paraglide/messages';
	import { enhance } from '$app/forms';

	let { data } = $props();

	let appName = $state(data.brandingSettings?.app_name ?? 'Optec GRC');
	let logoData = $state(data.brandingSettings?.logo_data ?? null);
	let faviconData = $state(data.brandingSettings?.favicon_data ?? null);
	let primaryColor = $state(data.brandingSettings?.primary_color ?? '#006aff');
	let accentColor = $state(data.brandingSettings?.accent_color ?? '#ff8a5b');

	const MAX_LOGO_KB = 500;
	const MAX_FAVICON_KB = 100;
	const ALLOWED_TYPES = ['image/png', 'image/jpeg', 'image/webp'];

	function handleFileUpload(
		event: Event,
		maxKb: number,
		setter: (value: string | null) => void
	) {
		const input = event.target as HTMLInputElement;
		const file = input.files?.[0];
		if (!file) return;

		if (!ALLOWED_TYPES.includes(file.type)) {
			alert(`File must be PNG, JPEG, or WebP.`);
			input.value = '';
			return;
		}
		if (file.size > maxKb * 1024) {
			alert(`File exceeds ${maxKb} KB limit.`);
			input.value = '';
			return;
		}

		const reader = new FileReader();
		reader.onload = () => {
			setter(reader.result as string);
		};
		reader.readAsDataURL(file);
	}
</script>

<form method="POST" action="?/branding" use:enhance>
	<div class="space-y-6">
		<div>
			<label class="label" for="app_name">
				<span class="font-semibold">{m.appName()}</span>
			</label>
			<input
				class="input px-3 py-2"
				type="text"
				id="app_name"
				name="app_name"
				bind:value={appName}
				maxlength="100"
			/>
		</div>

		<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
			<div>
				<label class="label" for="logo_upload">
					<span class="font-semibold">{m.uploadLogo()}</span>
					<span class="text-sm text-surface-500"> (max {MAX_LOGO_KB} KB, PNG/JPEG/WebP)</span>
				</label>
				<input
					class="input px-3 py-2"
					type="file"
					id="logo_upload"
					accept="image/png,image/jpeg,image/webp"
					onchange={(e) => handleFileUpload(e, MAX_LOGO_KB, (v) => (logoData = v))}
				/>
				{#if logoData}
					<div class="mt-3 flex items-center gap-3">
						<img src={logoData} alt="Logo preview" class="h-16 rounded" />
						<button
							type="button"
							class="btn btn-sm preset-outlined-error-500"
							onclick={() => (logoData = null)}
						>
							<i class="fa-solid fa-trash"></i> Remove
						</button>
					</div>
				{/if}
				<input type="hidden" name="logo_data" value={logoData ?? ''} />
			</div>

			<div>
				<label class="label" for="favicon_upload">
					<span class="font-semibold">{m.uploadFavicon()}</span>
					<span class="text-sm text-surface-500"> (max {MAX_FAVICON_KB} KB, PNG/JPEG/WebP)</span>
				</label>
				<input
					class="input px-3 py-2"
					type="file"
					id="favicon_upload"
					accept="image/png,image/jpeg,image/webp,image/x-icon"
					onchange={(e) => handleFileUpload(e, MAX_FAVICON_KB, (v) => (faviconData = v))}
				/>
				{#if faviconData}
					<div class="mt-3 flex items-center gap-3">
						<img src={faviconData} alt="Favicon preview" class="h-8 rounded" />
						<button
							type="button"
							class="btn btn-sm preset-outlined-error-500"
							onclick={() => (faviconData = null)}
						>
							<i class="fa-solid fa-trash"></i> Remove
						</button>
					</div>
				{/if}
				<input type="hidden" name="favicon_data" value={faviconData ?? ''} />
			</div>
		</div>

		<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
			<div>
				<label class="label" for="primary_color">
					<span class="font-semibold">{m.primaryColor()}</span>
				</label>
				<div class="flex items-center gap-3">
					<input
						type="color"
						id="primary_color"
						bind:value={primaryColor}
						class="h-10 w-14 cursor-pointer rounded"
					/>
					<input
						class="input px-3 py-2 w-32"
						type="text"
						name="primary_color"
						bind:value={primaryColor}
						pattern="#[0-9a-fA-F]{6}"
					/>
				</div>
			</div>

			<div>
				<label class="label" for="accent_color">
					<span class="font-semibold">{m.accentColor()}</span>
				</label>
				<div class="flex items-center gap-3">
					<input
						type="color"
						id="accent_color"
						bind:value={accentColor}
						class="h-10 w-14 cursor-pointer rounded"
					/>
					<input
						class="input px-3 py-2 w-32"
						type="text"
						name="accent_color"
						bind:value={accentColor}
						pattern="#[0-9a-fA-F]{6}"
					/>
				</div>
			</div>
		</div>

		<div class="flex gap-3">
			<button type="submit" class="btn preset-filled-primary-500">
				<i class="fa-solid fa-floppy-disk mr-2"></i>
				{m.save()}
			</button>
			<button type="submit" formaction="?/brandingReset" class="btn preset-outlined-surface-500">
				<i class="fa-solid fa-rotate-left mr-2"></i>
				{m.resetToDefaults()}
			</button>
		</div>
	</div>
</form>
