<script lang="ts">
	import { onMount } from 'svelte';
	import type { BaubleModule } from 'bauble-runtime';
	import { Bauble, createOutputChannel, Storage } from '$lib';

	const outputChannel = createOutputChannel();

	const baubleOpts = {
		print(x: string) {
			outputChannel.print(x, false);
		},
		printErr(x: string) {
			outputChannel.print(x, true);
		}
	};

	let runtime: BaubleModule;

	let initialScript: string;

	onMount(async () => {
		try {
			const { default: InitializeWasm } = await import('bauble-runtime');
			runtime = await InitializeWasm(baubleOpts);
			initialScript =
				Storage.getScript() ?? runtime.FS.readFile('intro.janet', { encoding: 'utf8' });
		} catch (error) {
			console.error(error);
		}
	});
</script>

<div class="container">
	{#if runtime && initialScript}
		<Bauble
			{runtime}
			{outputChannel}
			{initialScript}
			focusable={false}
			canSave={true}
			size={{ width: 512, height: 512 }}
		/>
	{/if}
</div>

<style>
	.container {
		height: calc(100% - var(--header-height));
	}
</style>
