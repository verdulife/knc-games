<script lang="ts">
	import type { CardType } from '@/lib/types';
	import { onMount } from 'svelte';
	import Add from '@/assets/Add.svelte';
	import { initials } from '@/lib/initials';

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
		} else {
			main.scrollTo({
				top: scrollTop + speed,
				behavior: 'instant'
			});
		}

		if (playing) {
			speed = speed >= maxSpeed ? maxSpeed : speed + 0.4;
		} else {
			speed = speed <= 0 ? 0 : speed - 0.4;
		}

		if (!playing && speed < 0.4) {
			main.classList.add('snap-y');
		} else {
			main.classList.remove('snap-y');
		}

		if (!playing && speed === 0) return;
		requestAnimationFrame(scrollLoop);

		console.log({ scrollHeight, scrollTop, speed, playing });
	}

	function toggleRoulette() {
		playing = !playing;
		if (playing) scrollLoop();
	}

	function resetStorage() {
		const check = confirm('¿Quieres resetear la lista de temáticas?');
		if (!check) return;

		window.localStorage.setItem('knc-cards', JSON.stringify(initials));
		cards = JSON.parse(window.localStorage.getItem('knc-cards')!);
	}

	onMount(() => {
		const rawStorage = window.localStorage.getItem('knc-cards');
		const main = document.querySelector('main') as HTMLElement;

		if (rawStorage) {
			cards = JSON.parse(rawStorage);
		} else {
			window.localStorage.setItem('knc-cards', JSON.stringify(initials));
			cards = JSON.parse(window.localStorage.getItem('knc-cards')!);
		}

		setTimeout(() => {
			main.scrollTo({
				top: 100,
				behavior: 'smooth'
			});
		}, 100);
	});
</script>

<main
	class="flex h-svh snap-y snap-mandatory flex-col items-center gap-8 overflow-y-auto px-6 pt-8"
>
	{#if speed === 0}
		<button class="fixed top-0 left-0 z-30 cursor-pointer p-4 opacity-0" onclick={resetStorage}>
			<Add class="size-6" />
		</button>

		<button class="fixed top-0 right-0 z-30 cursor-pointer p-4" onclick={addItem}>
			<Add class="size-6" />
		</button>
	{/if}

	{#if cards.length > 0}
		{#each cards as card, i}
			<article
				class="bg-knc-red grid py-8 w-xl max-w-full shrink-0 snap-center place-content-center rounded-4xl text-center select-none"
				ondblclick={() => removeItem(i)}
			>
				<p class="text-2xl md:text-3xl font-semibold">
					{card}
				</p>
			</article>
		{/each}

		{#each cards as card, i}
			<article
				class="bg-knc-red grid py-8 w-xl max-w-full shrink-0 snap-center place-content-center rounded-4xl text-center select-none"
				ondblclick={() => removeItem(i)}
			>
				<p class="text-2xl md:text-3xl font-semibold">
					{card}
				</p>
			</article>
		{/each}

		<div
			class="bg-knc-dark/80 pointer-events-none fixed top-0 z-20 h-[calc(50%_-_64px)] w-full opacity-0 transition-opacity"
			class:opacity-100={speed === 0}
		></div>
		<div
			class="bg-knc-dark/80 pointer-events-none fixed bottom-0 z-20 h-[calc(50%_-_64px)] w-full opacity-0 transition-opacity"
			class:opacity-100={speed === 0}
		></div>

		<button
			class="bg-knc-blue fixed bottom-4 z-30 w-48 cursor-pointer rounded-full px-8 py-4 transition-opacity disabled:opacity-10"
			disabled={spinning}
			onclick={toggleRoulette}
		>
			{playing ? 'Parar' : 'Girar'}
		</button>
	{:else}
		<div class="mt-48 flex flex-col items-center gap-4 text-center">
			<h1 class="text-4xl font-semibold">¡No hay temáticas!</h1>
			<p>Clica en el botón de la esquina superior derecha para añadir una temática.</p>
		</div>
	{/if}
</main>

<style>
	main {
		scrollbar-width: none;
	}
</style>
