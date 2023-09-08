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
  <div>
   <!-- <button id="page-button" v-for="page in totalPages" :key="page" @click="paginate(page)">{{page}}</button> -->
   <div class="pagination">
  <button v-for="page in filteredPages" :key="page" @click="paginate(page)">{{ page }}</button>
</div>

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
           this.map.setZoom(16);
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
    const allCurrentPageRowsSelected = this.displayedLocations.every((location) =>
        this.selectedLocations.includes(location)
      );

      // 自动勾选/取消全选框
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
    // 更新 displayedLocations 以包含剩余的记录
    const startIndex = (this.currentPage - 1) * this.itemsPerPage;
    const endIndex = startIndex + this.itemsPerPage;
    this.displayedLocations = this.searchedLocations.slice(startIndex, endIndex);
  }
},
   
    paginate(page) {
  const startIndex = (page - 1) * this.itemsPerPage;
  const endIndex = startIndex + this.itemsPerPage;
  const totalPages = Math.ceil(this.searchedLocations.length / this.itemsPerPage);
console.log('totalpages',totalPages);
  if (page > totalPages) {
    // 如果删除后的总页数小于当前页码，则将当前页码设置为最后一页
    this.currentPage = totalPages;
    console.log('currentpage',this.currentPage);
  } else {
    this.currentPage = page;
    console.log('currentpage',this.currentPage);
  }

  this.displayedLocations = this.searchedLocations.slice(startIndex, endIndex);
},

toggleMarker(location) {
      const marker = this.findMarkerForLocation(location);

      if (marker) {
        // 获取被点击标记的位置坐标
        const markerPosition = marker.getPosition();
        
        // 使用地图API缩放地图并将中心设置为标记位置
        this.map.setZoom(14); // 适当的缩放级别
        this.map.setCenter(markerPosition);
      }
    },

    isSelected(location) {
      return this.selectedLocations.includes(location);
    },

    findMarkerForLocation(location) {
      // 根据location查找匹配的marker
      return this.markers.find(
        (marker) => marker.getTitle().toLowerCase() === location.name.toLowerCase()
      );
    },

 },
 mounted() {
   // Initialize the map
  this.initMap();
 },
 //创建一个计算属性来返回过滤后的页码数组：
computed: {
  filteredPages() {
    const totalPages = Math.ceil(this.searchedLocations.length / this.itemsPerPage);
    return Array.from({ length: totalPages }, (_, index) => index + 1).filter(page => page > 0);
  }
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


