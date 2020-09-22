<script>
 /*
    This is an example of using Svelte features with Leaflet. Original blog post here: https://imfeld.dev/writing/leaflet_with_svelte
    
    The toolbar and the marker popups are both implemented by embedding Svelte components inside Leaflet elements. The marker and lines are toggled by updating the map from reactive statements.
    
    Any questions? Ask me at dimfeld on Twitter!
    
    Thanks to heroicons.dev for all the icons used here.
  */

 import {onMount} from 'svelte';
 import MapToolbar from '../components/MapToolbar.svelte';
 import MarkerPopup from '../components/MarkerPopup.svelte';
 import  'leaflet/dist/leaflet.css';  
 import * as markerIcons from '../lib/markers.js';
 let map;
 
 const l = (...a) => console.log(...a);
 let L;
 let toolbar;

 onMount(async () => {

      const mod = await import('leaflet');
      L = mod;
      toolbar = L.control({ position: 'topright' });

      toolbar.onAdd = (map) => {
	  let div = L.DomUtil.create('div');
	  toolbarComponent = new MapToolbar({
	      target: div,
	      props: {}
		     });

	      toolbarComponent.$on('click-eye', ({ detail }) => eye = detail);
	      toolbarComponent.$on('click-lines', ({ detail }) => lines = detail);
	      toolbarComponent.$on('click-reset', () => {
		  map.setView(initialView, 5, { animate: true })
	      })

	      return div;
	  }

	  toolbar.onRemove = () => {
	      if(toolbarComponent) {
		  toolbarComponent.$destroy();
		  toolbarComponent = null;
	      }
	  };      
      l('toolbar',toolbar);
      //l('mod',mod);
    });
 



    



      const markerLocations = [
	  [29.8283, -96.5795],
	  [37.8283, -90.5795],
	  [43.8283, -102.5795],
	  [48.40, -122.5795],
	  [43.60, -79.5795],
	  [36.8283, -100.5795],
	  [38.40, -122.5795],
      ];
 
      const initialView = [39.8283, -98.5795];
      function createMap(container) {
	  l('in createMap');
	  let m = L.map(container, {preferCanvas: true }).setView(initialView, 5);
	  l('map initialized',m);

	  L.tileLayer(
	      'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
	      {
		  attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OpenStreetMap</a>,
	        &copy;<a href="https://carto.com/attributions" target="_blank">CARTO</a>`,
		  subdomains: 'abcd',
		  maxZoom: 14,
	      }
	  ).addTo(m);
	  l('createMap has run.');
	  return m;
      }

      let eye = true;
      let lines = true;
      
      let toolbarComponent;



     
     // Create a popup with a Svelte component inside it and handle removal when the popup is torn down.
     // `createFn` will be called whenever the popup is being created, and should create and return the component.
      function bindPopup(marker, createFn) {
	  l('bindPopup');
	  let popupComponent;
	  marker.bindPopup(() => {
	      let container = L.DomUtil.create('div');
	      popupComponent = createFn(container);
	      return container;
	  });

	  marker.on('popupclose', () => {
	      if(popupComponent) {
		  let old = popupComponent;
		  popupComponent = null;
		  // Wait to destroy until after the fadeout completes.
		  setTimeout(() => {
		      old.$destroy();
		  }, 500);

	      }
	  });
      }
     
     let markers = new Map();
     
      function markerIcon(count) {
	  l('markerIcon');
	  let html = `<div class="map-marker"><div>${markerIcons.library}</div><div class="marker-text">${count}</div></div>`;
	  return L.divIcon({
	      html,
	      className: 'map-marker'
	  });
      }
     

      function createMarker(loc) {
	  l('createMarker');
	  let count = Math.ceil(Math.random() * 25);
	  let icon = markerIcon(count);
	  let marker = L.marker(loc, {icon});
	  bindPopup(marker, (m) => {
	      let c = new MarkerPopup({
		  target: m,
		  props: {
		      count
		  }
	      });
	      
	      c.$on('change', ({detail}) => {
		  count = detail;
		  marker.setIcon(markerIcon(count));
	      });
	      
	      return c;
	  });
	  
	  return marker;
      }
     
      function createLines() {
	  l('createLines');
	  return L.polyline(markerLocations, { color: '#E4E', opacity: 0.5 });
      }



     let markerLayers;
     let lineLayers;

     // We could do these in the toolbar's click handler but this is an example
     // of modifying the map with reactive syntax.
     $: if(markerLayers && map) {
	 if(eye) {
	     markerLayers.addTo(map);
	 } else {
	     markerLayers.remove();
	 }
     }
     
     $: if(lineLayers && map) {
	 if(lines) {
	     lineLayers.addTo(map);
	 } else {
	     lineLayers.remove();
	 }
     }


      function mapAction(container) {
	  l('mapAction triggered');
	  map = createMap(container); 
	  toolbar.addTo(map);
	  
	  markerLayers = L.layerGroup()
 	  for(let location of markerLocations) {
 	      let m = createMarker(location);
	      markerLayers.addLayer(m);
 	  }
	  
	  lineLayers = createLines();
	  
	  markerLayers.addTo(map);
	  lineLayers.addTo(map);
	  
	  return {
	      destroy: () => {
		  toolbar.remove();
		  map.remove();
		  map = null;
	      }
	  };
      }
     
      function resizeMap() {
	  l('resizemap triggered');
	 if(map) { map.invalidateSize(); }
     }

</script>
<svelte:window on:resize={resizeMap} />

<style>

	.map :global(.marker-text) {
		width:100%;
		text-align:center;
		font-weight:600;
		background-color:#444;
		color:#EEE;
		border-radius:0.5rem;
	}
	
	.map :global(.map-marker) {
		width:30px;
		transform:translateX(-50%) translateY(-25%);
	}
 .mapContainer {
     width:100%;
     height:600px;
     }
</style>

{#if L}
    <div class='mapContainer'>
	<div class="map" style="height:100%;width:100%" use:mapAction />
    </div>
{:else}
<h4>map loading</h4>
{/if}
