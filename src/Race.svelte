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

  function incrementScore(team, increment) {
    team.count += increment;
    updateScores()
  }

</script>

<h1>{race.name}</h1>

  {#each race.teams as team}
  <div class="teamBox">
    <div class="teamLabel">
      {team.name}
    </div>
    <div>
      <div class="scoreWrapper" style="">
        <div class="currentScore">{team.count}</div>
        <div class="buttons">
          <div class="button">
            <img class="buttonImage" src="/icons8-up-button-80.png" alt="increase score for {team.name}" onclick={() => incrementScore(team, 1)}/>
          </div>
          <div class="button">
            <img class="buttonImage" src="/icons8-down-button-40.png" alt="decrease score for {team.name}" onclick={() => incrementScore(team, -1)}/>
          </div>
        </div>
      </div>
    </div>
  </div>
  {/each}

  <!-- <div style="display:flex;justify-content:center;align-items:center;">
    <div style="width:5em;"></div>
  </div> -->
<style>

  .teamBox {
    padding: 1em;
  }

  .teamLabel {
    font-size: 2em;
    padding: 20px;
    background: #383838;
    color: #FFFFFF;
  }
  .currentScore {
    font-size: 8em;
    display: table-cell;
  }
  .scoreWrapper {
    padding: 20px;
    background: #F1F1F1;
  }
  .buttons {
    padding: 10px;
    display: table-cell;
    vertical-align: middle;
  }
  .button {
    display: flex;
    align-items: center;
    justify-content: center;
    vertical-align: middle;
  }

</style>