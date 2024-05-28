<script>
  import { onMount, afterUpdate } from "svelte";

  export let index;
  export let width;
  export let height;
  export let projection;

  let restaurants = [];
  let transformedRestaurants = [];
  let centerCoordinates;

  // Function to fetch restaurant data
  async function fetchData() {
    try {
      const response = await fetch("/santa_barbara_restaurants.csv");
      const data = await response.text();
      const rows = data.split("\n").slice(1); // Skip header row
      // Process each row of the CSV data
      restaurants = rows.map((row) => {
        const columns = row.split(",");
        return {
          name: columns[2],
          address: columns[3],
          latitude: parseFloat(columns[7]),
          longitude: parseFloat(columns[8]),
          stars: parseFloat(columns[9]),
          review_count: parseInt(columns[10]),
          category: columns[14],
          cuisine: columns[15],
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

  // Call fetchData on component mount
  onMount(fetchData);

  // Call fetchData when the index changes
  $: {
    if (index >= 3) {
      fetchData();
    }
  }

  function getCategoryColor(cuisine) {
    // Define color mappings for different categories
    const colorMap = {
      "American": "blue",
      "Italian": "green",
      "Mexican": "orange",
      "Chinese": "red",
      "Japanese": "white",
      "Mediterranean": "olive",
    };
    return colorMap[cuisine] || "gray";
  }

  const topRatedRestaurants = transformedRestaurants
    .slice()
    .sort((a, b) => b.stars - a.stars);
</script>

<svg class="graph" width={width} height={height}>
  {#if index === 3 | index === 4 | index === 6}
    {#each transformedRestaurants as restaurant}
      <circle
        cx={restaurant.x}
        cy={restaurant.y}
        r="3"
        fill={index === 4 ? getCategoryColor(restaurant.cuisine) : "#FF5733"}
        stroke="#000000"
        stroke-width="0.5"
        title={restaurant.name}
      >
        <title>{restaurant.name}</title>
      </circle>
    {/each}
  {/if}
  {#if index === 5}
    {#each transformedRestaurants.slice(0, 50) as restaurant}
      <circle
        cx={restaurant.x}
        cy={restaurant.y}
        r="3"
        fill="#FFCCBC"
        stroke="#000000"
        stroke-width="0.5"
        title={restaurant.name}
      >
        <title>{restaurant.name}</title>
      </circle>
    {/each}
  {/if}
</svg>

<style>
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
  .legend text {
    font-size: 12px;
    fill: black;
  }
</style>
