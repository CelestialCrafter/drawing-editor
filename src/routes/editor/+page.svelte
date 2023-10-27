<script>
	import {
		NoSymbol,
		Pencil,
		EyeDropper,
		ArrowDownOnSquare,
		ArrowUpOnSquare,
		Trash
	} from 'svelte-heros-v2';
	import Editor from './Editor.svelte';

	let color;
	let editor;
	let drawingMode;
</script>

<main class="flex h-screen w-screen flex-row items-center justify-center">
	<div class="flex flex-row gap-3">
		<!-- okay this is a dirty css workaround but itll do -->
		<div id="controls" class="flex w-fit flex-col gap-3 rounded-2xl bg-white/30 p-3 shadow-2xl">
			<input type="color" id="color" bind:this={color} class="m-0 aspect-square h-[5vw] w-[5vw]" />

			{#if editor}
				<!-- eslint-disable no-alert -->
				<button
					class="editor-button bg-white hover:bg-purple-400"
					on:click={() =>
						editor.importEditor(prompt('Input your Editor Code')) && editor.paintEditor()}
				>
					<ArrowDownOnSquare size="30" />
				</button>
				<button
					class="editor-button bg-white hover:bg-purple-400"
					on:click={() => alert(`Editor Code:\n${editor.exportEditor()}`)}
				>
					<ArrowUpOnSquare size="30" />
				</button>
				<!-- eslint-enable no-alert -->
				<button
					class="editor-button bg-white hover:bg-purple-400"
					on:click={() => (drawingMode === 2 ? (drawingMode = 0) : drawingMode++)}
				>
					{#if drawingMode === 0}
						<Pencil size="30" />
					{:else if drawingMode === 1}
						<NoSymbol size="30" />
					{:else}
						<EyeDropper size="30" />
					{/if}
				</button>

				<button
					class="editor-button mt-auto bg-white
					transition-all duration-300
					hover:border-4 hover:bg-red-400 hover:text-white"
					on:click={editor.clearCanvas}
				>
					<Trash size="30" />
				</button>
			{/if}
		</div>
		<Editor
			bind:this={editor}
			bind:drawingMode
			{color}
			class="aspect-square h-[vh] w-[40vw] rounded-2xl bg-white shadow-2xl"
		/>
	</div>
</main>

<style>
	input[type='color'] {
		&::-webkit-color-swatch,
		&::-webkit-color-swatch-wrapper,
		& {
			@apply overflow-hidden rounded-2xl border-none p-0 shadow-lg;
		}

		&::-moz-color-swatch {
			@apply border-none;
		}
	}
	.editor-button {
		@apply flex aspect-square h-[5vw] w-[5vw] items-center justify-center rounded-2xl border-gray-100 shadow-lg;
	}
</style>
