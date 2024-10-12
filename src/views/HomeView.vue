<template>
  <div class="home">
    <div id="CesiumContainer"></div>
  </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import 'cesium/Source/Widgets/widgets.css'

import { onMounted } from 'vue'

onMounted(() => {
  let viewer = new Cesium.Viewer('CesiumContainer')
  console.log(viewer)
})
</script>

<style lang="less" scoped>
.home, #CesiumContainer {
  width: 100%;
  height: 100%;
}
</style>
