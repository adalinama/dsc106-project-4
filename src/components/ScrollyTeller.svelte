<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import Map from "./Map.svelte";
  import Bar from "./Bar.svelte";
  import Search from "./Search.svelte";
  import { geoMercator } from "d3-geo";
  import { onMount } from "svelte";

  let count, index, offset, progress;
  let width, height;

  let geoJsonToFit = {
    type: "FeatureCollection",
    features: [
      {
        type: "Feature",
        geometry: {
          type: "Point",
          coordinates: [1, 0],
        },
      },
      {
        type: "Feature",
        geometry: {
          type: "Point",
          coordinates: [0, 1],
        },
      },
    ],
  };
  
  $: projection = geoMercator().fitSize([width, height], geoJsonToFit);
</script>
  
<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    transition: opacity 0.5s ease; /* Add transition for smooth opacity change */
  }

  .foreground {
    width: 80%;
    margin: 0 auto;
    height: auto;
    position: relative;
    font-size: 30px;
    font-family: American Typewriter, serif;
  }

  .section-text {
    position: -webkit-sticky; /* Safari */
    position: sticky;
    top: 0;
    background-color: rgba(255, 255, 255, 0);
    padding: 10px;
    z-index: 1;
  }

  .progress-bars {
    position: absolute;
    background: #ADD8E6; /*  20% opaque */
    opacity: 0.7;
    visibility: visible;
  }

  section {
    height: 120vh;
    background-color: rgba(0, 0, 0, 0); /* 20% opaque */
    /* color: white; */
    text-align: center;
    max-width: 1000px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }
</style>
  
<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>
  <div class="background" slot="background" bind:clientWidth={width} bind:clientHeight={height} style="opacity: {progress < 0.9 ? 1 : (1 - progress)};">
    <div style="position: relative; z-index: -1;">
      <Map bind:geoJsonToFit {index} />
    </div>

    <div class="progress-bars">
      <p>current section: <strong>{index + 1}/{count}</strong></p>
      <progress value={count ? (index + 1) / count : 0} />
      
      <p>offset in current section</p>
      <progress value={progress || 0} />
      
      <p>total progress</p>
      <progress value={progress || 0} />
    </div>
  </div>
  
  <div class="foreground" slot="foreground">
    <section>
      <h1>Flavors of Santa Barbara</h1>
      <p>A guide to the American Riviera's best culinary experiences</p>
    </section>
    <section>
      <p>You're on a daytrip to Santa Barbara with a group of your friends.
      After a long morning of exploring the various attractions that the city
      has to offer, everyone is hungry. However, the area is unfamiliar and your
      friends are indecisive, so it's difficult to narrow down food options. </p>

      <p>It turns out Santa Barbara is very diverse! From Mexican food to Japanese food,
      there are many options to choose from. Each restaurant has its own set of category tags
      that accurately describe the type of restaurant. Let's view what some of the most popular
      categories are:</p>
    </section>
    <section>
      <Bar />
      <p>This could give you a good idea of the diversity of restaurants to choose from. 
        However, it might be more useful to look at a map and to determine which restaurants are in
        your vicinity. Let's take a look at the spread of all restaurants in Santa
        Barbara County.
      </p>
    </section>
    <section>
      <div class="section-text">
        <p>{"All Restaurants in Santa Barbara"}</p>
      </div>
    </section>
    <section>
      <div class="section-text">
        <p>{"Restaurants by Cuisine"}</p>
      </div>
    </section>
    <section>
      <div class="section-text">
        <p>{"Highest Rated Restaurants"}</p>
      </div>
    </section>
    <section>
      <div class="section-text">
        <p>{"Most Reviewed Restaurants"}</p>
      </div>
    </section>
    <section>
      <p>Are you ready to find the perfect restaurant for you? Use the search bar below to
        discover the dining experience that suits your taste.
      </p>
      <Search /> 
    </section>
  </div>
</Scroller>