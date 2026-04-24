<script>
  import { PUBLIC_API_URL } from '$env/static/public';
  import { onMount } from "svelte";

  let props = $props();

  let race = $state(0);
  let csrf_token = $state(1);

  // let race; // will be populated by onMount
  onMount(async () => {
    // console.log(`going to try to get race ${id}`);
    await fetch(`${PUBLIC_API_URL}/board/race/${props.id}`)
    .then(r => r.json())
    .then(data => {
      race = data["race"];
    });

    await fetch(`${PUBLIC_API_URL}/board/login`, {
      credentials: 'include'
    })
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
        "Content-Type": "application/json",
        "X-CSRFToken": csrf_token,
      },
      credentials: 'include',
      body: JSON.stringify(post_data)
    };

    await fetch(`${PUBLIC_API_URL}/board/race/update_score`, requestOptions)
    .then(r => r.json())
    .then(data => {
      race = data;
    });
  }

  function incrementScore(team, increment) {
    const nextCount = Math.max(0, team.count + increment);
    if (nextCount === team.count) {
      return;
    }

    team.count = nextCount;
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
          <button class="button button-up" type="button" aria-label="increase score for {team.name}" onclick={() => incrementScore(team, 1)}>
            <span aria-hidden="true">➕</span>
          </button>
          <button class="button button-down" type="button" aria-label="decrease score for {team.name}" onclick={() => incrementScore(team, -1)} disabled={team.count === 0}>
            <span aria-hidden="true">➖</span>
          </button>
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
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }
  .button {
    display: flex;
    align-items: center;
    justify-content: center;
    background: transparent;
    border: 1px solid #bfbfbf;
    border-radius: 12px;
    color: #202020;
    cursor: pointer;
    line-height: 1;
  }
  .button-up {
    min-width: 106px;
    min-height: 106px;
    font-size: 3.6rem;
    background: #ffffff;
  }
  .button-down {
    min-width: 64px;
    min-height: 64px;
    font-size: 2.5rem;
    align-self: center;
    background: #f7f7f7;
  }
  .button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }
  .buttonWrapper {
    display: flex;
    align-items: center;
    justify-content: center;
  }

</style>