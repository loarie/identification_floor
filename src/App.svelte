<script lang="ts">
  let observationId = '';
  let observedOn = '';
  let taxonName = '';
  let loading = false;
  let error = '';

  async function fetchObservation() {
    if (!observationId.trim()) {
      error = 'Please enter an observation ID';
      return;
    }

    loading = true;
    error = '';
    observedOn = '';
    taxonName = '';

    try {
      const response = await fetch(`https://api.inaturalist.org/v1/observations/${observationId}`);

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();

      if (data.results && data.results.length > 0) {
        const observation = data.results[0];
        observedOn = observation.observed_on || 'No date recorded';
        taxonName = observation.taxon?.name || 'No taxon identified';
      } else {
        error = 'Observation not found';
      }
    } catch (e) {
      error = `Error fetching observation: ${e.message}`;
    } finally {
      loading = false;
    }
  }

  function handleSubmit(event: Event) {
    event.preventDefault();
    fetchObservation();
  }
</script>

<main>
  <h1>iNaturalist Observation Viewer</h1>

  <form on:submit={handleSubmit}>
    <div class="input-group">
      <label for="obs-id">Observation ID:</label>
      <input
        id="obs-id"
        type="text"
        bind:value={observationId}
        placeholder="e.g., 43048348"
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Loading...' : 'Fetch Observation'}
      </button>
    </div>
  </form>

  {#if error}
    <div class="error">
      {error}
    </div>
  {/if}

  {#if observedOn && taxonName}
    <div class="results">
      <h2>Observation Details</h2>
      <div class="detail">
        <strong>Observed On:</strong> {observedOn}
      </div>
      <div class="detail">
        <strong>Taxon Name:</strong> {taxonName}
      </div>
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 600px;
    margin: 2rem auto;
    padding: 2rem;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  }

  h1 {
    color: #333;
    margin-bottom: 2rem;
  }

  h2 {
    color: #555;
    font-size: 1.5rem;
    margin-bottom: 1rem;
  }

  .input-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  label {
    font-weight: 600;
    color: #555;
  }

  input {
    padding: 0.75rem;
    font-size: 1rem;
    border: 2px solid #ddd;
    border-radius: 4px;
    transition: border-color 0.3s;
  }

  input:focus {
    outline: none;
    border-color: #74ac00;
  }

  button {
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
    font-weight: 600;
    color: white;
    background-color: #74ac00;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  button:hover:not(:disabled) {
    background-color: #5d8800;
  }

  button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }

  .error {
    padding: 1rem;
    background-color: #fee;
    border: 1px solid #fcc;
    border-radius: 4px;
    color: #c00;
    margin-top: 1rem;
  }

  .results {
    margin-top: 2rem;
    padding: 1.5rem;
    background-color: #f8f8f8;
    border-radius: 4px;
    border: 1px solid #ddd;
  }

  .detail {
    padding: 0.5rem 0;
    font-size: 1.1rem;
  }

  .detail strong {
    color: #555;
  }
</style>
