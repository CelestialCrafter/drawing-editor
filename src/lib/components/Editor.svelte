<script>
	import { onMount } from 'svelte';
	import { Trash } from 'svelte-heros-v2';

	const editor = {
		colorMap: {
			0: 'clear',
			1: '#E9BCD1',
			2: '#34E2E2',
			3: '#FFFFFF'
		},
		board: Array(32).fill(
			Array(32)
				.fill(1)
				.map((cell, i) => (i % 3) + 1)
		)
	};

	let canvas;
	let color;
	let pixelSize;
	const spacing = 1;

	let painting = false;

	let ctx;

	const paintEditor = () => {
		const { board } = editor;

		for (let i = 0; i < board.length; i++) {
			const row = board[i];
			for (let j = 0; j < row.length; j++) {
				const cellColor = row[j];
				let type = 'fill';
				if (cellColor === 'clear') type = cellColor;
				else ctx.fillStyle = editor.colorMap[cellColor];

				ctx[`${type}Rect`](
					i * pixelSize - spacing,
					j * pixelSize - spacing,
					pixelSize - spacing * 2,
					pixelSize - spacing * 2
				);
			}
		}
	};

	onMount(() => {
		ctx = canvas?.getContext('2d');

		canvas.width = canvas.offsetWidth;
		canvas.height = canvas.offsetHeight;
		pixelSize = canvas.width / editor.board.length;

		paintEditor();
	});

	const activatePainting = () => {
		painting = true;
		ctx.fillStyle = color.value;
	};

	const deactivatePainting = () => (painting = false);

	const paint = event => {
		if (!painting) return;
		const startX = event.offsetX - (event.offsetX % pixelSize);
		const startY = event.offsetY - (event.offsetY % pixelSize);

		ctx.fillRect(
			startX - spacing,
			startY - spacing,
			pixelSize - spacing * 2,
			pixelSize - spacing * 2
		);
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
				class="m-0 aspect-square h-[80px] w-[80px]"
			/>
			<button
				class="editor-button mt-auto bg-white
				transition-all duration-300
				hover:border-4 hover:bg-red-400 hover:text-white"
				on:click={() => ctx && (ctx.clearRect(0, 0, canvas.width, canvas.height) || paintEditor())}
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

<style>
	input[type='color'] {
		&::-webkit-color-swatch,
		&::-webkit-color-swatch-wrapper,
		& {
			@apply overflow-hidden rounded-2xl border-none p-0;
		}

		&::-moz-color-swatch {
			@apply border-none;
		}
	}
	.editor-button {
		@apply flex aspect-square h-[80px] w-[80px] items-center justify-center rounded-2xl border-gray-100 shadow-lg;
	}
</style>
