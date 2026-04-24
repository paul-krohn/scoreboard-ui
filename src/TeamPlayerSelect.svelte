<script>
  export let heading;
  export let players = [];
  export let selectedExistingIds = [];
  export let newPlayers = [{ name: '', fargo: '' }];
  export let requiredCount = 1;

  let filterText = '';

  $: normalizedFilter = filterText.trim().toLowerCase();
  $: filteredPlayers = players.filter((player) => {
    if (!normalizedFilter) {
      return true;
    }
    return player.name.toLowerCase().includes(normalizedFilter);
  });

  $: selectedCount = selectedExistingIds.length + newPlayers.filter((player) => (player.name || '').trim()).length;

  function toggleExistingPlayer(playerId, checked) {
    if (checked) {
      if (!selectedExistingIds.includes(playerId)) {
        selectedExistingIds = [...selectedExistingIds, playerId];
      }
      return;
    }

    selectedExistingIds = selectedExistingIds.filter((id) => id !== playerId);
  }

  function addNewPlayerRow() {
    newPlayers = [...newPlayers, { name: '', fargo: '' }];
  }

  function removeNewPlayerRow(index) {
    if (newPlayers.length === 1) {
      newPlayers = [{ name: '', fargo: '' }];
      return;
    }
    newPlayers = newPlayers.filter((_, i) => i !== index);
  }

  function updateNewPlayer(index, field, value) {
    newPlayers = newPlayers.map((player, i) => {
      if (i !== index) {
        return player;
      }
      return {
        ...player,
        [field]: value,
      };
    });
  }
</script>

<section class="team-select">
  <h3>{heading}</h3>
  <p class="target-count">Selected: {selectedCount} / {requiredCount}</p>

  <div class="card">
    <h4>Existing Players</h4>
    <input
      type="text"
      placeholder="Filter players by name"
      bind:value={filterText}
      class="text-input"
    />

    <div class="player-list">
      {#if filteredPlayers.length > 0}
        {#each filteredPlayers as player}
          <label class="player-item">
            <input
              type="checkbox"
              checked={selectedExistingIds.includes(player.id)}
              on:change={(event) => toggleExistingPlayer(player.id, event.currentTarget.checked)}
            />
            <span>{player.name} {#if player.fargo != null}<small>({player.fargo})</small>{/if}</span>
          </label>
        {/each}
      {:else}
        <p>No players match this filter.</p>
      {/if}
    </div>
  </div>

  <div class="card">
    <h4>Add New Players</h4>
    {#each newPlayers as player, index}
      <div class="new-player-row">
        <input
          type="text"
          placeholder="Player name"
          value={player.name}
          on:input={(event) => updateNewPlayer(index, 'name', event.currentTarget.value)}
          class="text-input"
        />
        <input
          type="number"
          placeholder="Fargo"
          value={player.fargo}
          on:input={(event) => updateNewPlayer(index, 'fargo', event.currentTarget.value)}
          class="number-input"
        />
        <button type="button" on:click={() => removeNewPlayerRow(index)}>Remove</button>
      </div>
    {/each}

    <button type="button" on:click={addNewPlayerRow}>Add Another Player</button>
  </div>
</section>

<style>
  .team-select {
    border: 1px solid #d9d9d9;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1rem;
  }

  .target-count {
    margin-top: -0.4rem;
    color: #4c4c4c;
  }

  .card {
    background: #f8f8f8;
    border-radius: 6px;
    padding: 0.75rem;
    margin-top: 0.75rem;
  }

  .player-list {
    max-height: 220px;
    overflow-y: auto;
    border: 1px solid #e4e4e4;
    border-radius: 4px;
    background: #fff;
    padding: 0.5rem;
  }

  .player-item {
    display: block;
    margin-bottom: 0.35rem;
  }

  .new-player-row {
    display: grid;
    grid-template-columns: 1fr 120px 90px;
    gap: 0.5rem;
    margin-bottom: 0.5rem;
    align-items: center;
  }

  .text-input,
  .number-input {
    width: 100%;
    border: 1px solid #d1d1d1;
    border-radius: 4px;
    padding: 0.4rem;
  }

  @media (max-width: 700px) {
    .new-player-row {
      grid-template-columns: 1fr;
    }
  }
</style>