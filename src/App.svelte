<script lang="ts">
  import svelteLogo from "./assets/svelte.svg";
  import viteLogo from "/vite.svg";
  import Counter from "./lib/Counter.svelte";

  let coordinates = null;
  let error = null;

  const options = {
    enableHighAccuracy: true,
    timeout: 10000,
    maximumAge: 0,
  };

  function sharing() {
    if ("geolocation" in navigator) {
      // Prompt user for permission to access their location
      navigator.geolocation.watchPosition(
        // Success callback function
        function (position) {
          // Get the user's latitude and longitude coordinates
          const lat = position.coords.latitude;
          const lng = position.coords.longitude;
          coordinates = { lat, lng };
          // Update the map with the user's new location
          console.log(`Latitude: ${lat}, longitude: ${lng}`);
        },
        // Error callback function
        function (err) {
          // Handle errors, e.g. user denied location sharing permissions
          switch (err.code) {
            case err.PERMISSION_DENIED:
              error = "L'utilisateur a refusé la demande de géolocalisation.";
              break;
            case err.POSITION_UNAVAILABLE:
              error = "La localisation n'est pas disponible.";
              break;
            case err.TIMEOUT:
              error = "Le délai pour obtenir la localisation a expiré.";
              break;
            default:
              error = "Une erreur inconnue s'est produite.";
          }
        },
        options
      );
    } else {
      error = "Geolocation is not supported by this browser.";
    }
  }
</script>

<main>
  <div>
    <a href="https://vite.dev" target="_blank" rel="noreferrer">
      <img src={viteLogo} class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <button on:click={sharing}>Sharing</button>

  {#if coordinates}
    <p>Longitude: {coordinates.lat}, latitude: {coordinates.lng}</p>
  {/if}

  {#if error}
    <p>Error: {error}</p>
  {/if}
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
