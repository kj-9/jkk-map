<script>
  import RangeControl from './RangeControl.svelte';
  import MarkerPopup from './MarkerPopup.svelte';
  import { iconSVG } from './Icon.js';

  import railroad from './data/railroad.json';
  import station from './data/station.json';
  import residence from './data/residence.json';

  import { loading } from '$lib/stores/loading';

  // style sheet for leaflet
  import 'leaflet/dist/leaflet.css';

  // settings
  const viewpoint = [35.689655593827986, 139.70060999030161]; //新宿駅
  const paleGreen = '#ccebc5'; //schemeSet3[10]
  const orange = '#fb8072';
  const purple = '#bc80bd'; //schemeSet3[9]
  const baseStyle = {
    color: paleGreen,
    fillColor: paleGreen,
    weight: 1
  };

  const currentYear = new Date().getFullYear();

  // set loading true
  loading.set(true);
  // promise to dynamically import leaflet
  const L = import('leaflet');

  // parameter
  let rent = [80000, 120000];
  let rent_before = [];
  let year = [0, 40];
  let year_before = [];

  const buildMap = (node, { L }) => {
    loading.set(true);

    // helpers
    const createPopup = (props) => {
      const container = L.DomUtil.create('div');
      new MarkerPopup({
        target: container,
        props
      });
      return container;
    };

    const tileWhite = L.tileLayer('https://cyberjapandata.gsi.go.jp/xyz/blank/{z}/{x}/{y}.png', {
      attribution:
        '&copy; <a href=https://www.gsi.go.jp/kikakuchousei/kikakuchousei40182.html>GSI</a>'
    });

    const tileOsm = L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap'
    });

    let map = L.map(node, {
      layers: [tileWhite, tileOsm],
      scrollWheelZoom: false,
      maxZoom: 14,
      minZoom: 10,
      zoomControl: false
    }).setView(viewpoint, 11);

    L.control.zoom({ position: 'topright' }).addTo(map);

    L.control
      .layers({
        OpenStreetMap: tileOsm,
        WhiteMap: tileWhite
      })
      .addTo(map);

    // mapping railroad
    L.geoJson(railroad, {
      style: () => ({ ...baseStyle, smoothFactor: 1 })
    }).addTo(map);

    // mapping stations
    const stationOpts = {
      ...baseStyle,
      radius: 4,
      fillOpacity: 0.5,
      className: 'marker-station'
    };
    const stationOptsClicked = {
      ...stationOpts,
      radius: 8,
      fillColor: orange,
      fillOpacity: 1,
      className: 'marker-station-clicked'
    };

    console.log('Updating stations in map...');
    L.geoJson(station, {
      pointToLayer: (geoJsonPoint, latlng) => {
        const { N05_011, N05_003, N05_002 } = geoJsonPoint.properties;

        return L.circleMarker(latlng, stationOpts).bindPopup(() =>
          createPopup({ title: N05_011, paragraphs: [`${N05_003}: ${N05_002}`] })
        );
      },
      onEachFeature: (feature, layer) => {
        layer.on('click', ({ target }) => {
          target.options.className == 'marker-station'
            ? target.setStyle(stationOptsClicked)
            : target.setStyle(stationOpts);
        });
      }
    }).addTo(map);

    // add residence
    const residenceFG = L.featureGroup();
    residenceFG.addTo(map);

    let html = `<div class="map-marker"><div>${iconSVG(purple)}</div>`;
    const icon = L.divIcon({
      html,
      className: 'map-marker'
    });

    const layer = L.geoJson(residence, {
      pointToLayer: (geoJsonPoint, latlng) => {
        const {
          link,
          name,
          rent_from,
          rent_to,
          management_fee_raw,
          layouts,
          built_year,
          access,
          address
        } = geoJsonPoint.properties;
        return L.marker(latlng, { icon }).bindPopup(() =>
          createPopup({
            title: name,
            titleLink: `https://as.chizumaru.com${link}`,
            paragraphs: [
              `家賃: ${rent_from.toLocaleString()}~${rent_to.toLocaleString()}円 管理費: ${management_fee_raw}`,
              `間取り: ${layouts.replaceAll(/'|\[|\]/g, '')}`,
              `築年数: ${currentYear - built_year}年`,
              `${access}`,
              `${address}`
            ]
          })
        );
      }
    });

    residenceFG.addLayer(layer);

    const updateResidenceFG = () => {
      // check if parameters are updated
      if (rent === rent_before && year === year_before) {
        console.log('parameters are not changed.');
        return;
      } else {
        rent_before = rent;
        year_before = year;
      }

      layer.eachLayer((el) => {
        console.log('Updating residence in map...');
        const { rent_from, rent_to, built_year } = el.feature.properties;
        if (
          rent_from <= rent[1] &&
          rent_to >= rent[0] &&
          currentYear - built_year >= year[0] &&
          currentYear - built_year <= year[1]
        ) {
          el.setOpacity(1);
        } else {
          el.setOpacity(0);
        }
      });
    };

    updateResidenceFG();

    loading.set(false);

    return {
      update: updateResidenceFG,
      destroy() {
        // the node has been removed from the DOM
        console.log('Unloading Leaflet map.');
        map.remove();
        map = null;
      }
    };
  };
</script>

<svelte:head>
  <title>JKK Map</title>
  <meta name="description" content="JKK Map" />
</svelte:head>

{#await L then L}
  <div class="control">
    <RangeControl bind:rent bind:year />
  </div>
  <div class="map" use:buildMap={{ L: L, rent: rent, year: year }} />
{/await}

<style>
  .control {
    position: absolute;
    top: 10px;
    left: 10px;
    z-index: 1000;
  }
  .map {
    height: 100vh;
    width: 100vw;
  }

  :global(.map-marker) {
    transform: translateX(-70%) translateY(10%);
    transition: opacity 0.3s 0.3s ease;
  }
</style>
