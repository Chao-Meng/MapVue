<template>
 <div id ="map-container">
   <!-- The button to acquire user's cuttent location-->
   <div id="get-location-button"> 
     <button @click="getUserLocation"> 
     <img src="./map.png" alt="Get Current Location" /> 
     </button> 
     </div>
  
  <!-- Display the location on the map-->
  <div id="map" style="height: 600px; width: 100%;"></div>

   <div id="search-container">
     <!-- The search module-->
     <input v-model = "searchLocation" @keyup.enter="handleSearch" placeholder="Enter a location" />
      <button id="map-button" @click="handleSearch">Go</button>
   </div>

<!-- Delete button to remove selected records and markers -->
  <button id="delete-button" @click="showDeleteConfirmation">Delete</button>

  <!-- A table with pagination to show searched places-->
  <table id="custom-table">
    <thead>
      <tr>
        <th>
          <!-- A checkbox at the beginning of each row to let users select multiple records at the same time. -->
          <!-- //<input id="custom-checkbox" type="checkbox" v-model="selectAll" @change="handleSelectAll"/> -->
           <input id="custom-checkbox" type="checkbox" v-model="selectAll" @change="handleSelectAll" :disabled="!hasSearchedLocations" />
        </th>
        <th id="column-33">Location</th>
        <th id="column-33">Time Zone</th>
        <th id="column-33">Local Time</th>
      </tr>
    </thead>
    <tbody>
      <!-- Display 10 records per page-->
      <tr v-for="(location, index) in displayedLocations" :key="index"
      @click="toggleMarker(location)"
      :class="{ 'selected-row': isSelected(location) }">
        <td>
          <input id="custom-checkbox" type="checkbox" v-model="selectedLocations" :value="location" @change="handleSelect" />
        </td>
        <td>{{location.name}}</td>
        <td>{{location.timeZone}}</td>
        <td>{{location.localTime}}</td>
      </tr>
    </tbody>
  </table>

  <!-- Pagination buttons -->
  <div class="pagination">
    <button id="page-button" v-for="page in filteredPages" :key="page" @click="paginate(page)">{{ page }}</button>
  </div>
  
    <!-- Display the time zone and local time of the latest searched location -->
  <div v-if="latestSearchedLocation">
    <p>Searched location:</p>
    <p>Location: {{ latestSearchedLocation.name }}</p>
    <p>Time Zone: {{ latestSearchedLocation.timeZone }}</p>
    <p>Local Time: {{ latestSearchedLocation.localTime }}</p>
  </div>
 </div> 
</template>

<script>
import { DateTime} from 'luxon';
export default {
 data() {
   return {
     userLocation: null,
      searchLocation: '',// store the location name entered by the user
      searchedLocations: [],
      selectAll: false,
      selectedLocations: [],
      displayedLocations: [],
      totalPages: 1,
      latestSearchedLocation: null,
      map: null,
      marker:null,
      markers:[],
      currentPage: 1,
      itemsPerPage: 10,
      selectedRows:[],
      locations:[],
      userLocationMarker: null,
   };
 },
 methods: {
   getUserLocation() {  
     if("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition((position) =>{
         const longitude = position.coords.longitude;
         const latitude = position.coords.latitude;
         const userLocation = new window.google.maps.LatLng(latitude, longitude);
        this.map.setZoom(16);
        //Remove the previous marker.
         if (this.userLocationMarker) {
         // this.marker.setMap(null);
          this.userLocationMarker.setVisible(false);
          console.log('user location:',userLocation);
          //this.marker = null;
        }        
         this.userLocationMarker =  new window.google.maps.Marker({
          position:userLocation,
          map: this.map,
          title: 'Marker Title',
        });    
        this.map.setCenter(userLocation);       
       },
       (error) => {
         console.error("Error getting user location:", error.message);
        });
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
    const mapOptions = {
          center: { lat: 43.74822, lng: -79.28941 },
          zoom: 14,
          minZoom:3,
        };
    this.map = new window.google.maps.Map(document.getElementById('map'), mapOptions);
      };
      // Append the script to the document to load the API
      document.head.appendChild(script);
    //  google.maps.event.addListener(this.map, 'zoom_changed', () => {
    //       if (this.map.getZoom() < 2) {
    //         this.map.setZoom(2);
    //       }
    //     });
    },
 
  handleSearch() {
    const locationName = this.searchLocation.trim();
    if (locationName.length === 0) {
      alert('Please input a city.');
      return; 
    }
    if (locationName) {
      const formattedLocationName = locationName.toLowerCase().replace(/\b\w/g, (char) => char.toUpperCase());
      const existingLocationIndex = this.searchedLocations.findIndex(
        (location) => location.name === formattedLocationName
      );
      // Creating geocoding services
      const geocoder = new window.google.maps.Geocoder();
      // Converting addresses to latitude and longitude using geocoding services
      geocoder.geocode({ address: locationName }, (results, status) => {
        if (status === 'OK' && results[0]) {
          // Get the geographic coordinates of the first result
          const location = results[0].geometry.location;
          const latitude = location.lat();
          const longitude = location.lng();
          /*Display the location on a map and add a marker to 
          // each searched location every time the location changes.*/
          this.marker = new window.google.maps.Marker({
            position: location,
            map: this.map,
            title: locationName,
          });
          this.markers.push(this.marker);
          //Move the window view to the location of the search results
          this.map.setCenter(location);
          // Check if the location already exists
          if (existingLocationIndex !== -1) {
            const existingLocation = this.searchedLocations[existingLocationIndex];
            this.updateLocationDetails(existingLocation, latitude, longitude);
          } else {
            this.addNewLocation(formattedLocationName, latitude, longitude);
          }
           this.map.setZoom(14);  
        } else {
          console.error('Geocode was not successful for the following reason: ' + status);
          alert('City not found. Please enter a valid city name.');
        }
      });
    } else {
      console.log('Please enter a location name.');
    }
  },
//If the city the user searched for already exists, update the information and move its position to the top of the table.
  updateLocationDetails(existingLocation, latitude, longitude) {
      fetch(`https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${Date.now() / 1000}
      &key=AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI&timeZoneName=true`)
        .then(response => response.json())
        .then(data => {
          const cityTimeZone = data.timeZoneId;
          const localTime = DateTime.now().setZone(cityTimeZone).toFormat('yyyy-MM-dd HH:mm:ss');
          existingLocation.timeZone = cityTimeZone;
          existingLocation.localTime = localTime;
          this.moveLocationToTop(existingLocation);
          this.searchLocation = ''
        })
        .catch(error => {
          console.error("Error fetching time zone data:", error);
        });
    },
  //If the city the user searched for does not exist, add new information about it.
  addNewLocation(name, latitude, longitude) {
    fetch(`https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${Date.now() / 1000}
    &key=AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI&timeZoneName=true`)
      .then(response => response.json())
      .then(data => {
        const cityTimeZone = data.timeZoneId;
       const localTime = DateTime.now().setZone(cityTimeZone).toFormat('yyyy-MM-dd HH:mm:ss');
       console.log('Local Time:', localTime);
       const searchResult = {
          name: name,
          timeZone: cityTimeZone,
          localTime: localTime,
        };
        this.searchedLocations.unshift(searchResult);
        this.displayedLocations = this.searchedLocations.slice(0, 9);
        this.searchLocation = '';
        this.totalPages = Math.ceil(this.searchedLocations.length / this.itemsPerPage);
      })
      .catch(error => {
        console.error("Error fetching time zone data:", error);
      });
    },

  //Update the most recently research to the top of the table.
  moveLocationToTop(location) {
    const index = this.searchedLocations.indexOf(location);
    if (index !== -1) {
      this.searchedLocations.splice(index, 1);
      this.searchedLocations.unshift(location);
      this.displayedLocations = this.searchedLocations.slice(0, 9);
    }
  },

  handleSelectAll() {  
    if (!this.hasSearchedLocations) {
      return;
    }
    if (this.selectAll) {
    // If Select All is checked, adds all rows on the current page to the selectedLocations
      this.selectedLocations = [...this.displayedLocations];
    }else{
      // Clear selectedLocations if unselected any.
      this.selectedLocations = [];   
    }
    this.updateSelectAll();
  },

  updateSelectAll() {
  const allCurrentPageRowsSelected = this.displayedLocations.every((location) =>
      this.selectedLocations.includes(location)
    );
    // Auto-check/uncheck all boxes
    this.selectAll = allCurrentPageRowsSelected;
  },

  handleSelect(location) {  
    const index = this.selectedLocations.indexOf(location);
      if (index !== -1) {
        this.selectedLocations.splice(index, 1);
      } else {
        this.selectedLocations.push(location);
      }
      this.updateSelectAll();
  },

//Display a alert box prompting the user whether delete or not.
 showDeleteConfirmation() {
    if (this.selectedLocations.length > 0) {
      const confirmMessage = "Are you sure you want to delete?";
      const userConfirmed = confirm(confirmMessage);
      if (userConfirmed) {
        //if user click OK button, perform the delete operation
        this.executeDelete();
      } 
    } else {
      //If the user does not check any valid rows and clicks the delete button, an alert is given.  
      alert(' Please check the information you want to delete first');
    }
  },

  executeDelete() {
    const locationsToDelete = [...this.selectedLocations];
    locationsToDelete.forEach((locationToDelete) => {
      // Convert location names to lowercase or null if empty
      const nameToRemove = locationToDelete.name ? locationToDelete.name.toLowerCase() : null; 
      if (nameToRemove !== null) {
        // Use a forEach loop to remove markers from the markers array.
        this.markers.forEach((marker, index) => {
          // Convert the tag's title attribute to lowercase, or null if it's empty
          const markerTitle = marker.getTitle() ? marker.getTitle().toLowerCase() : null; 
          console.log('move',markerTitle,nameToRemove);
          if (markerTitle === nameToRemove) {
           // marker.setMap(null); // Remove marker from map
           marker.setVisible(false);
           console.log('move',marker);
          // Remove marker from markers
            this.markers.splice(index, 1); 
          }
        });

        // remove the selected row from searchedLocations 
        const indexToRemove = this.searchedLocations.findIndex((loc) => (loc.name ? loc.name.toLowerCase() : null) === nameToRemove);
        if (indexToRemove !== -1) {
          this.searchedLocations.splice(indexToRemove, 1);
        }

        //remove the selected row from displayedLocations
        const displayedIndexToRemove = this.displayedLocations.findIndex((loc) => (loc.name ? loc.name.toLowerCase() : null) === nameToRemove);
        if (displayedIndexToRemove !== -1) {
          this.displayedLocations.splice(displayedIndexToRemove, 1);
        }
        this.rowsSelected = false;
      } 
    });

    // Clear the selected locations
    this.selectedLocations = [];
    this.selectAll = false;
    this.rowsSelected = false;

    //update displayedLocations to show the remaining records
    const startIndex = (this.currentPage - 1) * this.itemsPerPage;
    const endIndex = startIndex + this.itemsPerPage;
    this.displayedLocations = this.searchedLocations.slice(startIndex, endIndex);
  },
  paginate(page) {
    const startIndex = (page - 1) * this.itemsPerPage;
    const endIndex = startIndex + this.itemsPerPage;
    const totalPages = Math.ceil(this.searchedLocations.length / this.itemsPerPage);
    console.log('totalpages',totalPages);
    if (page > totalPages) {
      /* If the total number of pages after deletion is less than the current page number
      set the current page number as the last page*/
      this.currentPage = totalPages;
      console.log('currentpage',this.currentPage);
    } else {
      this.currentPage = page;
      console.log('currentpage',this.currentPage);
    }
    this.displayedLocations = this.searchedLocations.slice(startIndex, endIndex);
    //When turning pages, update the status of all checked according to whether
    // the current row is fully checked or not.
    this.updateSelectAll(); 
  },
  
//If the user clicks on a row in the table, the map shows that area and displays the marker
  toggleMarker(location) {
    const marker = this.findMarkerForLocation(location);
    if (marker) {
      // Get the coordinates of the clicked marker's position
      const markerPosition = marker.getPosition(); 
      // Use the Maps API to zoom the map and set the center to the marker location
      this.map.setZoom(14); 
      this.map.setCenter(markerPosition);
    }
  },

  isSelected(location) {
    return this.selectedLocations.includes(location);
  },

  findMarkerForLocation(location) {
    // Find matching markers based on location
    return this.markers.find(
      (marker) => marker.getTitle().toLowerCase() === location.name.toLowerCase()
    );
    },
  },
  mounted() {
    // Initialize the map
    this.initMap();
  },
 //Creates a calculated property to return an array of filtered page numbers.
  computed: {
    filteredPages() {
      const totalPages = Math.ceil(this.searchedLocations.length / this.itemsPerPage);
      return Array.from({ length: totalPages }, (_, index) => index + 1).filter(page => page > 0);
    },
    hasSearchedLocations() {
    // 判断是否有搜索记录
    return this.searchedLocations.length > 0;
  },

  }
};
</script>

<style>
  #map-container {
    position: relative ;
    width:100%;
    height:600px;
    z-index: 1;  
  }

  #map-button{
      font-size:15px;
      padding:3px 5px;
      margin:2px 2px;
      background-color:#ddd;
      color:black;
      border:none;
      border-radius:3px;
      cursor:pointer;
      transition:background-color 0.3s
  }
  #map-button:hover {
      background-color: #1195a4;
  }

  #search-container{
      display:flex;
    justify-content:center;
    align-items:center;
  }

  #get-location-button {
    position: absolute;
    top:360px;
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

  #custom-table{
      width: 100%;
      border-collapse: collapse;
      font-size: 14px;
  }

  #custom-table thead th {
      background-color: #1195a4;
      padding: 10px;
      text-align: left;
      font-weight: bold;
      border: 1px solid #ddd;
    }

  #custom-table tbody td {
      padding: 10px;
      text-align: left;
      border: 1px solid #ddd;
  }
  #custom-table tbody tr:hover {
    background-color: #1195a4;
    cursor: pointer;
  }

  #column-33{
    width: 33%;
  }

  #custom-checkbox{
    width: 16px;
    height: 16px;
  }

  #pagination-container{
      display: flex;
      justify-content: flex-start;
      align-items: center;
      margin-top: 5px;
  }

  #page-button {
    font-size:15px;
    padding:3px 5px;
    margin:2px 2px;
    background-color:#ddd;
    color:black;
    border:none;
    border-radius:3px;
    cursor:pointer;
    transition:background-color 0.3s;
  }

  #page-button:hover {
    background-color: #1195a4;
  }

  #delete-button{
      font-size:15px;
      padding:3px 5px;
      margin:2px 2px;
      background-color:#ddd;
      color:black;
      border:none;
      border-radius:3px;
      cursor:pointer;
      transition:background-color 0.3s
  }
  #delete-button:hover {
      background-color: #1195a4;
  }

</style>


