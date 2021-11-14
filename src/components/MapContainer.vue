<template>
  <div ref="map-root" style="width: 100%; height: 100vh">
  </div>
  <ui-dialog v-model="openDialog" @confirm="onConfirm">
    <ui-dialog-title>Ketentuan Tata Ruang: {{RDTRData.kode}} </ui-dialog-title>
    <ui-dialog-content>
      <ui-divider></ui-divider>
      <h4>Data RDTR</h4>
      <p>Pola Ruang: {{RDTRData.pola_ruang}}</p>
      <p>Zona: {{RDTRData.zona}}</p>
      <p>Sub-Zona: {{RDTRData.sub_zona}}</p>
      <h4>Penggunaan Persil</h4>
      <p>Luas Persil: {{PersilData.DESA}} m<sup>2</sup></p>
      <p>Penggunaan: {{PersilData.KECAMATAN}}</p>
      <p>Tipe Persil: {{PersilData.TIPE}}</p>
    </ui-dialog-content>
    <ui-grid>
      <ui-grid-cell></ui-grid-cell>
      <ui-grid-cell>
        <ui-fab extended>
          <template #before="{ iconClass }">
            <ui-icon :class="iconClass">search</ui-icon>
          </template>
          <span>Cek Izin</span>
        </ui-fab>
      </ui-grid-cell>
    </ui-grid>
    <ui-dialog-actions></ui-dialog-actions>
  </ui-dialog>
</template>

<script>
import View from 'ol/View'
import Map from 'ol/Map'
import TileLayer from 'ol/layer/Tile'
import XYZ from 'ol/source/XYZ'
import TileWMS from 'ol/source/TileWMS'

import LayerSwitcher from 'ol-layerswitcher'
import mixinKegiatan from '../mixins/dataKegiatan.js'
// import { BaseLayerOptions, GroupLayerOptions } from 'ol-layerswitcher';

export default {
  name: 'MapContainer',
  mixins: [mixinKegiatan],
  components: {},
  props: {
    geojson: Object
  },
  computed: {},
  data: () => ({
    openDialog: false,
    olMap: null,
    vectorLayer: null,
    selectedFeature: null,
    drag: false,
    RDTRData: {},
    PersilData: {},
    luasPersil: null
  }),
  mounted () {
    this.loadData()

    this.wmsSource = new TileWMS({
      // url: 'https://geoserver.jogjakota.go.id/geoserver/sitaru/wms',
      url: 'http://geoportal.ppids.ft.ugm.ac.id/geoserver/sitaru/wms',
      params: {
        LAYERS: 'sitaru:persil_kota_yogyakarta_wgs',
        FORMAT: 'image/png8',
        TILED: true,
        VERSION: '1.1.1'
      },
      serverType: 'geoserver',
      crossOrigin: 'anonymous'
    })

    this.jalanGSB = new TileLayer({
      title: 'Jaringan Jalan',
      visible: true,
      source: new TileWMS({
        url: 'https://geoserver.jogjakota.go.id/geoserver/sitaru/wms',
        params: {
          LAYERS: 'sitaru:jalan_gsb',
          TILED: true
        },
        serverType: 'geoserver',
        transition: 0
      })
    })

    this.rdtrtile = new TileLayer({
      title: 'Rencana Detil Tata Ruang',
      visible: true,
      opacity: 0.6,
      source: new TileWMS({
        url: 'http://geoportal.ppids.ft.ugm.ac.id/geoserver/sitaru/wms',
        params: {
          LAYERS: 'sitaru:rdtr_kota_yogyakarta_wgs',
          TILED: true
        },
        serverType: 'geoserver',
        transition: 0
      })
    })

    this.esriSatellite = new TileLayer({
      title: 'ESRI Satellite',
      visible: true,
      source: new XYZ({
        url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
        attributions: 'Tiles &copy; Esri &mdash; Source: Esri, i-cubed, USDA, USGS, AEX, GeoEye, Getmapping, Aerogrid, IGN, IGP, UPR-EGP, and the GIS User Community'
      })
    })

    this.view = new View({
      zoom: 16,
      center: [110.3650042, -7.8049497],
      projection: 'EPSG:4326'
    })

    this.olMap = new Map({
      target: this.$refs['map-root'],
      layers: [
        this.esriSatellite,
        this.rdtrtile,
        this.jalanGSB
      ],
      view: this.view
    })

    const layerSwitcher = new LayerSwitcher({
      reverse: true,
      groupSelectStyle: 'group'
    })
    this.olMap.addControl(layerSwitcher)

    this.olMap.on('singleclick', (evt) => {
      var viewResolution = /** @type {number} */ (this.view.getResolution())
      var url = this.wmsSource.getFeatureInfoUrl(
        evt.coordinate, viewResolution, 'EPSG:4326', {
          INFO_FORMAT: 'application/json',
          QUERY_LAYERS: 'sitaru:persil_kota_yogyakarta_wgs',
          FEATURE_COUNT: 5
        })
      if (url) {
        fetch(url)
          .then((response) => response.text())
          .then((html) => {
            // eslint-disable-next-line dot-notation
            console.log(JSON.parse(html))
            this.RDTRData = JSON.parse(html).features[0].properties
            this.PersilData = JSON.parse(html).features[1].properties
            this.luasPersil = this.PersilData.luas.toFixed(2)
          })
      }
      this.openDialog = true
    })
  },
  methods: {
    onConfirm (result) {
      if (result) {
        console.log('ok')
      } else {
        console.log('cancel')
      }
    },
    openDialogKegiatan () {
    }
  }
}

</script>
