<script>
	import { onMount } from 'svelte';
	import { LZMA as lzma } from 'lzma/src/lzma_worker-min';

	export let canvas;
	export let color;
	export let drawingMode = 0;

	const size = 32;
	// @TODO add padding @CelestialCrafter
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

	export let currentEditor = generateEditor();

	let pixelSize;
	let painting = false;
	let ctx;

	export const paintEditor = () => {
		const { board } = currentEditor;
		ctx.clearRect(0, 0, canvas.width, canvas.height);

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

	const debounce = (fn, timeout) => {
		let timer;
		return (...args) => {
			clearTimeout(timer);
			timer = setTimeout(() => fn.apply(this, args), timeout);
		};
	};

	export const exportEditor = () => JSON.stringify(lzma.compress(JSON.stringify(currentEditor), 6));
	export const importEditor = compressed =>
		(currentEditor = JSON.parse(lzma.decompress(JSON.parse(compressed))));

	const autoSave = debounce(
		() => localStorage.setItem('editorAutoSave', exportEditor(currentEditor)),
		750
	);

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

	const deactivatePainting = () => {
		painting = false;
		autoSave();
	};

	const getBoardPosition = pos => (pos - (pos % pixelSize)) / pixelSize;

	const paint = ({ offsetX, offsetY }) => {
		if (!painting) return;
		const x = getBoardPosition(offsetX);
		const y = getBoardPosition(offsetY);
		const currentPixel = currentEditor.board[x][y];

		if (drawingMode === 2 && currentPixel !== 'clear')
			return (color.value = currentEditor.idToColor[currentPixel]);

		const startX = x * pixelSize;
		const startY = y * pixelSize;

		currentEditor.board[x][y] = currentEditor.colorToId[ctx.fillStyle];

		const type = drawingMode === 0 ? 'fill' : 'clear';

		ctx[`${type}Rect`](
			startX - spacing,
			startY - spacing,
			pixelSize - spacing * 2,
			pixelSize - spacing * 2
		);
	};

	export const clearCanvas = () => {
		if (!ctx) return;

		currentEditor = generateEditor();
		localStorage.removeItem('editorAutoSave');
		paintEditor();
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
</script>

<canvas
	bind:this={canvas}
	on:mousedown={activatePainting}
	on:mouseup={deactivatePainting}
	on:mousemove={paint}
	{...$$restProps}
/>
