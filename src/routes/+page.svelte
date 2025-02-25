<script lang="ts">
	import { onMount } from 'svelte';
	import axios from 'axios';

	let pokemons = [];
	let allPokemons = [];
	let page = 1;
	let totalPages = 0;
	let searchTerm = '';
	let selectedPokemon = null;
    let showModal = false;

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
	
	async function showPokemonDetails(pokemonUrl) {
        const id = pokemonUrl.split('/')[6];
        const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`);
        selectedPokemon = response.data;
        showModal = true;
    }

	function closeModal() {
        showModal = false;
        selectedPokemon = null;
    }
</script>

<div class="container p-4 mx-auto">
	<h1 class="mb-8 text-4xl font-bold text-center">POKEDEX</h1>
	<div class="mb-5">
		<div class="flex justify-center mb-3">
			<input
				type="text"
				class="w-3/4 px-3 py-2 leading-tight text-gray-700 border border-gray-400 rounded shadow appearance-none focus:outline-none focus:shadow-outline"
				placeholder="Search Pokémon..."
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
					class="mx-auto transition-transform duration-500 ease-in-out hover:scale-110"
				/>
				<h3 class="text-lg font-semibold text-start">
					{pokemon.url.split('/')[6]}
				</h3>
				<h2 class="mb-4 text-lg text-center capitalize">{pokemon.name}</h2>
				<button on:click={() => showPokemonDetails(pokemon.url)} class="text-white inline-flex w-full justify-center bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
                    Detail
                </button>
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

{#if showModal && selectedPokemon}
    <div class="fixed inset-0 z-50 flex items-center justify-center overflow-auto bg-black bg-opacity-50">
        <div class="relative w-3/4 max-w-md p-8 bg-white rounded-lg">
            <button 
                class="absolute text-gray-600 top-2 right-2 hover:text-gray-800"
                on:click={closeModal}
            >
			<div class="inline-flex items-center justify-center w-8 h-8 text-sm text-gray-400 bg-transparent rounded-lg hover:bg-gray-200 hover:text-gray-900 ms-auto dark:hover:bg-gray-600 dark:hover:text-white" data-modal-toggle="select-modal">
				<svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
					<path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
				</svg>
				<span class="sr-only">Close modal</span>
			</div>
            </button>
            <h2 class="mb-4 text-2xl font-bold text-center capitalize">{selectedPokemon.name}</h2>
            <img 
                src={selectedPokemon.sprites.other['official-artwork'].front_default} 
                alt={selectedPokemon.name}
                class="w-48 h-48 mx-auto mb-4 transition-transform duration-500 ease-in-out hover:scale-110"
            />
            <div class="grid grid-cols-2 gap-2">
                <p><strong>Height:</strong> {selectedPokemon.height / 10} m</p>
                <p><strong>Weight:</strong> {selectedPokemon.weight / 10} kg</p>
                <p><strong>Base Experience:</strong> {selectedPokemon.base_experience}</p>
                <p><strong>ID:</strong> {selectedPokemon.id}</p>
            </div>
            <div class="mt-4 capitalize">
                <strong>Types:</strong>
                {#each selectedPokemon.types as type}
                    <span class="inline-block px-3 py-1 mr-2 text-sm font-semibold text-white bg-gray-600 rounded-full">
                        {type.type.name}
                    </span>
                {/each}
            </div>
            <div class="mt-4 capitalize">
                <strong>Abilities:</strong>
                <ul class="list-disc list-inside">
                    {#each selectedPokemon.abilities as ability}
                        <li>{ability.ability.name}</li>
                    {/each}
                </ul>
            </div>
        </div>
    </div>
{/if}

<!-- Styling -->
<style>
	@import 'tailwindcss/tailwind.css';
</style>
