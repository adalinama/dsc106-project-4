<script>
  import { onMount } from "svelte";
  import { geoMercator } from "d3-geo";

  export let index, width, height;

  let restaurants = [];
  let projection;

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

  onMount(() => {
    fetchData();

    // Initialize projection
    projection = geoMercator()
      .center([-119.705822, 34.426811]) // Center on Santa Barbara
      .translate([width / 2, height / 2]) // Translate to center of the SVG
      .scale(10000); // Adjust scale as needed
  });

  // Transform restaurant coordinates
  $: transformedRestaurants = restaurants.map(restaurant => {
    const [x, y] = projection([restaurant.longitude, restaurant.latitude]);
    return { ...restaurant, x, y };
  });
</script>

<svg class="graph" width={width} height={height}>
  {#if index === 4}
    {#each transformedRestaurants as restaurant}
      <circle
        cx={restaurant.x}
        cy={restaurant.y}
        r="5"
        fill="#FFCCBC"
        stroke="#FF5733"
        stroke-width="2"
        title={restaurant.name}
      >
        <title>{restaurant.name}</title>
      </circle>
    {/each}
  {/if}
</svg>

<style>
  /* Add your custom styles here */
  svg.graph {
    width: 100%;
    height: 100%;
  }
  circle {
    transition: transform 0.5s ease;
  }
  circle:hover {
    transform: scale(1.5);
  }
</style>