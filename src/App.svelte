<script lang="ts">
  import { onMount } from "svelte";
  import { Map, Marker } from "mapbox-gl";
  import toGeoJSON from "@mapbox/togeojson";
  import "mapbox-gl/dist/mapbox-gl.css";
  import runGpx from "../public/run.gpx?raw";
  import * as turf from "turf";

  let coordinates: { lng: number; lat: number } = {
    lat: 44.8495616,
    lng: -0.5472256,
  };
  let error: string | null = null;
  let map: Map;

  const options = {
    enableHighAccuracy: true,
    timeout: 10000,
    maximumAge: 0,
  };

  let gpx = new DOMParser().parseFromString(runGpx, "text/xml");

  const route = toGeoJSON.gpx(gpx);
  const routeCoordinates = route.features[0].geometry.coordinates;
  const marker1 = new Marker({ color: "red" }).setLngLat(routeCoordinates[0]); // Point de départ
  const marker2 = new Marker({ color: "blue" }).setLngLat(routeCoordinates[0]); // Point de départ

  // const pinIcon = L.icon({ iconUrl: "vite.svg" });

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
  let counter1 = 0;
  let counter2 = 0;
  let progress1 = 0;
  let progress2 = 0;

  let runners = {
    auxence: 0,
    noemie: 0,
  };

  function getRandomInt(min, max) {
    return Math.random() * (max - min) + min;
  }

  function animateSmoothMarker(
    marker: Marker,
    progress,
    progressFactor,
    counter,
    id
  ) {
    if (counter < routeCoordinates.length - 1) {
      const start = routeCoordinates[counter];
      const end = routeCoordinates[counter + 1];

      const lng = start[0] + (end[0] - start[0]) * progress;
      const lat = start[1] + (end[1] - start[1]) * progress;

      marker.setLngLat([lng, lat]);

      progress += progressFactor; // Ajuster pour une vitesse plus lente ou plus rapide
      let newProgress = progressFactor;

      if (progress >= 1) {
        const coordinates = marker.getLngLat();
        let meter = calculateProgress(routeCoordinates, [
          coordinates.lng,
          coordinates.lat,
        ]);
        runners[id] = Math.round(meter);

        progress = 0;
        newProgress = getRandomInt(0.01, 0.02);
        counter++;
      }

      requestAnimationFrame(() =>
        animateSmoothMarker(marker, progress, newProgress, counter, id)
      );
    }
  }

  $: rank = rankRunners(runners);

  function calculateProgress(course, runnerPosition) {
    let totalDistance = 0; // Distance totale parcourue sur le parcours

    for (let i = 0; i < course.length - 1; i++) {
      const start = turf.point(course[i]);
      const end = turf.point(course[i + 1]);
      const segment = turf.lineString([course[i], course[i + 1]]);

      // Calcul de la distance du segment
      const segmentDistance = turf.lineDistance(segment, "meters");

      // Calcul de la distance du coureur par rapport aux extrémités du segment
      const runner = turf.point(runnerPosition);
      const distanceToStart = turf.distance(runner, start, "meters");
      const distanceToEnd = turf.distance(runner, end, "meters");

      // Si le coureur est sur le segment actuel
      if (distanceToStart + distanceToEnd <= segmentDistance + 1) {
        // Tolérance de 1 mètre
        totalDistance += distanceToStart;
        break;
      } else {
        // Sinon, ajoute la distance complète du segment
        totalDistance += segmentDistance;
      }
    }

    return totalDistance;
  }

  function rankRunners(runners: typeof runners) {
    const distances = Object.keys(runners).map((id) => ({
      runnerId: id,
      distance: runners[id],
    }));

    // Tri des coureurs par distance parcourue (descendant)
    distances.sort((a, b) => b.distance - a.distance);

    return distances;
  }

  onMount(() => {
    map = new Map({
      accessToken: import.meta.env.VITE_MAPBOX_KEY,
      container: "map", // container ID
      style: "mapbox://styles/mapbox/streets-v12", // style URL
      center: coordinates,
      zoom: 14, // starting zoom
    });

    new Marker({}).setLngLat(coordinates).addTo(map);

    map.flyTo({ center: coordinates });

    map.on("style.load", () => {
      map.addSource("gpx", { type: "geojson", data: route });

      map.addLayer({
        id: "gpx",
        type: "line",
        source: "gpx",
        paint: {
          "line-color": "#FF6961",
          "line-width": 5,
        },
      });

      marker1.addTo(map);
      marker2.addTo(map);
    });

    // L.marker(coordinates, { icon: pinIcon }).addTo(Map);

    // Map.flyTo(coordinates, 14, {
    //   animate: true,
    //   duration: 3,
    //   noMoveStart: true,
    // });

    // const gpx = new L.GPX("run.gpx", {
    //   async: true,
    //   polyline_options: { color: "red" },
    // }).addTo(Map);
  });
</script>

<main>
  <button
    on:click={() => {
      animateSmoothMarker(marker1, progress1, 0.02, counter1, "auxence");
      animateSmoothMarker(marker2, progress2, 0.03, counter2, "noemie");
    }}>Start</button
  >
  <section class="rank">
    {#each rank as { runnerId, distance }, index}
      <p>[{index + 1}] Runner : {runnerId} - Distance: {distance / 1000}km</p>
    {/each}
  </section>
  <div id="map"></div>
</main>

<style>
  button {
    position: absolute;
    z-index: 2;
    text-align: center;
    left: 32px;
    top: 12px;
  }

  .rank {
    display: flex;
    position: absolute;
    z-index: 2;
    background: #ffff;
    flex-direction: column;
    padding: 16px;
    gap: 8px;
    border-radius: 8px;
    right: 32px;
    top: 16px;
  }
  #map {
    z-index: 1;
    width: 100%;
    height: 100vh;
  }
</style>
