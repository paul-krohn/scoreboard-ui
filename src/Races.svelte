<script>
  import { onMount } from "svelte";
	import RaceSummary from "./RaceSummary.svelte";
  // define the data holding variable
  let races;
  onMount(async () => {
    await fetch(`http://localhost:8000/board/races/`)
      .then(r => r.json())
      .then(data => {
        races = data;
      });
  })

</script>


<h2>some races</h2>

{#if races}
{races.length}
  <ul>
  {#each races as race }
      <li>    
        <RaceSummary {race} />
      </li>
      {/each}
  </ul>
{:else}
  <p class="loading">loading...</p>
{/if}

