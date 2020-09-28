<script>
 import {onMount} from 'svelte';
 import PickAPlace from 'svelte-pick-a-place';
 import  'leaflet/dist/leaflet.css';   
 let leaflet;
 onMount(async () => {
     leaflet = await import('leaflet');
     });
</script>
<style>
 .wrapper {
     width:100%;
     height:600px;
 }
</style>
{#if leaflet}
    <h5>pickaplace:</h5>
    <div class='wrapper'>
<PickAPlace leaflet={leaflet} on:update={({detail}) => console.log('Update!',detail)} on:save={({detail}) =>
									       console.log('On save!',detail)} />
</div>
{:else}
<p>leaflet loading</p>
{/if}

