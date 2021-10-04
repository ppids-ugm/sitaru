<template>
<div>
  <map-container></map-container>
</div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
// import SitaruMapContainer from '@/components/SitaruMap.vue'
import MapContainer from '@/components/MapContainer.vue'

export default {
  name: 'Home',
  components: {
    MapContainer
  }
}
</script>

<style scoped lang="scss">
.container {
  height: 100%;
}

</style>
