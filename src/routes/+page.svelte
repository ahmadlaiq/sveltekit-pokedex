<script lang="ts">
	import { onMount } from 'svelte';
	import axios from 'axios';

	let pokemons = [];
	let allPokemons = [];
	let page = 1;
	let totalPages = 0;
	let searchTerm = '';

	onMount(async () => {
		const response = await axios.get('https://pokeapi.co/api/v2/pokemon?offset=0&limit=1000');
		allPokemons = response.data.results;
		totalPages = Math.ceil(allPokemons.length / 18);
		updateDisplayedPokemons();
	});

	function updateDisplayedPokemons() {
		let filteredPokemons = allPokemons;
		if (searchTerm) {
			filteredPokemons = allPokemons.filter(
				(pokemon) =>
					pokemon.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
					pokemon.url.split('/')[6].includes(searchTerm)
			);
		}
		totalPages = Math.ceil(filteredPokemons.length / 18);
		page = 1;
		pokemons = filteredPokemons.slice(0, 18);
	}

	function debounce(func, timeout = 300) {
		let timer;
		return (...args) => {
			clearTimeout(timer);
			timer = setTimeout(() => {
				func.apply(this, args);
			}, timeout);
		};
	}

	const debouncedSearch = debounce(() => {
		updateDisplayedPokemons();
	});

	function changePage(newPage) {
		page = newPage;
		let filteredPokemons = allPokemons;
		if (searchTerm) {
			filteredPokemons = allPokemons.filter(
				(pokemon) =>
					pokemon.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
					pokemon.url.split('/')[6].includes(searchTerm)
			);
		}
		const startIndex = (page - 1) * 18;
		pokemons = filteredPokemons.slice(startIndex, startIndex + 18);
	}
</script>

<div class="container p-4 mx-auto">
	<h1 class="mb-8 text-4xl font-bold text-center">POKEDEX</h1>
	<div class="mb-5">
		<div class="flex justify-center mb-3">
			<input
				type="text"
				class="w-3/4 px-3 py-2 leading-tight text-gray-700 border rounded shadow appearance-none focus:outline-none focus:shadow-outline"
				placeholder="Search Pokémon by name or number..."
				bind:value={searchTerm}
				on:input={debouncedSearch}
			/>
		</div>
	</div>
	<div class="grid grid-cols-2 gap-4 mt-3 md:grid-cols-4 lg:grid-cols-6">
		{#each pokemons as pokemon}
			<div class="p-4 bg-white rounded-lg shadow-md hover:bg-gray-600 hover:text-white">
				<img
					src={`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${pokemon.url.split('/')[6]}.png`}
					alt={pokemon.name}
					class="mx-auto"
				/>
				<h3 class="text-lg font-semibold text-start">
					{pokemon.url.split('/')[6]}
				</h3>
				<h2 class="text-lg text-center capitalize">{pokemon.name}</h2>
			</div>
		{/each}
	</div>
	<div class="flex justify-center mt-4">
		{#if page > 1}
			<button
				on:click={() => changePage(page - 1)}
				class="px-4 py-2 font-bold text-white bg-gray-500 rounded hover:bg-gray-700"
				>Previous</button
			>
		{/if}
		<span class="p-2 mx-2">{page} / {totalPages}</span>
		{#if page < totalPages}
			<button
				on:click={() => changePage(page + 1)}
				class="px-4 py-2 font-bold text-white bg-gray-500 rounded hover:bg-gray-700">Next</button
			>
		{/if}
	</div>
</div>

<!-- Styling -->
<style>
	@import 'tailwindcss/tailwind.css';
</style>
