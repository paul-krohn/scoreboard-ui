<script>
  import { PUBLIC_API_URL } from '$env/static/public';
  import { onMount } from "svelte";
  let props = $props();

  // let race = $state(0);
  let csrf_token = $state(1);

  onMount(async () => {
    await fetch(`${PUBLIC_API_URL}/board/login`)
    .then(r => r.json())
    .then(data => {
      csrf_token = data["token"];
      /* also send an update; this is a hack to force the scren updater to get the race ID and do a full drow. */
      // console.log(`we are mounted ${race.id}, team id: ${race.teams[0].id}, team count:${race.teams[0].count}`);
      // updateScore(race.id, race.teams[0].id, race.teams[0].count);
    });
  })

  const clearScreen = async() => {
    // console.log(`updateScore called with: ${race_id}, ${team_id}, ${count}`);
    var post_data = {};
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

    await fetch(`${PUBLIC_API_URL}/board/controls/clear_screen`, requestOptions)
      .then(r => r.json())
      .then(data => {
        race = data;
    });
  }
</script>
<h1>Controls</h1>

<div class="button">
  <input type="button" alt="clear the screen" onclick={() => clearScreen()} value="Clear Display"/>
</div>

