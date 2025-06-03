<script lang="ts">
	let { children, ondblclick = $bindable() } = $props();
</script>

<article class="w-xl border-2 bg-white p-6 text-center text-black" {ondblclick}>
	<p class="text-6xl font-semibold">
		{@render children()}
	</p>
</article>
