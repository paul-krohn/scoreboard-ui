<script>
  import RaceSummary from './RaceSummary.svelte'
  import { onMount } from "svelte";
  let props = $props();

  let race = $state(0)

  // let race; // will be populated by onMount
  onMount(async () => {
    // console.log(`going to try to get race ${id}`);
    await fetch(`http://localhost:8000/board/race/${props.id}`)
    .then(r => r.json())
    .then(data => {
      race = data["race"];
    });
  })

</script>
id is :{props.id}:
{race.name}
