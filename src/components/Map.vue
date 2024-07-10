<script setup>
import mapboxgl from 'mapbox-gl';
import { ref, watch, onMounted, onUnmounted } from 'vue'
mapboxgl.accessToken = import.meta.env.VITE_APP_MAPBOX_ACCESS_TOKEN // <UserAccessToken />;

const props = defineProps({
    modelValue:
    {
        type: Object,
        required: true
    }
});

const emit = defineEmits(['update:modelValue']);

const mapContainer = ref(null);
const thismap = ref(null);

onMounted(() => {
    const { lng, lat, zoom, bearing, pitch } = props.modelValue;
    
    const map = new mapboxgl.Map({
        container: mapContainer.value,
        style: "mapbox://styles/mapbox/streets-v12", 
        center: [lng, lat],
        bearing,
        pitch,
        zoom
    });

    // Updates the location and emits an event with the new location
    const updateLocation = () => {
        emit('update:modelValue', getLocation());
    }

    map.on('move', updateLocation);
    map.on('zoom', updateLocation);
    map.on('rotate', updateLocation);
    map.on('pitch', updateLocation);
    
    thismap.value = map;
        
});

onUnmounted(() => {
    thismap.value.remove();
    thismap.value = null;
    }); 

// this is triggered any time modelValue changes.
watch (() => props.modelValue, (next) => {
    const curr = getLocation();
    const map = thismap.value;
    if (curr.lng != next.lng || curr.lat != next.lat)
    map.setCenter({ lng: next.lng, lat: next.lat });
    if (curr.pitch != next.pitch) map.setPitch(next.pitch);
    if (curr.bearing != next.bearing) map.setBearing(next.bearing);
    if (curr.zoom != next.zoom) map.setZoom(next.zoom);
});

// A function that retrieves the location details including center, bearing, pitch, and zoom.
function getLocation() {
    return {
        ...thismap.value.getCenter(),
        bearing: thismap.value.getBearing(),
        pitch: thismap.value.getPitch(),
        zoom: thismap.value.getZoom()
    }
}

</script>

<template>
    <div ref="mapContainer" class="map-container"></div>
</template>  

<style>
.map-container {
    flex: 1;
}
</style>