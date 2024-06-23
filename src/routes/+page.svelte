<script lang="ts">
  /* global google */

  import { Loader } from '@googlemaps/js-api-loader';
  import { onMount } from 'svelte';
  import SearchBar from './components/SearchBar.svelte';
  import Sections from './sections/Sections.svelte';

  const googleMapsApiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
  const defaultPlace = {
    name: 'THE SOLAR HOUSE',
    address: '4 Forest Glen Court, OFallon, MO 63368',
  };
  let location: google.maps.LatLng | undefined;
  const zoom = 19;
  let mapElement: HTMLElement;
  let map: google.maps.Map;
  let infowindow: google.maps.InfoWindow;
  let geometryLibrary: google.maps.GeometryLibrary;
  let mapsLibrary: google.maps.MapsLibrary;
  let placesLibrary: google.maps.PlacesLibrary;

  onMount(async () => {
    // Load the Google Maps libraries.
    const loader = new Loader({ apiKey: googleMapsApiKey });
    const libraries = {
      geometry: await loader.importLibrary('geometry'),
      maps: await loader.importLibrary('maps'),
      places: await loader.importLibrary('places'),
    };

    geometryLibrary = libraries.geometry;
    mapsLibrary = libraries.maps;
    placesLibrary = libraries.places;

    // Get the address information for the default location.
    const geocoder = new google.maps.Geocoder();
    const geocoderResponse = await geocoder.geocode({
      address: defaultPlace.address,
    });
    const geocoderResult = geocoderResponse.results[0];

    // Initialize the map at the desired location.
    location = geocoderResult.geometry.location;
    map = new mapsLibrary.Map(mapElement, {
      center: location,
      zoom: zoom,
      tilt: 0,
      mapTypeId: 'satellite',
      mapTypeControl: true,
      fullscreenControl: true,
      rotateControl: true,
      streetViewControl: false,
      zoomControl: true,
    });

    infowindow = new google.maps.InfoWindow();

    // Configure the click listener.
    map.addListener("click", (mapsMouseEvent) => {
      addPin(mapsMouseEvent.latLng);
    });
  });

  function addPin(latLng: google.maps.LatLng) {
    const marker = new google.maps.Marker({
      position: latLng,
      map: map,
      draggable: true,
    });

    // Reverse geocode to get the address
    const geocoder = new google.maps.Geocoder();
    geocoder.geocode({ location: latLng }, (results, status) => {
      if (status === 'OK') {
        if (results[0]) {
          const address = results[0].formatted_address;
          const contentString = `<div><b>Address:</b> ${address}</div>`;
          infowindow.setContent(contentString);
          infowindow.open(map, marker);
        } else {
          console.error('No results found');
        }
      } else {
        console.error(`Geocoder failed due to: ${status}`);
      }
    });
  }
</script>

<!-- Top bar -->
<div class="flex flex-row h-full">
  <!-- Main map -->
  <div bind:this={mapElement} class="w-full" />

  <!-- Side bar -->
  <aside class="flex-none md:w-96 w-80 p-2 pt-3 overflow-auto">
    <div class="flex flex-col space-y-2 h-full">
      {#if placesLibrary && map}
      <!-- Pass location and map to the SearchBar component -->
      <SearchBar bind:location {placesLibrary} {map} initialValue={defaultPlace.name} />
      {/if}

      <div class="p-4 surface-variant outline-text rounded-lg space-y-3">
        <!-- Button to go to the form page -->
	<button onclick="window.location.href = 'https://helios-ss.com/form2.html';" class="bg-white-750 hover:bg-white-700 text-black font-bold py-2 px-4 rounded">Go to Form</button>

  	<!-- Button to go home -->
  	<button onclick="window.location.href = 'https://helios-ss.com';" class="bg-white-750 hover:bg-white-700 text-black font-bold py-2 px-4 rounded">Go Home</button>
      </div>

      {#if location}
      <Sections location={location} map={map} geometryLibrary={geometryLibrary} googleMapsApiKey={googleMapsApiKey} />
      {/if}

      <div class="grow" />

      <div class="flex flex-col items-center w-full">
      </div>

      <span class="pb-4 text-center outline-text label-small">
        This is not an officially supported Google product.
      </span>
    </div>
  </aside>
</div>
