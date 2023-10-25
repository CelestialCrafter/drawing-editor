<script>
	import { setContext } from 'svelte';
	import { writable } from 'svelte/store';

	const editor = writable();

	setContext('editor', editor);
</script>

<slot />
