<script>
  import { onMount } from "svelte";
	import RaceSummary from "./RaceSummary.svelte";
  import { PUBLIC_API_URL } from '$env/static/public';
  // define the data holding variable
  let races;
  onMount(async () => {
    await fetch(`${PUBLIC_API_URL}/board/races/`)
      .then(r => r.json())
      .then(data => {
        races = data["races"];
      });
  })

</script>

{#if races}
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

