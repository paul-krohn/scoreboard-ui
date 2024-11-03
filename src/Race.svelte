<script>
  import RaceSummary from './RaceSummary.svelte'
  import { onMount } from "svelte";
  let race; // will be populated by onMount
onMount(async () => {
  await fetch(`http://localhost:8000/board/race/${race.id}`)
  .then(r => r.json())
  .then(data => {
    race = data;
  });
})

</script>

<RaceSummary {race}/>