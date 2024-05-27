<script>
    import { onMount } from 'svelte';
    import mapboxgl from 'mapbox-gl';
    import { csv } from 'd3-fetch';
  
    onMount(async () => {
      mapboxgl.accessToken = 'pk.eyJ1IjoiYWRhbGluYW1hIiwiYSI6ImNsd2VtZGZvOTFpaG4ycXAwZnVteHlnMGcifQ.KIPf4xUPq_UDokRB5kB8NQ';
  
      const bounds = [
        [-120.0, 34.0],
        [-119.0, 35.0]  
      ];
  
      const map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/streets-v11',
        center: [-119.6982, 34.4208], 
        zoom: 12,
        maxBounds: bounds 
      });
  

      const restaurantData = await csv('santa_barbara_restaurants.csv');
  

      restaurantData.forEach(restaurant => {
        const marker = new mapboxgl.Marker()
          .setLngLat([parseFloat(restaurant.longitude), parseFloat(restaurant.latitude)])
          .addTo(map);
        
       
        const popup = new mapboxgl.Popup({ offset: 25 }).setHTML(
          `<h3>${restaurant.name}</h3><p>${restaurant.address}</p>`
        );
  
        marker.setPopup(popup);
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
  </script>
  
  <style>
    #map {
      height: 600px;
      width: 800px; 
      margin: 0 auto; 
    }
  </style>
  
  <div id="map"></div>
  