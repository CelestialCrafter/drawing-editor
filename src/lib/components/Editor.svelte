<script>
	import { getContext, onMount } from 'svelte';

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

<section class="flex h-full">
	<div class="flex-1">
		<input type="color" bind:this={color} />
		<button on:click={() => ctx?.clearRect(0, 0, canvas.width, canvas.height)}>Clear</button>
	</div>

	<canvas
		class="h-full w-full"
		bind:this={canvas}
		on:mousedown={activatePainting}
		on:mouseup={deactivatePainting}
		on:mousemove={paint}
	/>
</section>
