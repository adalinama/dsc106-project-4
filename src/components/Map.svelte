<script>
  import mapboxgl from "mapbox-gl";
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";
  export let index;
  export let geoJsonToFit;

  mapboxgl.accessToken = "pk.eyJ1IjoibWFya2xlZTQ4MjMiLCJhIjoiY2x2eDkzc3EyMmMzMjJrcnJwNDlqdzAyOSJ9.82DRH8tlG4X8Wx6E2bcbgQ";

  let container;
  let map;
  let restaurants = [];
  let source;

  let categoryColors = {
      "American": "blue",
      "Mexican": "yellow",
      "Italian": "lightgreen",
      "Chinese": "red",
      "Japanese": "white",
      "Mediterranean": "darkgreen",
      "Thai": "purple",
      "Korean": "lightblue",
      "Indian": "orange",
      "Other": "gray",
    };

  // Define zoom levels for each index
  const zoomLevels = [10, 10, 10, 10.25, 11.5, 12.5, 12];

  // Function to fetch restaurant data
  async function fetchData() {
    try {
      const response = await fetch("/santa_barbara_restaurants.csv");
      const data = await response.text();
      const rows = data.split("\n").slice(1); // Skip header row

      // Process each row of the CSV data
      restaurants = rows.map((row) => {
        // Use regex to split the row by commas, handling quoted strings or lists with commas
        const columns = row.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/);

        // Trim whitespace and remove quotes from values
        const cleanColumns = columns.map((column) => column.trim().replace(/(^")|("$)/g, ''));

        return {
          name: cleanColumns[2],
          address: cleanColumns[3],
          latitude: parseFloat(cleanColumns[7]),
          longitude: parseFloat(cleanColumns[8]),
          stars: parseFloat(cleanColumns[9]),
          review_count: parseInt(cleanColumns[10]),
          category: cleanColumns[14],
          cuisine: cleanColumns[15],
        };
      });

      // Transform restaurant coordinates using the projection
      transformedRestaurants = restaurants.map(restaurant => {
        const [x, y] = projection([restaurant.longitude, restaurant.latitude]);
        return { ...restaurant, x, y };
      });

      // Calculate center coordinates after data loading
      centerCoordinates = projection([-119.75822, 34.426811]);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  }

  function getCategoryColor(cuisine) {
    const colorMap = {
      "American": "blue",
      "Italian": "lightgreen",
      "Mexican": "yellow",
      "Chinese": "red",
      "Japanese": "white",
      "Mediterranean": "olive",
      "Thai": "purple",
      "Korean": "lightblue",
      "Indian": "orange",
    };
    return colorMap[cuisine] || "gray";
  }

  function getCircleRadius(index) {
    // Adjust the coefficient as needed for appropriate scaling
    const coefficient = 0.0025;
    return coefficient * (zoomLevels[index])**3;
  }

  function addRestaurantsToMap() {

    let filteredRestaurants = restaurants;
    if (index < 2) {
      filteredRestaurants = restaurants.slice(0,0);
    }
    if (index === 5) {
      filteredRestaurants = restaurants.filter(restaurant => restaurant.stars === 5);
    }
    if (index === 6) {
      filteredRestaurants = restaurants.filter(restaurant => restaurant.review_count >= 500);
    }

    const features = filteredRestaurants.map((restaurant) => ({
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
        review_count: restaurant.review_count,
        color: index === 4 ? getCategoryColor(restaurant.cuisine) : index === 5 ? "#DEB94F" : index === 6 ? "#E16491" : "#FF5733",
        radius: getCircleRadius(index),
      }
    }));

    source = map.getSource("filteredRestaurants");

    if (!source) {
      source = {
        type: "geojson",
        data: {
          type: "FeatureCollection",
          features
        }
      };

      map.addSource("filteredRestaurants", source)

      map.addLayer(
        {
        id: "filteredRestaurants",
        type: "circle",
        source: "filteredRestaurants",
        paint: {
          "circle-radius": ["get", "radius"],
          "circle-color": ["get", "color"],
          "circle-stroke-color": "#000000",
          "circle-stroke-width": 0.5,
          "circle-opacity": 1
        }
      });
    } else {
      source.setData({
        type: "FeatureCollection",
        features
      });
    }
  }

  

  onMount(async () => {
    await fetchData();

    const zoomLevel = window.innerWidth <= 600 ? 10 : zoomLevels[0];
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

    map.on("moveend", () => {
      addRestaurantsToMap();
    });

    window.addEventListener("resize", () => {
      const screenWidth = window.innerWidth;
      const newZoomLevel = screenWidth <= 600 ? 10 : zoomLevels[index] || zoomLevels[0];
      map.easeTo({ zoom: newZoomLevel, duration: 1000 });
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
  $: isVisible = index === 0 || index > 2;

  let showLegend = false;
  $: showLegend = index === 4;

  // Update zoom level when index changes with smooth transition
  $: if (map && zoomLevels[index] !== undefined) {
    map.easeTo({ zoom: zoomLevels[index], duration: 1000 });
  }

</script>

<svelte:head>
  <link rel="stylesheet" href="https://api.mapbox.com/mapbox-gl-js/v2.14.0/mapbox-gl.css" />
</svelte:head>

<div class="map" class:visible={isVisible} bind:this={container} />

{#if showLegend}
  <div class="map-legend" transition:fade={{ delay: 800, duration: 200 }}>
    <h3>Cuisine Legend</h3>
    <ul>
      {#each Object.entries(categoryColors) as [cuisine, color]}
        <li>
          <span class="legend-circle" style="background-color: {color};"></span>
          {cuisine}
        </li>
      {/each}
    </ul>
  </div>
{/if}

<style>
  .map {
    width: 100%;
    height: 100vh;
    position: absolute;
    opacity: 0;
    visibility: hidden;
    transition: opacity 1s, visibility 1s;
  }

  .map.visible {
    opacity: 1;
    visibility: visible;
  }

  .map-legend {
    position: absolute;
    top: 20px;
    right: 20px;
    background-color: rgba(255, 255, 255, 0.5);
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    z-index: 1000;
    transition: opacity 1s;
  }

  .map-legend h3 {
    margin-top: 0;
  }

  .map-legend ul {
    padding: 0;
    list-style: none;
  }

  .map-legend li {
    margin-bottom: 5px;
  }

  .legend-circle {
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    margin-right: 5px;
  }
</style>
