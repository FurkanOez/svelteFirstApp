<script>
  import debounce from 'lodash-es/debounce';
  import { onMount } from 'svelte';

  let data = [];
  let keyQuery = '';
  let valueQuery = '';
  let searchResults = [];

  const handleSearch = debounce(async () => {
    try {
      const url = new URL('http://localhost:8080/search');
      if (keyQuery.length >= 2) {
        url.searchParams.append('key', keyQuery);
      }
      if (valueQuery.length >= 2) {
        url.searchParams.append('value', valueQuery);
      }

      const response = await fetch(url.toString());
      const jsonData = await response.json();
      data = jsonData.map(item => ({
        key: item.attrName,
        value: item.attrValue
      }));

      searchResults = data.filter(item => {
        const keyMatch = keyQuery === '' || item.key.toLowerCase().includes(keyQuery.toLowerCase());
        const valueMatch = valueQuery === '' || item.value.toLowerCase().includes(valueQuery.toLowerCase());
        return keyMatch && valueMatch;
      });
    } catch (error) {
      console.error('Error fetching data: ', error);
    }
  }, 100); // 300 ms debounce süresi

  $: if (keyQuery.length >= 2 || valueQuery.length >= 2) {
    handleSearch();
  }
</script>

<div>
  <label for="keySearch">Search Key: </label>
  <input id="keySearch" type="text" bind:value={keyQuery}>

  <label for="valueSearch">Search Value: </label>
  <input id="valueSearch" type="text" bind:value={valueQuery}>
</div>

<div>
  <h2>Search Results: </h2>
  {#if searchResults.length > 0}
    <ul>
      {#each searchResults as result}
        <li>{result.key}: {result.value}</li>
      {/each}
    </ul>
  {:else}
    <p>No results found.</p>
  {/if}
</div>
