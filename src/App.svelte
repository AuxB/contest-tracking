<script lang="ts">
  import type { Map as LeafletMap } from "leaflet";
  import { map, tileLayer, marker } from "leaflet";
  import "leaflet/dist/leaflet.css";
  import { onMount } from "svelte";

  let coordinates: { lng: number; lat: number } | null = null;
  let error: string | null = null;
  let Map: LeafletMap;

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
          Map.flyTo(coordinates, 14, {
            animate: true,
            duration: 3,
            noMoveStart: true,
          });
          marker(coordinates).addTo(Map);

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

  onMount(() => {
    Map = map("map", {
      center: [46.2276, coordinates?.lat ?? 2.2137],
      zoom: 6.4,
    });

    tileLayer("https://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png", {
      maxZoom: 19,
      attribution:
        '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
    }).addTo(Map);
  });
</script>

<main>
  <button on:click={sharing}>Sharing</button>
  <div id="map"></div>
</main>

<style>
  button {
    position: absolute;
    z-index: 2;
    text-align: center;
    left: 100px;
    top: 12px;
  }
  #map {
    z-index: 1;
    width: 100%;
    height: 100vh;
  }
</style>
