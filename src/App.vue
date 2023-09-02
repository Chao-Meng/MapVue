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
  <button>Detlete</button>

  <!-- Display the time zone and local time of the latest searched location -->
  <div>
    <p></p>
  </div>
 </div>
  
</template>

<script>
export default {
 
}
</script>

<style>
#app {
  
}
</style>


