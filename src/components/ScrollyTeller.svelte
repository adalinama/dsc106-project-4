<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import Map from "./Map.svelte";
  import Bar from "./Bar.svelte";
  import Search from "./Search.svelte";

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
</script>
  
<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    transition: opacity 0.5s ease;
  }

  .foreground {
    width: 60%;
    margin: 0 auto;
    height: auto;
    position: relative;
    font-size: 30px;
    font-family: American Typewriter, serif;
  }

  .progress-bars {
    position: absolute;
    background: #ADD8E6;
    opacity: 0.7;
    visibility: visible;
  }

  section {
    height: 120vh;
    background-color: rgba(0, 0, 0, 0);
    text-align: center;
    max-width: 1000px;
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
    <Map bind:geoJsonToFit {index} />
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
      You're on a daytrip to Santa Barbara with a group of your friends.
      After a long morning of exploring the various attractions that the city
      has to offer, everyone is hungry. However, the area is unfamiliar and your
      friends are indecisive, so it's difficult to narrow down food options. 
    </section>
    <section>
      <Bar />
    </section>
    <section>This is the fourth section.</section>
    <section>This is the fifth section.</section>
    <section>This is the sixth section.</section>
    <section>This is the seventh section.</section>
    <section>This is the eighth section.</section>
    <section>This is the ninth section.</section>
    <section> 
      <Search /> 
      Looking for a resturant?
      
    </section>
  </div>
</Scroller>
