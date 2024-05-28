<script>
    import { onMount } from 'svelte';
    import mapboxgl from 'mapbox-gl';
    import { csv } from 'd3-fetch';
  
    let searchValue = ''; 
    let map; 
    let markers = []; 
  
    onMount(async () => {
      mapboxgl.accessToken = 'pk.eyJ1IjoiYWRhbGluYW1hIiwiYSI6ImNsd2VtZGZvOTFpaG4ycXAwZnVteHlnMGcifQ.KIPf4xUPq_UDokRB5kB8NQ';
  
      const bounds = [
        [-120.0, 34.0],
        [-119.0, 35.0]  
      ];
  
      map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/streets-v11',
        center: [-119.6982, 34.4208], 
        zoom: 12,
        maxBounds: bounds 
      });
  
      const restaurantData = await csv('santa_barbara_restaurants.csv');
  
      restaurantData.forEach(restaurant => {
        const marker = new mapboxgl.Marker({
          color: 'red', 
          scale: 0.7 
        })
          .setLngLat([parseFloat(restaurant.longitude), parseFloat(restaurant.latitude)])
          .addTo(map);
        
        const popup = new mapboxgl.Popup({ offset: 25 }).setHTML(
          `<h3>${restaurant.name}</h3>
          <p>${restaurant.address}</p>
          <div>
            ${getStarIcons(restaurant.stars)}
          </div>
          <div>
            <a href="https://www.yelp.com/biz/${restaurant.name.toLowerCase().replace(/['']/g, "")
            .replace(/&/g, "and").split(/[\s-]+/).join("-")}-${restaurant.city.toLowerCase().split(" ").join("-")}"
            target="_blank">Link</a>
          </div>`
        );
  
        marker.setPopup(popup);
        markers.push({ marker, name: restaurant.name.toLowerCase() }); 
      });
  
      map.on('move', () => {
        const newBounds = map.getBounds();
        const withinBounds = newBounds._ne.lng <= bounds[1][0] &&
                             newBounds._ne.lat <= bounds[1][1] &&
                             newBounds._sw.lng >= bounds[0][0] &&
                             newBounds._sw.lat >= bounds[0][1];
        
        if (!withinBounds) {
          map.fitBounds(bounds, { padding: 50 });
        }
      });
    });
  
    // Function to handle search input change
    function handleSearchInputChange(event) {
      searchValue = event.target.value.toLowerCase(); 
      markers.forEach(({ marker, name }) => {
        if (name.includes(searchValue)) {
          marker.addTo(map);
        } else {
          marker.remove();
        }
      });
    }

    // Function to generate star icons based on the rating
    function getStarIcons(stars) {
      let starsHTML = '';
      for (let i = 0; i < Math.floor(stars); i++) {
        starsHTML += '<i class="fas fa-star"></i>';
      }
      if (stars % 1 > 0) {
        starsHTML += '<i class="fas fa-star-half-alt"></i>';
      }
      return starsHTML;
    }
</script>
  
<style>
    #map {
      height: 600px;
      width: 800px; 
      margin: 0 auto; 
    }
  
    .search-container {
      text-align: center;
      margin-top: 20px;
    }
  
    .search-input {
      width: 300px;
      padding: 8px;
      font-size: 16px;
    }

    /* Customize marker size */
    .mapboxgl-marker {
      width: 20px;
      height: 20px;
    }

    /* CSS to style the Font Awesome icons */
    .fa-star,
    .fa-star-half-alt {
      color: gold; /* Set the color of the stars */
      font-size: 20px; /* Adjust the size of the stars */
    }
</style>
  
<div id="map"></div>
  
<div class="search-container">
    <input type="text" class="search-input" placeholder="Search..." bind:value={searchValue} on:input={handleSearchInputChange}>
</div>
