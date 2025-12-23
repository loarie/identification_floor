<script lang="ts">
  let observationId = '';
  let observation: any = null;
  let loading = false;
  let error = '';
  let taxonSearchQuery = '';
  let taxonSearchResults: any[] = [];
  let selectedTaxon: any = null;
  let showTaxonDropdown = false;
  let demoIdentifications: any[] = [];

  async function fetchObservation() {
    if (!observationId.trim()) {
      error = 'Please enter an observation ID';
      return;
    }

    loading = true;
    error = '';
    observation = null;

    try {
      const response = await fetch(`https://api.inaturalist.org/v1/observations/${observationId}`);

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();

      if (data.results && data.results.length > 0) {
        observation = data.results[0];
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

  function getTaxonDisplay(obs: any): string {
    if (!obs.taxon) return 'No taxon identified';
    const commonName = obs.taxon.preferred_common_name;
    const scientificName = obs.taxon.name;

    if (commonName && scientificName) {
      return `${commonName} (${scientificName})`;
    } else if (scientificName) {
      return scientificName;
    } else if (commonName) {
      return commonName;
    }
    return 'No taxon identified';
  }

  function getPhotoUrl(obs: any): string | null {
    if (obs.observation_photos && obs.observation_photos.length > 0) {
      return obs.observation_photos[0].photo.url;
    }
    return null;
  }

  function getQualityGradeDisplay(grade: string): string {
    if (grade === 'research') return 'Research Grade';
    if (grade === 'needs_id') return 'Needs ID';
    if (grade === 'casual') return 'Casual';
    return grade;
  }

  function getTaxonDisplayForId(taxon: any): string {
    if (!taxon) return 'Unknown';
    const commonName = taxon.preferred_common_name;
    const scientificName = taxon.name;

    if (commonName && scientificName) {
      return `${commonName} (${scientificName})`;
    } else if (scientificName) {
      return scientificName;
    } else if (commonName) {
      return commonName;
    }
    return 'Unknown';
  }

  function getSortedIdentifications(obs: any): any[] {
    if (!obs.identifications || obs.identifications.length === 0) {
      return [];
    }
    return [...obs.identifications].sort((a, b) => {
      return new Date(a.created_at).getTime() - new Date(b.created_at).getTime();
    });
  }

  function getIdPhotoUrl(identification: any): string | null {
    if (identification.taxon && identification.taxon.default_photo && identification.taxon.default_photo.url) {
      return identification.taxon.default_photo.url;
    }
    return null;
  }

  async function searchTaxa() {
    if (taxonSearchQuery.length < 2) {
      taxonSearchResults = [];
      showTaxonDropdown = false;
      return;
    }

    try {
      const response = await fetch(`https://api.inaturalist.org/v1/taxa/autocomplete?q=${encodeURIComponent(taxonSearchQuery)}`);
      const data = await response.json();
      taxonSearchResults = data.results || [];
      showTaxonDropdown = true;
    } catch (e) {
      console.error('Error searching taxa:', e);
      taxonSearchResults = [];
    }
  }

  function selectTaxon(taxon: any) {
    selectedTaxon = taxon;
    taxonSearchQuery = taxon.name;
    showTaxonDropdown = false;
  }

  function submitIdentification() {
    if (!selectedTaxon) return;

    const newId = {
      id: `demo-${Date.now()}`,
      user: {
        login: 'demo_user',
        icon: null
      },
      taxon: selectedTaxon,
      current: true,
      created_at: new Date().toISOString()
    };

    demoIdentifications = [...demoIdentifications, newId];
    taxonSearchQuery = '';
    selectedTaxon = null;
    taxonSearchResults = [];
  }

  function removeDemoIdentification(idToRemove: string) {
    demoIdentifications = demoIdentifications.filter(id => id.id !== idToRemove);
  }

  function getAllIdentifications(obs: any): any[] {
    return [...getSortedIdentifications(obs), ...demoIdentifications];
  }
</script>

<main>
  <div class="header-container">
    <h1>iNaturalist identification floor demo</h1>
  </div>

  <form on:submit={handleSubmit}>
    <div class="input-group">
      <input
        id="obs-id"
        type="text"
        bind:value={observationId}
        placeholder="Enter an iNaturalist Observation ID, e.g., 331799949"
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

  {#if observation}
    <div class="results">
      <div class="taxon-quality-row">
        <span class="taxon-name-display">{getTaxonDisplay(observation)}</span>
        <span class="quality-grade {observation.quality_grade}">
          {getQualityGradeDisplay(observation.quality_grade)}
        </span>
      </div>

      <div class="obs-content">
        <div class="photo-column">
          {#if getPhotoUrl(observation)}
            <img src={getPhotoUrl(observation).replace('square', 'medium')} alt="Observation" class="observation-photo" />
          {/if}
        </div>
        <div class="info-column">
          <div class="detail">
            <strong>Observed:</strong> {observation.observed_on || 'No date recorded'}
          </div>
          <div class="detail user-detail">
            <strong>Observer:</strong>
            <div class="user-info">
              {#if observation.user.icon}
                <img src={observation.user.icon} alt={observation.user.login} class="user-icon" />
              {:else}
                <div class="user-icon-placeholder">
                  {observation.user.login.charAt(0).toUpperCase()}
                </div>
              {/if}
              <span class="user-login">{observation.user.login}</span>
            </div>
          </div>
        </div>
      </div>

      {#if getSortedIdentifications(observation).length > 0 || demoIdentifications.length > 0}
        <div class="identifications-section">
          <h3>Identification activity</h3>
          {#each getAllIdentifications(observation) as identification}
            <div class="identification-item {identification.current === false ? 'withdrawn' : ''}">
              <div class="id-user">
                {#if identification.user.icon}
                  <img src={identification.user.icon} alt={identification.user.login} class="user-icon" />
                {:else}
                  {#if identification.user.login === 'demo_user'}
                    <div class="user-icon-placeholder smiley">
                      😊
                    </div>
                  {:else}
                    <div class="user-icon-placeholder">
                      {identification.user.login.charAt(0).toUpperCase()}
                    </div>
                  {/if}
                {/if}
                <span class="user-login">{identification.user.login}</span>
              </div>
              <div class="id-taxon">
                {#if getIdPhotoUrl(identification)}
                  <img src={getIdPhotoUrl(identification)} alt={identification.taxon?.name || 'Taxon'} class="taxon-photo" />
                {/if}
                <div class="taxon-info">
                  <span class="taxon-name">{getTaxonDisplayForId(identification.taxon)}</span>
                  {#if identification.disagreement && identification.previous_observation_taxon}
                    <div class="disagreement">
                      Disagrees with {identification.previous_observation_taxon.name}
                    </div>
                  {/if}
                </div>
              </div>
              {#if identification.user.login === 'demo_user' && identification.id}
                <button class="remove-id-btn" on:click={() => removeDemoIdentification(identification.id)}>
                  ×
                </button>
              {/if}
            </div>
          {/each}

          {#if demoIdentifications.length === 0}
          <div class="add-id-form">
            <h4>Add your identification</h4>
            <div class="taxon-search">
              <input
                type="text"
                bind:value={taxonSearchQuery}
                on:input={searchTaxa}
                placeholder="Search for a taxon..."
                class="taxon-input"
              />
              {#if showTaxonDropdown && taxonSearchResults.length > 0}
                <div class="taxon-dropdown">
                  {#each taxonSearchResults as taxon}
                    <div class="taxon-result" on:click={() => selectTaxon(taxon)}>
                      {#if taxon.default_photo?.square_url}
                        <img src={taxon.default_photo.square_url} alt={taxon.name} class="taxon-result-photo" />
                      {/if}
                      <div class="taxon-result-info">
                        <div class="taxon-result-name">
                          {taxon.preferred_common_name ? `${taxon.preferred_common_name} (${taxon.name})` : taxon.name}
                        </div>
                      </div>
                    </div>
                  {/each}
                </div>
              {/if}
            </div>
            <button
              type="button"
              class="submit-id-btn"
              on:click={submitIdentification}
              disabled={!selectedTaxon}
            >
              Add Identification
            </button>
          </div>
          {/if}
        </div>
      {/if}
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 800px;
    min-width: 688px;
    margin: 2rem auto;
    padding: 2rem;
    padding-top: 5rem;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  }

  .header-container {
    position: fixed;
    top: 0;
    left: 0;
    padding: 1rem 1.5rem;
    background-color: white;
    z-index: 1000;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    width: 100%;
  }

  h1 {
    color: #333;
    font-size: 1.2rem;
    margin: 0;
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
    width: 100%;
  }

  input {
    padding: 0.75rem;
    font-size: 1rem;
    border: 2px solid #ddd;
    border-radius: 4px;
    transition: border-color 0.3s;
    width: 100%;
    box-sizing: border-box;
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
    width: 100%;
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

  .taxon-quality-row {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1.5rem;
    flex-wrap: wrap;
  }

  .taxon-name-display {
    font-size: 1.3rem;
    font-weight: 600;
    color: #333;
  }

  .obs-content {
    display: flex;
    gap: 1.5rem;
    margin-bottom: 1.5rem;
  }

  .photo-column {
    flex-shrink: 0;
    width: 200px;
  }

  .observation-photo {
    width: 100%;
    height: auto;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }

  .info-column {
    flex: 1;
    min-width: 350px;
    display: flex;
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }

  .detail {
    font-size: 1rem;
    white-space: nowrap;
  }

  .detail strong {
    color: #555;
    display: inline;
    margin-right: 0.5rem;
  }

  .quality-grade {
    display: inline-block;
    padding: 4px 12px;
    border-radius: 3px;
    font-size: 0.95rem;
    font-weight: 600;
  }

  .quality-grade.research {
    background-color: #74ac00;
    color: white;
  }

  .quality-grade.needs_id {
    background-color: #ffc107;
    color: #333;
  }

  .quality-grade.casual {
    background-color: #999;
    color: white;
  }

  .user-detail {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    white-space: nowrap;
  }

  .user-info {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
  }

  .user-icon {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    object-fit: cover;
  }

  .user-icon-placeholder {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    background-color: #74ac00;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 600;
  }

  .user-login {
    font-weight: 500;
  }

  .identifications-section {
    margin-top: 1.5rem;
    padding-top: 1rem;
    border-top: 2px solid #ddd;
  }

  .identifications-section h3 {
    font-size: 1.1rem;
    color: #555;
    margin-bottom: 0.75rem;
    text-align: left;
  }

  .identification-item {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 0.75rem;
    margin-bottom: 0.5rem;
    background-color: white;
    border-radius: 4px;
    border: 1px solid #e0e0e0;
    position: relative;
  }

  .remove-id-btn {
    position: absolute;
    top: 0.25rem;
    right: 0.25rem;
    background: none;
    border: none;
    font-size: 1.5rem;
    line-height: 1;
    color: #999;
    cursor: pointer;
    padding: 0.25rem 0.5rem;
    width: auto;
  }

  .remove-id-btn:hover {
    color: #c00;
    background-color: transparent;
  }

  .identification-item.withdrawn {
    background-color: #f5f5f5;
    opacity: 0.6;
  }

  .identification-item.withdrawn .taxon-name {
    text-decoration: line-through;
    color: #999;
  }

  .identification-item.withdrawn .user-login {
    color: #999;
  }

  .id-user {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    min-width: 150px;
  }

  .id-taxon {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    flex: 1;
  }

  .taxon-photo {
    width: 48px;
    height: 48px;
    border-radius: 4px;
    object-fit: cover;
    flex-shrink: 0;
  }

  .taxon-info {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .taxon-name {
    font-weight: 500;
  }

  .disagreement {
    font-size: 0.85rem;
    color: #c00;
    font-style: italic;
  }

  .user-icon-placeholder.smiley {
    font-size: 20px;
    line-height: 32px;
  }

  .add-id-form {
    margin-top: 1rem;
    padding: 1rem;
    background-color: #f9f9f9;
    border-radius: 4px;
    border: 1px solid #ddd;
  }

  .add-id-form h4 {
    font-size: 1rem;
    color: #555;
    margin: 0 0 0.75rem 0;
  }

  .taxon-search {
    position: relative;
    margin-bottom: 0.75rem;
  }

  .taxon-input {
    width: 100%;
    padding: 0.5rem;
    font-size: 1rem;
    border: 2px solid #ddd;
    border-radius: 4px;
    box-sizing: border-box;
  }

  .taxon-input:focus {
    outline: none;
    border-color: #74ac00;
  }

  .taxon-dropdown {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    max-height: 300px;
    overflow-y: auto;
    background: white;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    z-index: 100;
  }

  .taxon-result {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem;
    cursor: pointer;
    border-bottom: 1px solid #f0f0f0;
  }

  .taxon-result:hover {
    background-color: #f5f5f5;
  }

  .taxon-result-photo {
    width: 40px;
    height: 40px;
    border-radius: 4px;
    object-fit: cover;
  }

  .taxon-result-info {
    flex: 1;
  }

  .taxon-result-name {
    font-weight: 500;
  }

  .submit-id-btn {
    padding: 0.5rem 1rem;
    font-size: 1rem;
    font-weight: 600;
    color: white;
    background-color: #74ac00;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    width: 100%;
  }

  .submit-id-btn:hover:not(:disabled) {
    background-color: #5d8800;
  }

  .submit-id-btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
</style>
