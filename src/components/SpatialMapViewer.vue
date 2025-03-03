<template>
  <div class="flex">
    <div id="map" class="w-3/4 h-screen"></div>
    <div class="w-1/4 p-4 bg-gray-100">
      <div v-if="selectedFeature" class="card">
        <h3 class="text-xl mb-2">Feature Details</h3>
        <pre>{{ selectedFeature.properties }}</pre>
      </div>
      <p v-else>Select a feature on the map to see details.</p>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

export default {
  data() {
    return {
      map: null,
      pointData: [],
      polygonData: [],
      selectedFeature: null,
    };
  },
  mounted() {
    // Initialize the map
    this.map = L.map('map').setView([51.505, -0.09], 13);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);
    this.fetchData();
  },
  methods: {
    async fetchData() {
      // Fetch point and polygon data from a sample API
      // const pointResponse = await fetch('/api/points');
      // const polygonResponse = await fetch('/api/polygons');

      const pointResponse = await fetch('http://localhost:5000/points');
      const polygonResponse = await fetch('http://localhost:5000/polygons');
      this.pointData = await pointResponse.json();
      this.polygonData = await polygonResponse.json();
      this.addMarkers();
      this.addPolygons();
    },
    addMarkers() {
      // Add point markers to the map
      this.pointData.forEach(point => {
        const marker = L.marker([point.lat, point.lng]).addTo(this.map);
        marker.bindPopup(point.name);
      });
    },
    addPolygons() {
      // Add polygon layers to the map
      const geoJsonLayer = L.geoJSON(this.polygonData, {
        onEachFeature: (feature, layer) => {
          layer.on('click', () => {
            this.selectedFeature = feature;
          });
        }
      }).addTo(this.map);
    }
  }
};
</script>

<style>
#map { height: 100vh; }
</style>
