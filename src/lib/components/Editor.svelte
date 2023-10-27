<script>
	import { onMount } from 'svelte';
	import { Trash } from 'svelte-heros-v2';
	import { LZMA as lzma } from 'lzma/src/lzma_worker-min';

	const size = 32;
	const spacing = 1;

	const generateEditor = () => {
		const editor = {
			idToColor: {
				0: 'clear',
				1: '#E9BCD1',
				2: '#34E2E2',
				3: '#FFFFFF'
			},
			board: Array(size)
				.fill(null)
				.map(() =>
					Array(size)
						.fill(null)
						.map((cell, i) => (i % 3) + 1)
				)
		};

		editor.colorToId = Object.fromEntries(
			Object.entries(editor.idToColor).map(([key, value]) => [value, key])
		);

		return editor;
	};

	const exportEditor = editor => JSON.stringify(lzma.compress(JSON.stringify(editor), 6));
	const importEditor = compressed => JSON.parse(lzma.decompress(JSON.parse(compressed)));

	// let currentEditor = generateEditor();
	let currentEditor = generateEditor();

	let canvas;
	let color;
	let pixelSize;
	let painting = false;
	let ctx;

	const paintEditor = () => {
		const { board } = currentEditor;

		for (let i = 0; i < board.length; i++) {
			const row = board[i];
			for (let j = 0; j < row.length; j++) {
				const cellColorId = row[j];
				let type = 'fill';
				if (cellColorId === 'clear') type = cellColorId;
				else ctx.fillStyle = currentEditor.idToColor[cellColorId];

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
		pixelSize = canvas.width / currentEditor.board.length;

		const editorAutoSave = localStorage.getItem('editorAutoSave');
		if (editorAutoSave) currentEditor = importEditor(editorAutoSave);
		paintEditor();
	});

	const activatePainting = () => {
		painting = true;
		const selectedColor = color.value;
		ctx.fillStyle = selectedColor;

		if (!currentEditor.colorToId[selectedColor]) {
			let previousId = Object.keys(currentEditor.idToColor);
			previousId = previousId[previousId.length - 1];
			const nextId = Number(previousId) + 1;

			currentEditor.idToColor[nextId] = selectedColor;
			currentEditor.colorToId[selectedColor] = nextId;
		}
	};

	const debounce = (fn, timeout) => {
		let timer;
		return (...args) => {
			clearTimeout(timer);
			timer = setTimeout(() => fn.apply(this, args), timeout);
		};
	};

	const autoSave = debounce(
		() => localStorage.setItem('editorAutoSave', exportEditor(currentEditor)),
		750
	);
	const deactivatePainting = () => {
		painting = false;
		autoSave();
	};

	const paint = event => {
		if (!painting) return;
		const startX = event.offsetX - (event.offsetX % pixelSize);
		const startY = event.offsetY - (event.offsetY % pixelSize);

		currentEditor.board[startX / pixelSize][startY / pixelSize] =
			currentEditor.colorToId[ctx.fillStyle];

		ctx.fillRect(
			startX - spacing,
			startY - spacing,
			pixelSize - spacing * 2,
			pixelSize - spacing * 2
		);
	};

	const clearCanvas = () => {
		if (!ctx) return;
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		currentEditor = generateEditor();
		paintEditor();
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
				class="m-0 aspect-square h-[5vw] w-[5vw]"
			/>
			<button
				class="editor-button mt-auto bg-white
				transition-all duration-300
				hover:border-4 hover:bg-red-400 hover:text-white"
				on:click={clearCanvas}
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
