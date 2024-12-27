<script>
  export let race; // the one passed in from Races
</script>
{#if race}
<a href="/race/{race['id']}" >
{race['name']}
</a>
{:else}
  <p>no race</p>
{/if}
