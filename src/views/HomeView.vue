<template>
  <div class="home">
    <div id="CesiumContainer"></div>
  </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import 'cesium/Source/Widgets/widgets.css'
import { onMounted } from 'vue'

const img = require('../assets/nongyerongdan.png')
// const model = require('../assets/Monster.gltf')
let viewer = null

onMounted(() => {
  const option = {
    // imageryProvider : new Cesium.UrlTemplateImageryProvider({
    //     url : 'https://webrd0{s}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}',
    //     subdomains: ['1', '2', '3', '4'],
    //     tilingScheme: new Cesium.WebMercatorTilingScheme(),
    //     maximumLevel: 18
    // }),
    // 我使用高德影像地形地图
    // imageryProvider: new Cesium.UrlTemplateImageryProvider({
    //   url: "https://webst02.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}",
    // }),
    // baseLayerPicker: false,
    // infoBox: false  // 禁用默认提示框
    infoBox: false,
    selectionIndicator: false,
    shadows: true,
    shouldAnimate: true,
  }
  initMap(option)

  viewer.scene.moon.show = false // 隐藏月亮

  // 调试工具
  // viewer.extend(Cesium.viewerCesiumInspectorMixin);


  var beijingPosition = Cesium.Cartesian3.fromDegrees(116.397128, 39.916527, 10000.0); // 经度, 纬度, 高度（米）

  // 再加上高德影像注记地图
  // viewer.imageryLayers.addImageryProvider(
  //   new Cesium.UrlTemplateImageryProvider({
  //     url: "http://webst02.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scale=1&style=8",
  //   })
  // );

  // viewer.camera.flyTo({
  //   destination: beijingPosition,
  //   dutation: 2
  // })

  const boundingSphere = new Cesium.BoundingSphere(
    beijingPosition,
    // 15000
  );
  // 定位到初始位置
  viewer.camera.flyToBoundingSphere(boundingSphere, {
    // 动画，定位到初始位置的过渡时间，设置成0，就没有动画
    duration: 0,
  });
  viewer._cesiumWidget._creditContainer.style.display = "none"; // 隐藏版权
  console.log(viewer)

  // addPoint()
  // addMultiPoint()
  // addCustomizePoint()
  addModelByEntity()
  // addModelByCZML()
})

function initMap (option) {
  viewer = new Cesium.Viewer('CesiumContainer', Object.assign({}, {
    geocoder: false, // 搜索按钮
    baseLayerPicker: false, // 地图风格切换按钮
    homeButton: false, // 恢复默认视角按钮
    sceneModePicker: false, // 2d,2.5d,3d切换按钮
    navigationHelpButton: false, // 操作指引按钮
    animation: false,// 动画播放控件
    timeline: false // 时间轴控件
  }, option))
}

// 添加点
function addPoint() {
  var position = Cesium.Cartesian3.fromDegrees(116.391, 39.907)
  var pointEntity = viewer.entities.add({
    position: position,
    point: {
      pixelSize: 10, // 点的大小
      color: Cesium.Color.RED, // 点的颜色
      outlineColor: Cesium.Color.WHITE, // 点的边框颜色
      outlineWidth: 2 // 点的边框宽度
    },
    label: {
      text: '天安门',
      font: '24pt monospace',
      style: Cesium.LabelStyle.FILL_AND_OUTLINE,
      outlineWidth: 2,
      verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
      pixelOffset: new Cesium.Cartesian2(0, -15)
    },
    // description: '<h1>天安门</h1><p>天安门位于北京市中心，是中国的象征之一。</p>'
  })

  viewer.flyTo(pointEntity)

  // 创建自定义的弹出信息框
  var customInfoBox = document.createElement('div');
  customInfoBox.id = 'customInfoBox';
  customInfoBox.style.position = 'absolute';
  customInfoBox.style.backgroundColor = 'rgba(255, 255, 255, 0.9)';
  customInfoBox.style.padding = '10px';
  customInfoBox.style.border = '1px solid #ccc';
  customInfoBox.style.borderRadius = '5px';
  customInfoBox.style.display = 'none'; // 初始隐藏
  document.body.appendChild(customInfoBox);

  viewer.screenSpaceEventHandler.setInputAction(function (click) {
    var pickedObject = viewer.scene.pick(click.position)
    if (Cesium.defined(pickedObject) && pickedObject.id === pointEntity) {
      // viewer.selectedEntity = pointEntity

      const position = pickedObject.id.position.getValue(viewer.clock.currentTime)

      // 地理坐标（cartesian3）转换为屏幕坐标
      console.log(viewer.scene)
      console.log(Cesium.SceneTransforms)
      console.log(Cesium.SceneTransforms.wgs84ToWindowCoordinates)
      // var screenPosition = Cesium.SceneTransforms.wgs84ToWindowCoordinates(viewer.scene, position);
      var screenPosition = Cesium.SceneTransforms.worldToWindowCoordinates(viewer.scene, position);

      // 设置自定义信息框内容
      customInfoBox.innerHTML = `
        <h3>天安门</h3>
        <p>这是一个自定义的信息框。</p>
      `;

      // 显示自定义信息框
      customInfoBox.style.display = 'block';

      // 将信息框显示在点击点位的上方
      customInfoBox.style.left = screenPosition.x - customInfoBox.offsetWidth / 2 + 'px';  // 使信息框水平居中
      customInfoBox.style.top = screenPosition.y - customInfoBox.offsetHeight - 15 + 'px';  // 显示在点的上方，偏移 15px
    } else {
      customInfoBox.style.display = 'none';
    }
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK)

  // 右键点击隐藏信息框
  // viewer.screenSpaceEventHandler.setInputAction(function() {
  //     customInfoBox.style.display = 'none'; // 隐藏信息框
  // }, Cesium.ScreenSpaceEventType.RIGHT_CLICK);
}

// 添加多个点
function addMultiPoint() {
  var positions = [
    { name: "北京", longitude: 116.391, latitude: 39.907 },
    { name: "上海", longitude: 121.473, latitude: 31.230 },
    { name: "广州", longitude: 113.264, latitude: 23.129 }
  ]

  positions.forEach(function(position) {
    viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(position.longitude, position.latitude),
      point: {
        pixelSize: 10,
        color: Cesium.Color.BLUE,
        outlineColor: Cesium.Color.WHITE,
        outerWidth: 2
      },
      label: {
        text: position.name,
        font: '24pt monospace',
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        outlineWidth: 2,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
        pixelOffset: new Cesium.Cartesian2(0, -15)
      }
    })
  })
  viewer.zoomTo(viewer.entities)
}

// 添加自定义点
function addCustomizePoint() {
  var position = Cesium.Cartesian3.fromDegrees(116.391, 39.907) // 天安门
  
  viewer.entities.add({
    position: position,
    billboard: {
      image: img,
      width: 32,
      height: 32
    },
    label: {
      text: '天安门',
      font: '14pt monospace',
      style: Cesium.LabelStyle.FILL_AND_OUTLINE,
      outlineWidth: 2,
      verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
      pixelOffset: new Cesium.Cartesian2(0, -35)
    }
  })
}

// 添加模型 entity方式
function addModelByEntity() {
  // viewer.scene.primitives.add(Cesium.Model.fromGltf({
  //   url: 'https://s3.amazonaws.com/cesiumjs.org/Cesium_Logo_Color.gltf',
  //   modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(position),
  //   minimumPixelSize: 128
  // }))
  viewer.entities.removeAll();
  const position = Cesium.Cartesian3.fromDegrees(
    116.391, 
    39.907,
    10.0,
  );
  const heading = Cesium.Math.toRadians(135);
  const pitch = 0;
  const roll = 0;
  const hpr = new Cesium.HeadingPitchRoll(heading, pitch, roll);
  const orientation = Cesium.Transforms.headingPitchRollQuaternion(position, hpr);
  var modelEntity = viewer.entities.add({
    name: 'Cesium Air',
    model: {
      // uri: '/Monster.gltf',
      // uri: '/Buggy.gltf',
      uri: '/Cesium_Air.glb',
      // uri: 'https://s3.amazonaws.com/cesiumjs.org/Cesium_Logo_Color.gltf',
      minimumPixelSize: 3,
      maximumScale: 20,
      show: true,
    },
    position: position,
    orientation: orientation
  })
  viewer.trackedEntity = modelEntity;
  // viewer.flyTo(modelEntity)
}

function addModelByCZML() {
  const czml = [
    {
      id: 'document',
      name: 'CZML Model',
      version: '1.0'
    }, 
    {
      id: 'airplane_model',
      name: 'airplane',
      position: {
        cartographicDegrees: [116.391, 39.907, 1000],  // 使用标准的 CZML cartographicDegrees 经度, 纬度, 高度
      },
      model: {
        gltf: '/Soldier.glb',     // 模型的 glTF 路径
        scale: 12.0,            // 模型的缩放比例
        minimumPixelSize: 64,    // 模型的最小像素大小
      }
  }];
  
  const dataSourcePromise = viewer.dataSources.add(Cesium.CzmlDataSource.load(czml))

  // dataSourcePromise.then(dataSource => {
  //   viewer.trackedEntity = dataSource.entities.getById('airplane_model')
  // }).catch(function(err) {
  //   console.log(err)
  // })

  dataSourcePromise.then(dataSource => {
    viewer.trackedEntity = dataSource.entities.getById('airplane_model');
    console.log(111)
  }).catch(function(err) {
    console.error("Error loading CZML:", err);
  });
  viewer.zoomTo(dataSourcePromise)
}

</script>

<style lang="less" scoped>
.home, #CesiumContainer {
  width: 100%;
  height: 100%;
}
</style>
