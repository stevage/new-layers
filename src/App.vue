<template lang="pug">
    #app.flex.flex-column.vh-100.avenir
        #top.bb.b--gray.bg-washed-yellow
            //- h1 Community Map
        #middle.flex.flex-auto
            #sidebar.br.b--light-gray.w-25.overflow-auto
                FeatureInfo
            #map-container.relative.flex-auto
                Map
                #overlay.absolute
        #bottom.bt.b--light-gray.flex-none
</template>

<script>
import Map from './components/Map.vue'
import FeatureInfo from './components/FeatureInfo.vue'
window.app = {
}
export default {
    name: 'app',
    components: {
      Map,
      FeatureInfo
    },
    created() {
        window.app.App = this;
    }
}

require('tachyons/css/tachyons.min.css');
</script>

<style>
html, body {
  height: 100vh;
  width: 100%;
  margin:0;
  padding:0;
}
#sidebar {
    min-width:200px;
}

</style>
