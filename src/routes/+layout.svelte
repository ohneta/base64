<script lang="ts">
	import '../app.css';
	import Header from './Header.svelte';
	import Footer from './Footer.svelte';

	let { children } = $props();
</script>

<div class="app">
	<Header />

	<main class="w-full p-5">
		{@render children()}
	</main>

	<Footer />
</div>

<style>
</style>
