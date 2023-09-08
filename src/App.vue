<template>
 <div id ="map-container">
   <!-- The button to acquire user's cuttent location-->
   <!-- <button @click="getUserLocation">Get Current Location</button> -->
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
  <button id="delete-button" @click="handleDelete">Delete</button>
  <!-- A table with pagination to show searched places-->
  <table id="custom-table">
    <thead>
      <tr>
        <th>
          <input id="custom-checkbox" type="checkbox" v-model="selectAll" @change="handleSelectAll"/>
        </th>
        <th>Location</th>
        <th>Time Zone</th>
        <th>Local Time</th>
      </tr>
    </thead>
    <tbody>
      <!-- Display 10 records per page-->
      <tr v-for="(location, index) in displayedLocations" :key="index">
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
  <div>
    <button id="page-button" v-for="page in totalPages" :key="page" @click="paginate(page)">{{page}}</button>
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
      searchLocation: '',// 用于存储用户输入的位置名称
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
      
    const mapOptions = {
          center: { lat: 43.74822, lng: -79.28941 },
          zoom: 14,
        };
    this.map = new window.google.maps.Map(document.getElementById('map'), mapOptions);
      };
      // Append the script to the document to load the API
      document.head.appendChild(script);
    },
  //   handleSearch() {
  //     const locationName = this.searchLocation.trim();
      
  //     if (locationName) {
  //       const formattedLocationName = locationName.toLowerCase().replace(/\b\w/g, (char) => char.toUpperCase());
  //       const existingLocationIndex = this.searchedLocations.findIndex(
  //     (location) => location.name === formattedLocationName
      
  //   );
  //    if (existingLocationIndex !== -1) {
  //     //  const existingLocation = this.searchedLocations[existingLocationIndex];
  //     const latitude = this.searchedLocations[existingLocationIndex].lat;
  //     const longitude = this.searchedLocations[existingLocationIndex].lng;
  //      fetch(`https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${Date.now() / 1000}&key=AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI&timeZoneName=true`)
  //      .then(response => response.json())
  // .then(data => {
  //     const cityTimeZone = data.timeZoneId;
  //     const localTime = DateTime.now().setZone(cityTimeZone).toLocaleString(DateTime.TIME_SIMPLE);
  //     //const formattedTimeZone = cityTimeZone.split('/').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join('/');
  //      this.searchedLocations[existingLocationIndex].timeZone = cityTimeZone;
  //      this.searchedLocations[existingLocationIndex].localTime = localTime;
  // }).catch(error => {
  //   console.error("Error fetching time zone data:", error);
  // });
  //    }else{

     
  //       //Creating geocoding services
  //       const geocoder = new window.google.maps.Geocoder();
  //       //Converting addresses to latitude and longitude using geocoding services
  //       geocoder.geocode({ address: locationName }, (results, status) => {
  //         if (status === 'OK' && results[0]) {
  //           //Get the geographic coordinates of the first result
  //           const location = results[0].geometry.location;
  //           const latitude = results[0].geometry.location.lat();
  //           const longitude = results[0].geometry.location.lng();
  //           /*Display the location on a map and add a marker to 
  //           each searched location every time the location changes.*/
  //           this.marker = new window.google.maps.Marker({
  //             position: location,
  //             map: this.map,
  //             title: locationName,
  //           });

  //           this.markers.push(this.marker);
  //           //Move the window view to the location of the search results
  //           this.map.setCenter(location);
            
  //       const searchResult = {
  //         name: formattedLocationName,
  //         timeZone: '',
  //         localTime:'',
  //          lat: latitude,
  //          lng: longitude,
  //       }
  //   // this.searchedLocations.push(searchResult);
  //    this.searchedLocations.unshift(searchResult);
  //    this.displayedLocations = this.searchedLocations.slice(0,9);
  //   //clear the input box
  //   this.searchLocation = '';
  //    this.totalPages = Math.ceil(this.searchedLocations.length / this.itemsPerPage);
  //   } else {
  //           console.error('Geocode was not successful for the following reason: ' + status);
  //           alert('City not found. Please enter a valid city name.');
  //        } });
  //        }
  //   }else{
  //     console.log('Please enter a location name.');
  //   }},
handleSearch() {
  const locationName = this.searchLocation.trim();

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
  //           each searched location every time the location changes.*/
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
      } else {
        console.error('Geocode was not successful for the following reason: ' + status);
        alert('City not found. Please enter a valid city name.');
      }
    });
  } else {
    console.log('Please enter a location name.');
  }
},
updateLocationDetails(existingLocation, latitude, longitude) {
    fetch(`https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${Date.now() / 1000}&key=AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI&timeZoneName=true`)
      .then(response => response.json())
      .then(data => {
        const cityTimeZone = data.timeZoneId;
        const localTime = DateTime.now().setZone(cityTimeZone).toFormat('yyyy-MM-dd HH:mm:ss');
      //  const cityTimeZone = data.timeZoneId;
      //   const zone = IANAZone.create(cityTimeZone);
      //   const localTime = DateTime.now().setZone(zone).toFormat('yyyy-MM-dd HH:mm:ss');
       existingLocation.timeZone = cityTimeZone;
        existingLocation.localTime = localTime;
        this.moveLocationToTop(existingLocation);
        this.searchLocation = ''
      })
      .catch(error => {
        console.error("Error fetching time zone data:", error);
      });
  },
  
  addNewLocation(name, latitude, longitude) {
    fetch(`https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${Date.now() / 1000}&key=AIzaSyBnOZude3pec_o9QgQhDNM7d2HNZ8LlGbI&timeZoneName=true`)
      .then(response => response.json())
      .then(data => {
        const cityTimeZone = data.timeZoneId;
       const localTime = DateTime.now().setZone(cityTimeZone).toFormat('yyyy-MM-dd HH:mm:ss');
       console.log('Local Time:', localTime);
       const searchResult = {
          name: name,
          timeZone: cityTimeZone,
          localTime: localTime,
          // lat: latitude,
          // lng: longitude,
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
  
  moveLocationToTop(location) {
    const index = this.searchedLocations.indexOf(location);
    if (index !== -1) {
      this.searchedLocations.splice(index, 1);
      this.searchedLocations.unshift(location);
      this.displayedLocations = this.searchedLocations.slice(0, 9);
    }
  },

    handleSelectAll() {
      
      if (this.selectAll) {
        // 如果全选被勾选，将当前页的所有行都加入 selectedLocations
      this.selectedLocations = [...this.displayedLocations];
      
      }else{
        // 如果全选被取消，清空 selectedLocations
         this.selectedLocations = [];
          
      }
     this.updateSelectAll();
    },

    updateSelectAll() {
  this.selectAll = this.selectedLocations.length === this.displayedLocations.length;
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

    handleDelete() {
  if (this.selectedLocations.length > 0) {
    const locationsToDelete = [...this.selectedLocations]; // 创建要删除的项的副本

    locationsToDelete.forEach((locationToDelete) => {
      const nameToRemove = locationToDelete.name ? locationToDelete.name.toLowerCase() : null; // 将地点名称转换为小写，如果为空则为null
 
      if (nameToRemove !== null) {
        // 使用 forEach 循环删除 markers 数组中的标记
        this.markers.forEach((marker, index) => {
          const markerTitle = marker.getTitle() ? marker.getTitle().toLowerCase() : null; // 将标记的 title 属性转换为小写，如果为空则为null
          console.log('move',markerTitle,nameToRemove);
          if (markerTitle === nameToRemove) {
           // marker.setMap(null); // 从地图上移除标记
           marker.setVisible(false);

            console.log('move',marker);
            this.markers.splice(index, 1); // 从 markers 数组中移除标记
          }
        });

        // 从 searchedLocations 中移除选中行
        const indexToRemove = this.searchedLocations.findIndex((loc) => (loc.name ? loc.name.toLowerCase() : null) === nameToRemove);
        if (indexToRemove !== -1) {
          this.searchedLocations.splice(indexToRemove, 1);
        }

        // 从 displayedLocations 中移除选中行
        const displayedIndexToRemove = this.displayedLocations.findIndex((loc) => (loc.name ? loc.name.toLowerCase() : null) === nameToRemove);
        if (displayedIndexToRemove !== -1) {
          this.displayedLocations.splice(displayedIndexToRemove, 1);
        }
      }
    });

    // 清空选中的地点
    this.selectedLocations = [];
    this.selectAll = false;
  }
},
    paginate(page){
      const startIndex = (page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      this.currentPage = page;
      this.displayedLocations = this.searchedLocations.slice(startIndex, endIndex);
    },
    

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


