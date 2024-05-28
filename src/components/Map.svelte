<script>
  import mapboxgl from "mapbox-gl";
  import { onMount } from "svelte";
  export let index;
  export let geoJsonToFit;

  mapboxgl.accessToken = "pk.eyJ1IjoibWFya2xlZTQ4MjMiLCJhIjoiY2x2eDkzc3EyMmMzMjJrcnJwNDlqdzAyOSJ9.82DRH8tlG4X8Wx6E2bcbgQ";

  let container;
  let map;
  let restaurants = [];

  // Function to fetch restaurant data
  async function fetchData() {
    const response = await fetch("/santa_barbara_restaurants.csv");
    const data = await response.text();
    const rows = data.split("\n").slice(1); // Skip header row
    // Process each row of the CSV data
    restaurants = rows.map((row) => {
      const [latitude, longitude, name, cuisine, category, stars, review_count] = row.split(",");
      return {
        latitude: parseFloat(latitude),
        longitude: parseFloat(longitude),
        name,
        cuisine,
        category,
        stars: parseFloat(stars),
        review_count: parseInt(review_count)
      };
    });
  }

  function addRestaurantsToMap() {
    const features = restaurants.map((restaurant) => ({
      type: "Feature",
      geometry: {
        type: "Point",
        coordinates: [restaurant.longitude, restaurant.latitude]
      },
      properties: {
        name: restaurant.name,
        cuisine: restaurant.cuisine,
        category: restaurant.category,
        stars: restaurant.stars,
        review_count: restaurant.review_count
      }
    }));

    map.addSource("restaurants", {
      type: "geojson",
      data: {
        type: "FeatureCollection",
        features
      }
    });

    map.addLayer({
      id: "restaurants",
      type: "circle",
      source: "restaurants",
      paint: {
        "circle-radius": 5,
        "circle-color": "#FFCCBC",
        "circle-stroke-color": "#FF5733",
        "circle-stroke-width": 2
      }
    });

    map.on('mouseenter', 'restaurants', (e) => {
      map.getCanvas().style.cursor = 'pointer';
      const coordinates = e.features[0].geometry.coordinates.slice();
      const { name, stars, review_count } = e.features[0].properties;

      new mapboxgl.Popup()
        .setLngLat(coordinates)
        .setHTML(`<strong>${name}</strong><br/>Stars: ${stars}<br/>Reviews: ${review_count}`)
        .addTo(map);
    });

    map.on('mouseleave', 'restaurants', () => {
      map.getCanvas().style.cursor = '';
      map.getPopup().remove();
    });
  }

  onMount(async () => {
    await fetchData();

    const zoomLevel = window.innerWidth <= 600 ? 10 : 11.5;
    map = new mapboxgl.Map({
      container,
      style: "mapbox://styles/mapbox/outdoors-v12",
      center: [-119.705822, 34.426811], // coordinates of Santa Barbara
      zoom: zoomLevel,
      attributionControl: true,
    });

    map.on("load", () => {
      addRestaurantsToMap();
      updateBounds();
      map.on("zoom", updateBounds);
      map.on("drag", updateBounds);
      map.on("move", updateBounds);
    });

    window.addEventListener("resize", () => {
      const screenWidth = window.innerWidth;
      map.setZoom(screenWidth <= 600 ? 10 : 11.5);
    });
  });

  function updateBounds() {
    const bounds = map.getBounds();
    geoJsonToFit.features[0].geometry.coordinates = [
      bounds._ne.lng,
      bounds._ne.lat,
    ];
    geoJsonToFit.features[1].geometry.coordinates = [
      bounds._sw.lng,
      bounds._sw.lat,
    ];
  }

  let isVisible = false;
  $: isVisible = index > 2 || index === 0;
</script>

<svelte:head>
  <link rel="stylesheet" href="https://api.mapbox.com/mapbox-gl-js/v2.14.0/mapbox-gl.css" />
</svelte:head>

<div class="map" class:visible={isVisible} bind:this={container} />

<style>
  .map {
    width: 100%;
    height: 100vh;
    position: absolute;
    opacity: 0;
    visibility: hidden;
    transition: opacity 2s, visibility 2s;
  }

  .map.visible {
    opacity: 1;
    visibility: visible;
  }
</style>
