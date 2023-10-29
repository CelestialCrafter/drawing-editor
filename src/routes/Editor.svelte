<script>
	import { onMount } from 'svelte';
	import { LZMA as lzma } from 'lzma/src/lzma_worker-min';

	export let canvas;
	export let color;
	export let drawingMode = 0;
	export let spacing = 0;

	export let axisMultiplier = 1.5;
	export let axis = 'vertical';

	const horizontalMultiplier = Math.max((axis === 'horizontal') * axisMultiplier, 1);
	const verticalMultiplier = Math.max((axis === 'vertical') * axisMultiplier, 1);

	const size = 32;
	// @TODO add padding @CelestialCrafter

	const generateEditor = () => {
		const editor = {
			idToColor: {
				0: 'clear'
			},
			board: Array(size)
				.fill(null)
				.map(() =>
					Array(size)
						.fill(null)
						.map(() => 0)
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

	const calculatePixelPosition = (first, second, type) => {
		const alternatePixel = type === axis;
		return alternatePixel
			? (first + (second % 2)) * (pixelSize * axisMultiplier) - spacing
			: first * pixelSize - spacing;
	};

	export const paintEditor = () => {
		const { board } = currentEditor;
		ctx.clearRect(0, 0, canvas.width, canvas.height);

		for (let i = 0; i < board.length; i++) {
			const row = board[i];
			for (let j = 0; j < row.length; j++) {
				const cellColorId = row[j];
				let type = 'fill';
				if (cellColorId === 0) type = 'clear';
				else ctx.fillStyle = currentEditor.idToColor[cellColorId];

				ctx[`${type}Rect`](
					calculatePixelPosition(i, j, 'horizontal'),
					calculatePixelPosition(j, i, 'vertical'),
					pixelSize * horizontalMultiplier - spacing * 2,
					pixelSize * verticalMultiplier - spacing * 2
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

		currentEditor.board[x][y] = currentEditor.colorToId[ctx.fillStyle];

		const type = drawingMode === 0 ? 'fill' : 'clear';

		ctx[`${type}Rect`](
			x * pixelSize - spacing,
			(y + (x % 2)) * pixelSize - spacing,
			pixelSize * horizontalMultiplier - spacing * 2,
			pixelSize * verticalMultiplier - spacing * 2
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
