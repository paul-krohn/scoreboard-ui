<script>
  export let race; // the one passed in from Races
</script>
{#if race}
<a href="/race/{race['id']}" >
{race['name']} {race['teams'][0]['name']} vs {race['teams'][1]['name']}
</a>
{:else}
  <p>no race</p>
{/if}
