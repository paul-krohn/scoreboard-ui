<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';
  import Nav from '../../Nav.svelte';
  import TeamPlayerSelect from '../../TeamPlayerSelect.svelte';

  let players = [];
  let csrfToken = '';
  let loading = true;
  let submitting = false;
  let errorMessage = '';

  let raceName = '';
  let raceTo = 5;
  let playersPerTeam = 1;

  let favoriteExistingIds = [];
  let underdogExistingIds = [];
  let favoriteNewPlayers = [{ name: '', fargo: '' }];
  let underdogNewPlayers = [{ name: '', fargo: '' }];

  onMount(async () => {
    await loadFormData();
  });

  async function loadFormData() {
    loading = true;
    errorMessage = '';

    try {
      const loginResponse = await fetch(`${PUBLIC_API_URL}/board/login`, {
        credentials: 'include'
      });
      const playersResponse = await fetch(`${PUBLIC_API_URL}/board/players/`);

      if (!playersResponse.ok || !loginResponse.ok) {
        throw new Error('Failed to load player list');
      }

      const playersData = await playersResponse.json();
      const loginData = await loginResponse.json();
      players = playersData.players || [];
      csrfToken = loginData.token;
    } catch (error) {
      errorMessage = error.message || 'Failed to load form data';
    } finally {
      loading = false;
    }
  }

  function cleanNewPlayers(list) {
    return list
      .map((player) => ({
        name: (player.name || '').trim(),
        fargo: player.fargo
      }))
      .filter((player) => player.name)
      .map((player) => ({
        ...player,
        fargo: player.fargo === '' ? null : player.fargo
      }));
  }

  function totalSelected(existingIds, newList) {
    return existingIds.length + cleanNewPlayers(newList).length;
  }

  function getMatchType() {
    return playersPerTeam === 1 ? 'singles' : 'team';
  }

  async function createRace() {
    errorMessage = '';

    if (totalSelected(favoriteExistingIds, favoriteNewPlayers) !== playersPerTeam) {
      errorMessage = `Favorite side must include exactly ${playersPerTeam} player(s).`;
      return;
    }

    if (totalSelected(underdogExistingIds, underdogNewPlayers) !== playersPerTeam) {
      errorMessage = `Underdog side must include exactly ${playersPerTeam} player(s).`;
      return;
    }

    submitting = true;
    try {
      const matchType = getMatchType();
      const payload = {
        race_name: raceName,
        to: Number(raceTo),
        teams: [
          {
            player_ids: favoriteExistingIds,
            new_players: cleanNewPlayers(favoriteNewPlayers)
          },
          {
            player_ids: underdogExistingIds,
            new_players: cleanNewPlayers(underdogNewPlayers)
          }
        ]
      };

      const endpoint = matchType === 'singles' ? 'create_singles' : 'create_team';
      const response = await fetch(`${PUBLIC_API_URL}/board/races/${endpoint}`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-CSRFToken': csrfToken
        },
        credentials: 'include',
        body: JSON.stringify(payload)
      });

      const data = await response.json();
      if (!response.ok) {
        throw new Error(data.error || 'Could not create match');
      }

      await goto(`/race/${data.race.id}`);
    } catch (error) {
      errorMessage = error.message || 'Could not create match';
    } finally {
      submitting = false;
    }
  }
</script>

<Nav />

<h1>Create Match</h1>

{#if loading}
  <p>Loading players...</p>
{:else}
  <div class="form-section">
    <div class="controls">
      <label>
        Match Type
        <select bind:value={playersPerTeam}>
          <option value={1}>Singles (1v1)</option>
          <option value={2}>Scotch Doubles (2v2)</option>
        </select>
      </label>

      <label>
        Race name (optional)
        <input type="text" bind:value={raceName} placeholder="e.g. Friday Night Match" />
      </label>

      <label>
        Race to
        <input type="number" min="1" bind:value={raceTo} />
      </label>
    </div>

    <TeamPlayerSelect
      heading="Favorite"
      {players}
      bind:selectedExistingIds={favoriteExistingIds}
      bind:newPlayers={favoriteNewPlayers}
      requiredCount={playersPerTeam}
    />

    <TeamPlayerSelect
      heading="Underdog"
      {players}
      bind:selectedExistingIds={underdogExistingIds}
      bind:newPlayers={underdogNewPlayers}
      requiredCount={playersPerTeam}
    />

    {#if errorMessage}
      <p class="error">{errorMessage}</p>
    {/if}

    <button type="button" on:click={createRace} disabled={submitting}>
      {submitting ? 'Creating...' : 'Create Match'}
    </button>
  </div>
{/if}

<style>
  .form-section {
    max-width: 800px;
    margin: 0 auto;
  }

  .controls {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 0.75rem;
    margin-bottom: 1rem;
  }

  label {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
  }

  input,
  select {
    border: 1px solid #d1d1d1;
    border-radius: 4px;
    padding: 0.45rem;
  }

  .error {
    color: #a11717;
  }

  button {
    margin-top: 1rem;
    padding: 0.6rem 1.2rem;
    background: #0066cc;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  button:disabled {
    background: #ccc;
    cursor: not-allowed;
  }

  button:not(:disabled):hover {
    background: #0052a3;
  }
</style>
