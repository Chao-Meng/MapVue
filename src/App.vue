<template>
 <div id ="map-container">
   <!-- The button to acquire user's cuttent location-->
   <!-- <button @click="getUserLocation">Get Current Location</button> -->
   <div id="get-location-button"> 
     <button @click="getUserLocation"> 
     <img src="./map.png" alt="Get Current Location" /> 
     </button> 
     </div>
   <div>
     <!-- The search module-->
     <input v-model = "searchLocation" @keyup.enter="handleSearch" placeholder="Enter a location" />
      <button @click="handleSearch">Go</button>
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
      searchLocation: '',// 用于存储用户输入的位置名称
      searchedLocations: [],
      selectAll: false,
      selectedLocations: [],
      displayedLocations: [],
      totalPages: 1,
      latestSearchedLocation: null,
      map: null,
      marker:null,
   };
 },
 methods: {
   getUserLocation() {
     
     if("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition((position) =>{
        // this.$geolocation
        // .getCurrentPosition()
        // .then((position) => {
         const longitude = position.coords.longitude;
         const latitude = position.coords.latitude;

         //this.userLocation = {longitude,latitude};
         const userLocation = new window.google.maps.LatLng(latitude, longitude);
        //Remove the previous marker.
         if (this.marker) {
          this.marker.setMap(null);
          this.marker = null;
        }
         
         this.marker =  new window.google.maps.Marker({
          position:userLocation,
          map: this.map,
          title: 'Marker Title',
        });
        this.map.setCenter(userLocation);
       },
       (error) => {
         console.error("Error getting user location:", error.message);
      }
       );
     }else {
       alert ("This browser does not support Geolocation, please change a browser");
     }
   },
   initMap() {
    const apiKey = 'AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI';
    const script = document.createElement('script');
    script.src = `https://maps.googleapis.com/maps/api/js?key=${apiKey}&callback=initMap`;
    script.defer = true;
    script.async = true;
    script.onload = () => {
      // new window.google.maps.Map(document.getElementById('map'), {
      //     center: { lat: 43.74822, lng: -79.28941 }, // Set the initial center of the map
      //     zoom: 14, // Set the initial zoom level
      //   });
    const mapOptions = {
          center: { lat: 43.74822, lng: -79.28941 },
          zoom: 14,
        };
    this.map = new window.google.maps.Map(document.getElementById('map'), mapOptions);
        // const marker = new window.google.maps.Marker({
        //   position: { lat: 43.74822, lng: -79.28941 },
        //   map: map,
        //   title: 'Marker Title',
        // });
      };
      // Append the script to the document to load the API
      document.head.appendChild(script);
    },
    handleSearch() {
      const locationName = this.searchLocation.trim();
      if (locationName) {
        //Creating geocoding services
        const geocoder = new window.google.maps.Geocoder();
        //Converting addresses to latitude and longitude using geocoding services
        geocoder.geocode({ address: locationName }, (results, status) => {
          if (status === 'OK' && results[0]) {
            //Get the geographic coordinates of the first result
            const location = results[0].geometry.location;
            this.marker = new window.google.maps.Marker({
              position: location,
              map: this.map,
              title: locationName,
            });
            //Move the window view to the location of the search results
            this.map.setCenter(location);
        
    // this.searchedLocations.push(searchResult);
    //clear the input box
    this.searchLocation = '';
    } else {
            console.error('Geocode was not successful for the following reason: ' + status);
            alert('City not found. Please enter a valid city name.');
         } });
    }else{
      console.log('Please enter a location name.');
    }},
 },
 mounted() {
   // Initialize the map
  this.initMap();
 },

};
</script>

<style>
#map-container {
  position: relative ;
  width:100%;
  height:600px;
  z-index: 1;
}

#get-location-button {
  position: absolute;
  top:380px;
  right:-10px; 
  
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  z-index: 2;
}

#get-location-button img {
width:25px;
height:30px;
}
#get-location-button img:hover {
  background-color: #1195a4;
}
</style>


