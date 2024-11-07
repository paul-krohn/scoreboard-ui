<script>
  import RaceSummary from './RaceSummary.svelte'
  import { onMount } from "svelte";
  import { Input } from '@sveltestrap/sveltestrap';

  let props = $props();

  let race = $state(0);
  let csrf_token = $state(1);

  // let race; // will be populated by onMount
  onMount(async () => {
    // console.log(`going to try to get race ${id}`);
    await fetch(`http://localhost:8080/board/race/${props.id}`)
    .then(r => r.json())
    .then(data => {
      race = data["race"];
    });

    await fetch(`http://localhost:8080/board/login`)
    .then(r => r.json())
    .then(data => {
      csrf_token = data["token"];
    });

  })


  const updateScores = () => {
    console.log("update scores called");
    race.teams.forEach(team => {
      console.log(`in ${race.name} team ${team.name} count is ${team.count}`);
      updateScore(race.id, team.id, team.count);
    })

  }

  const updateScore = async(race_id, team_id, count) => {
    console.log(`updateScore called with: ${race_id}, ${team_id}, ${count}`);
    var post_data = {
        "id": race_id,
        "team_id": team_id,
        "count": count,
      };
    const requestOptions = {
      method: "POST",
      headers: {
        "Access-Control-Allow-Crendentials": "true",
        "Content-Type": "application/json",
        "X-CSRFToken": csrf_token,
        "X-Method": "POST",
      },
      credentials: 'include',
      body: JSON.stringify(post_data)
    };

    await fetch(`http://localhost:8080/board/race/update_score`, requestOptions)
    .then(r => r.json())
    .then(data => {
      race = data;
    });
  }

</script>

<h1>{race.name}</h1>

  <form method="POST">
    {#each race.teams as team}
		<label>
			{team.name}
      <Input type="number" bind:value={team.count} min="0" max={team.to} onchange={updateScores}/>
		</label>
    {/each}
	</form>
