<template>
 <div>
   <!-- The button to acquire user's cuttent location-->
   <button>Get Current Location</button>
   <div>
     <!-- The search module-->
      <button>Go</button>
   </div>
  <!-- Display the location on the map-->
  <div id="map" style="height: 600px; width: 100%;"></div>

  <!-- A table with pagination to show searched places-->
  <table>
    <thead>
      <tr>
        <th>
          <input type="checkbox" />
        </th>
        <th>Location</th>
        <th>Time Zone</th>
        <th>Local Time</th>
      </tr>
    </thead>
    <tbody>
      <!-- Display 10 records per page-->
      <tr>
        <td>
          <input type="checkbox" />
        </td>
        <td>name</td>
        <td>timeZone</td>
        <td>localtime</td>
      </tr>
    </tbody>
  </table>

  <!-- Pagination buttons -->
  <div>
    <button></button>
  </div>

  <!-- Delete button to remove selected records and markers -->
  <button>Delete</button>

  <!-- Display the time zone and local time of the latest searched location -->
  <div>
    <p></p>
  </div>
 </div>
  
</template>

<script>
export default {
 data() {
   return {
     userLocation: null,
      searchLocation: '',
      searchedLocations: [],
      selectAll: false,
      selectedLocations: [],
      displayedLocations: [],
      totalPages: 1,
      latestSearchedLocation: null,
   };
 },
 methods: {
   getUserLocation() {

   },
   initMap() {
    const apiKey = 'AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI';
    const script = document.createElement('script');
    script.src = `https://maps.googleapis.com/maps/api/js?key=${apiKey}&callback=initMap`;
    script.defer = true;
    script.async = true;
    script.onload = () => {
      new window.google.maps.Map(document.getElementById('map'), {
          center: { lat: 43.74822, lng: -79.28941 }, // Set the initial center of the map
          zoom: 14, // Set the initial zoom level
        });

        // const marker = new window.google.maps.Marker({
        //   position: { lat: 43.74822, lng: -79.28941 },
        //   map: map,
        //   title: 'Marker Title',
        // });
      };
      document.head.appendChild(script);
    },
 },
 mounted() {
  this.initMap();
 },

};
</script>

<style>
#app {
  
}
</style>


