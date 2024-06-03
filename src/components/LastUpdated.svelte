<script>
    export let lastUpdated = "2024-06-01"; // Default value, can be updated dynamically if needed
  </script>
  
  <style>
    .last-updated {
      position: fixed;
      bottom: 0;
      width: 100%;
      text-align: center;
      background-color: #f8f9fa; /* Light background color */
      padding: 10px 0;
      font-size: 14px;
      color: #6c757d; /* Gray color for text */
    }
  </style>
  
  <div class="last-updated">
    Last updated on: {lastUpdated}
  </div>
  