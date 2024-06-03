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
    margin-right: 105px;
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

  .grid-content { 
    /* displays images side by side */
    width: 73%; 
    margin-left: 20px; 
    height: 28%; 
    padding: 20px; 
    display: grid; 
    grid-template-columns: repeat(3, 1fr); 
    gap: 130px;
  } 

  .grid-content-2 { 
    /* displays images side by side */
    width: 73%; 
    height: 20%; 
    padding: 5px; 
    display: grid; 
    grid-template-columns: repeat(3, 1fr); 
    gap: 60px;
  } 

  .sbimage1, .sbimage2, .foodimage1, .foodimage2, .foodimage3 {
  /* create shadow effect */
  box-shadow: 0 0 10px 5px rgba(0,0,0,0.3);
  border-radius: 15px;
}

.welcome {
  /* create slightly smaller shadow effect */
  box-shadow: 0 0 10px 5px rgba(0,0,0,0.3);
  border-radius: 10px;
  margin-top: 160px;
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

    
  </div>
  
  <div class="foreground" slot="foreground">
    <section>
      <h1>Flavors of Santa Barbara</h1>
      <p>A guide to the American Riviera's best culinary experiences</p>
      <p>By Mark Lee, Adalina Ma and Eric Stratford</p>
      <img class="welcome" src="/welcome_sign.png" alt="welcome-sign" width="400" height="250">
    </section>
    <section>
      <div class="grid-content">
        <img class="sbimage1" src="/sbbeach.jpeg" alt="Beach" width="400" height="250"> 
  
      <img class="sbimage2" src="/sbdowntown.jpeg" alt="Downtown" width="400" height="250" style="top-margin: 500px;">
    </div>

      <p>You're on a daytrip to Santa Barbara with a group of your friends.
      After a long morning of exploring the various attractions that the city
      has to offer, everyone is hungry. However, the area is unfamiliar and your
      friends are indecisive, so it's difficult to narrow down food options. </p>

      <p>It turns out Santa Barbara is very diverse! From Mexican food to Japanese food,
      there are many options to choose from.</p>

      <div class="grid-content-2">
        <img class="foodimage1" src="/pasta.jpeg" alt="Pasta" width="300" height="180"> 
  
      <img class="foodimage2" src="/chinese.webp" alt="Chinese" width="300" height="180">

      <img class="foodimage3" src="/seafood.webp" alt="Seafood" width="300" height="180">
    </div>
      
      <p>Each restaurant has its own set of category tags that accurately describe the 
      type of restaurant. Let's view what some of the most popular categories are:</p>
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
      <p style="margin-bottom: 105px">Are you ready to find the perfect restaurant for you?
        Use the search bar below to discover the dining experience that suits your taste.
        <br>
        <small style="font-size:small">(Drag the map around to explore Santa Barbara. If 
          a restaurant seems appealing to you, click on the link to direct you to the
          restaurant's Yelp page, where you can check dining hours, view the menu, and even 
          make reservations.)</small>

      </p>
      
      <Search /> 
    </section>
  </div>
</Scroller>