<script lang="ts">
	import { onMount } from 'svelte';
	import axios from 'axios';
	import '../app.css';
	let pokemons = [];
	let page = 1;
	let totalPages = 0;

	onMount(async () => {
		const response = await axios.get('https://pokeapi.co/api/v2/pokemon?offset=0&limit=18');
		pokemons = response.data.results;
		totalPages = Math.ceil(response.data.count / 20);
	});

	async function changePage(newPage) {
		page = newPage;
		const response = await axios.get(`https://pokeapi.co/api/v2/pokemon?offset=${(page - 1) * 20}&limit=18`);
		pokemons = response.data.results;
	}
</script>

<div class="container p-4 mx-auto">
	<h1 class="mb-8 text-4xl font-bold text-center">POKEDEX</h1>
	<div class="grid grid-cols-2 gap-4 md:grid-cols-4 lg:grid-cols-6 ">
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
			<button on:click={() => changePage(page - 1)} class="px-4 py-2 font-bold text-white bg-gray-500 rounded hover:bg-gray-700">Previous</button>
		{/if}
		<span class="p-2 mx-2">{page} / {totalPages}</span>
		{#if page < totalPages}
			<button on:click={() => changePage(page + 1)} class="px-4 py-2 font-bold text-white bg-gray-500 rounded hover:bg-gray-700">Next</button>
		{/if}
	</div>
</div>
