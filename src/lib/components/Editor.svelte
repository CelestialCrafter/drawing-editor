<script>
	import { getContext, onMount } from 'svelte';

	// const editor = getContext('editor');

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

<main class="flex w-screen h-screen justify-center items-center flex-row">
	<div class="flex flex-row gap-3"> <!-- okay this is a dirty css workaround but itll do -->
		<div id="controls" class="w-fit shadow-2xl bg-white/30 p-3 rounded-2xl flex flex-col gap-3">
			<input type="color" id="color" bind:this={color} class="w-[80px] h-[80px] aspect-square rounded-lg m-0 border-none"/>
			<button class="editor-button bg-white mt-auto transition-all hover:text-white hover:border-4 hover:bg-red-400"><Trash size="30"/></button>
		</div>
		<canvas bind:this={canvas} class="shadow-2xl w-[40vw] h-[vh] aspect-square rounded-2xl bg-white"></canvas>
	</div>
</main>

<style lang="postcss">
	.editor-button {
		@apply w-[80px] h-[80px] aspect-square rounded-2xl shadow-lg border-gray-100 flex justify-center items-center
	}
</style>