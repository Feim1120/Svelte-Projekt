<script>
  import { onMount, createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let amount = 1;
  let from = 'EUR';
  let to = 'USD';
  let result = null;
  let currencies = [];
  let rates = {};

  onMount(async () => {
    const res = await fetch('https://api.frankfurter.app/currencies');
    const data = await res.json();
    currencies = Object.keys(data);
    await loadRates();
  });

  async function loadRates() {
    const res = await fetch(`https://api.frankfurter.app/latest?from=${from}`);
    const data = await res.json();
    rates = data.rates;
    convert();
  }

  function convert() {
    if (rates[to]) {
      result = (amount * rates[to]).toFixed(2);
    } else {
      result = null;
    }
  }

  function switchCurrencies() {
    [from, to] = [to, from];
    loadRates();
  }

  function goBack() {
    dispatch('back');
  }
</script>

<button on:click={goBack} class="mb-4 bg-gray-300 px-3 py-1 rounded hover:bg-gray-400">
  ⬅ zurück
</button>

<main class="max-w-md mx-auto p-4 bg-white rounded shadow text-center">
  <h2 class="text-xl font-bold mb-4">💱 Währungsrechner</h2>

  <div class="flex flex-col gap-4">
    <input
      type="number"
      bind:value={amount}
      min="0"
      class="border p-2 rounded"
      on:input={convert}
    />

    <div class="flex gap-2 items-center justify-center">
      <select bind:value={from} on:change={loadRates} class="border p-2 rounded">
        {#each currencies as cur}
          <option value={cur}>{cur}</option>
        {/each}
      </select>

      <button on:click={switchCurrencies} class="text-2xl">⇆</button>

      <select bind:value={to} on:change={convert} class="border p-2 rounded">
        {#each currencies as cur}
          <option value={cur}>{cur}</option>
        {/each}
      </select>
    </div>

    {#if result}
      <div class="text-lg mt-4">
        {amount} {from} = <strong>{result} {to}</strong>
      </div>
    {/if}
  </div>
</main>
