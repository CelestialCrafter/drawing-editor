<script>
	import { getContext, onMount } from 'svelte';
	import { Trash } from 'svelte-heros-v2';

	const editor = getContext('editor');

	let canvas = null;
	let color = null;
	let painting = false;

	let ctx;

	onMount(() => {
		ctx = canvas?.getContext('2d');

		canvas.width = canvas.offsetWidth;
		canvas.height = canvas.offsetHeight;
	});

	const activatePainting = () => {
		painting = true;
		ctx.strokeStyle = color.value;
	};

	const deactivatePainting = () => {
		painting = false;
		ctx.stroke();
		ctx.beginPath();
	};

	const paint = event => {
		if (!painting) return;
		ctx.lineWidth = 5;
		ctx.lineCap = 'round';
		ctx.lineTo(event.offsetX, event.offsetY);
		ctx.stroke();
	};
</script>

<main class="flex h-screen w-screen flex-row items-center justify-center">
	<div class="flex flex-row gap-3">
		<!-- okay this is a dirty css workaround but itll do -->
		<div id="controls" class="flex w-fit flex-col gap-3 rounded-2xl bg-white/30 p-3 shadow-2xl">
			<input
				type="color"
				id="color"
				bind:this={color}
				class="m-0 aspect-square h-[80px] w-[80px] rounded-lg border-none"
			/>
			<button
				class="editor-button mt-auto bg-white transition-all hover:border-4 hover:bg-red-400 hover:text-white"
				on:click={() => ctx?.clearRect(0, 0, canvas.width, canvas.height)}
			>
				<Trash size="30" />
			</button>
		</div>
		<canvas
			bind:this={canvas}
			on:mousedown={activatePainting}
			on:mouseup={deactivatePainting}
			on:mousemove={paint}
			class="aspect-square h-[vh] w-[40vw] rounded-2xl bg-white shadow-2xl"
		></canvas>
	</div>
</main>

<style lang="postcss">
	.editor-button {
		@apply flex aspect-square h-[80px] w-[80px] items-center justify-center rounded-2xl border-gray-100 shadow-lg;
	}
</style>
