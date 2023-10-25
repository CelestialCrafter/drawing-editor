<script>
	import { setContext, writable } from 'svelte';

	const editor = writable();

	setContext('editor', editor);
</script>

<slot />
