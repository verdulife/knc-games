<script lang="ts">
	import type { CardType } from '@/lib/types';
	import { onMount } from 'svelte';
	import Add from '@/assets/Add.svelte';

	let cards: CardType[] = $state([]);

	function removeItem(index: number) {
		const check = confirm('¿Quieres borrar esta temática?');
		if (!check) return;

		cards = cards.filter((_, i) => i !== index);
		window.localStorage.setItem('knc-cards', JSON.stringify(cards));
	}

	function addItem() {
		const check = prompt('¿Que nombre tiene la temática?');
		if (!check) return;

		cards.push(check);
		window.localStorage.setItem('knc-cards', JSON.stringify(cards));
	}

	const maxSpeed = 40;
	let speed = $state(0);
	let playing = $state(false);
	let spinning = $derived(speed > 0 && speed < maxSpeed);

	function scrollLoop() {
		const main = document.querySelector('main') as HTMLElement;
		const { scrollHeight, scrollTop, clientHeight } = main;

		if (scrollTop + speed >= scrollHeight - clientHeight) {
			main.scrollTo({
				top: 0,
				behavior: 'instant'
			});
		}

		main.scrollBy({
			top: speed,
			behavior: 'instant'
		});

		if (playing) {
			speed = speed >= maxSpeed ? maxSpeed : speed + 0.2;
		} else {
			speed = speed <= 0 ? 0 : speed - 0.4;
		}

		if (!playing && speed === 0) {
			main.classList.add('snap-y');
			main.classList.add('scroll-smooth');
			return;
		} else {
			main.classList.remove('snap-y');
			main.classList.remove('scroll-smooth');
		}

		requestAnimationFrame(scrollLoop);

		console.log({ scrollHeight, scrollTop, speed, playing });
	}

	function toggleRoulette() {
		playing = !playing;
		if (playing) scrollLoop();
	}

	onMount(() => {
		const rawStorage = window.localStorage.getItem('knc-cards');
		const main = document.querySelector('main') as HTMLElement;

		if (rawStorage) cards = JSON.parse(rawStorage);
		else window.localStorage.setItem('knc-cards', JSON.stringify(cards));

		setTimeout(() => {
			main.scrollBy({
				top: 10,
				behavior: 'smooth'
			});
		}, 100);
	});
</script>

<main
	class="flex h-svh snap-y snap-mandatory flex-col items-center gap-12 overflow-y-auto scroll-smooth bg-black pb-12 text-white"
>
	<button class="fixed top-0 right-0 z-30 cursor-pointer p-4" onclick={addItem}>
		<Add class="size-8" />
	</button>

	{#if cards.length > 0}
		{#each cards as card, i}
			<article
				class="grid h-32 w-xl shrink-0 origin-right snap-center place-content-center border-2 bg-white text-center text-black select-none"
				ondblclick={() => removeItem(i)}
			>
				<p class="text-6xl font-semibold">
					{card}
				</p>
			</article>
		{/each}

		{#each cards as card, i}
			<article
				class="grid h-32 w-xl shrink-0 origin-right snap-center place-content-center border-2 bg-white text-center text-black select-none"
				ondblclick={() => removeItem(i)}
			>
				<p class="text-6xl font-semibold">
					{card}
				</p>
			</article>
		{/each}

		{#each cards as card, i}
			<article
				class="grid h-32 w-xl shrink-0 origin-right snap-center place-content-center border-2 bg-white text-center text-black select-none"
				ondblclick={() => removeItem(i)}
			>
				<p class="text-6xl font-semibold">
					{card}
				</p>
			</article>
		{/each}
	{/if}

	<div
		class="pointer-events-none fixed top-0 z-20 h-[calc(50%_-_64px)] w-full bg-black/80 opacity-0 transition-opacity"
		class:opacity-100={speed === 0}
	></div>
	<div
		class="pointer-events-none fixed bottom-0 z-20 h-[calc(50%_-_64px)] w-full bg-black/80 opacity-0 transition-opacity"
		class:opacity-100={speed === 0}
	></div>

	<button
		class="fixed bottom-4 z-30 w-48 cursor-pointer rounded-full bg-blue-400 px-8 py-4 transition-opacity disabled:opacity-10"
		disabled={spinning}
		onclick={toggleRoulette}
	>
		{playing ? 'Parar' : 'Girar'}
	</button>
</main>

<style>
	main {
		scrollbar-width: none;
	}
</style>
