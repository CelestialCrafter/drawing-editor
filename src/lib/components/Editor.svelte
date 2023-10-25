<script>
	import { getContext, onMount } from 'svelte';

	const editor = getContext('editor');

	let canvas = null;
	let color = null;
	let ctx = null;

	canvas.getContext('2d');
	let painting = false;
</script>

<input type="color" id="color" bind:this={color} />
<button>Clear</button>

<canvas bind:this={canvas}></canvas>
