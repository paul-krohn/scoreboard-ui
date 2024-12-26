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
      /* also send an update; this is a hack to force the scren updater to get the race ID and do a full drow. */
      console.log(`we are mounted ${race.id}, team id: ${race.teams[0].id}, team count:${race.teams[0].count}`);
      updateScore(race.id, race.teams[0].id, race.teams[0].count);
    });

  })

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
    updateScore(race.id, team.id, team.count);
  }

  function fargos(team) {
    var fargo_list = new Array;
    team.players.forEach(p => {
      fargo_list.push(p.fargo);
    })
    console.log(`the fargos are: ${fargo_list}`);
    return fargo_list.join(", ")
  }

</script>

<h1>{race.name}</h1>

  {#each race.teams as team}
  <div class="teamBox">
    <div class="teamLabel">
      {team.name} <small>({fargos(team)})</small>
    </div>
    <div class="scoreWrapper">
      <div class="currentScore">{team.count}</div>
      <div class="buttonWrapper">
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
    flex-grow: 1;
  }
  .scoreWrapper {
    width: 100%;
    padding: 20px;
    background: #F1F1F1;
    display: flex;
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
  .buttonWrapper {
    display: flex;
    align-items: center;
    justify-content: center;
  }

</style>