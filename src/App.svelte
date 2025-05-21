<script>
  import QuakMohle from './QuakMohle.svelte';
  import Currency from './Currency.svelte';
  import Pacman from './Pacman.svelte';

  let currentPage = 'home';
</script>

{#if currentPage === 'home'}
  <main class="text-center p-4">
    <h1 class="text-2xl font-bold mb-6">🚀 willkommen</h1>
    <button class="bg-green-500 text-white px-4 py-2 rounded mb-4" on:click={() => currentPage = 'quak'}>
      🐸 quak-mohle spiel starten
    </button>
    <br />
    <button class="bg-yellow-500 text-white px-4 py-2 rounded mb-4" on:click={() => currentPage = 'pacman'}>
      👻 pacman spielen
    </button>
    <br />
    <button class="bg-blue-500 text-white px-4 py-2 rounded" on:click={() => currentPage = 'currency'}>
      💱 währungsrechner öffnen
    </button>
  </main>
{:else if currentPage === 'quak'}
  <QuakMohle on:back={() => currentPage = 'home'} />
{:else if currentPage === 'pacman'}
  <Pacman on:back={() => currentPage = 'home'} />
{:else if currentPage === 'currency'}
  <Currency on:back={() => currentPage = 'home'} />
{/if}

<style>
  main {
    font-family: sans-serif;
  }
</style>
