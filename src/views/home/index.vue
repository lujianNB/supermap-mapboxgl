<template>
  <div class="lj-home" id="map">
    <div class="btn-one">
      <div class="btn-item" @click="setWMTS('shiliang')">矢量底图</div>
      <div class="btn-item" @click="setWMTS('chushi')">超图底图</div>
      <div class="btn-item" @click="setWMTS('yingxiang')">影像底图</div>
      <div class="btn-item" @click="renderPoint">画点</div>
      <div class="btn-item" @click="renderLine">画线</div>
      <div class="btn-item" @click="renderFill">画面</div>
      <div class="btn-item" @click="removePoint">清除点</div>
      <div class="btn-item" @click="removeLine">清除线</div>
      <div class="btn-item" @click="removeFill">清除面</div>
    </div>
    <div class="btn-two">
      <div class="btn-item" @click="addControl">添加控件</div>
      <div class="btn-item" @click="baseDraw">基本绘制</div>
      <div class="btn-item" @click="addQuery">查询图层</div>
      <div class="btn-item" @click="removeQuery">删除查询</div>
      <div class="btn-item" @click="filterLayer">筛选过滤</div>
      <div class="btn-item" @click="heat">热力图</div>
    </div>
  </div>
</template>

<script>
import { MeasureParameters, Unit, MeasureService, GetFeaturesByGeometryParameters, FeatureService, HeatMapLayer } from '@supermap/iclient-mapboxgl'
import MapboxDraw from '@mapbox/mapbox-gl-draw'
import '@/assets/css/mapbox-gl-draw.css'
import * as turf from '@turf/turf'
export default {
  name: 'ljHome',

  data () {
    return {
      idList: []
    }
  },

  computed: {},

  beforeDestroy () { },

  mounted () {
    this.initMap()
  },

  methods: {
    // 初始化地图
    initMap () {
      this.map = new this.$mapboxgl.Map({
        container: 'map',
        style: {
          "version": 8,
          "sources": {},
          "layers": []
        },
        center: [120.12, 30.16], // 地图初始中心点
        zoom: 10 // 地图初始缩放级别
      })
      this.map.on('load', () => {
        this.setWMTS('chushi')
      })
    },

    // 切换底图
    setWMTS (str) {
      if (this.map.getLayer('chushi-layer')) {
        this.map.removeLayer('chushi-layer')
        this.idList = this.idList.filter(item => {
          return item !== 'chushi-layer'
        })
      }
      this.map.getSource('chushi-source') && this.map.removeSource('chushi-source')
      if (this.map.getLayer('shiliang-layer')) {
        this.map.removeLayer('shiliang-layer')
        this.idList = this.idList.filter(item => {
          return item !== 'shiliang-layer'
        })
      }
      this.map.getSource('shiliang-source') && this.map.removeSource('shiliang-source')
      if (this.map.getLayer('yingxiang-layer')) {
        this.map.removeLayer('yingxiang-layer')
        this.idList = this.idList.filter(item => {
          return item !== 'yingxiang-layer'
        })
      }
      this.map.getSource('yingxiang-source') && this.map.removeSource('yingxiang-source')
      switch (str) {
        case 'chushi':
          this.loadChushi()
          break;
        case 'shiliang':
          this.loadShiliang()
          break;
        case 'yingxiang':
          this.loadYingxiang()
          break;

        default:
          break;
      }
    },
    loadChushi () {
      let baseUrl = "https://iserver.supermap.io/iserver/services/map-world/rest/maps/World/zxyTileImage.png?z={z}&x={x}&y={y}"
      this.map.addSource('chushi-source', {
        "type": "raster",
        "tiles": [baseUrl],
        "tileSize": 256
      })
      const wmtsLayer = {
        "id": "chushi-layer",
        "type": "raster",
        "source": "chushi-source"
      }
      this.map.addLayer(wmtsLayer)
      this.map.setZoom(2)
      this.idList = ['chushi-layer', ...this.idList]
      this.changeZoom()
    },
    loadShiliang () {
      this.map.addSource('shiliang-source', {
        type: 'raster',
        tiles: [
          'http://t0.tianditu.gov.cn/vec_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=c96c04581d6626f3fa38b22fa8ebf860'
        ],
        style: {
          layout: { visibility: 'visible' }
        },
        tileSize: 256
      })
      const wmtsLayer = {
        id: "shiliang-layer",
        type: 'raster',
        source: 'shiliang-source',
        minZoom: 7,
        maxZoom: 17,
        queryable: true,
        style: {
          layout: { visibility: 'visible' }
        },
        zoom: 17,
        paint: {
          'raster-opacity': 1
        }
      }
      this.map.addLayer(wmtsLayer)
      this.map.setZoom(10)
      this.idList = ['shiliang-layer', ...this.idList]
      this.changeZoom()
    },
    loadYingxiang () {
      this.map.addSource('yingxiang-source', {
        type: 'raster',
        tiles: [
          'http://t0.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=c96c04581d6626f3fa38b22fa8ebf860'
        ],
        style: {
          layout: { visibility: 'visible' }
        },
        tileSize: 256
      })
      const wmtsLayer = {
        id: "yingxiang-layer",
        type: 'raster',
        source: 'yingxiang-source',
        minZoom: 7,
        maxZoom: 17,
        queryable: true,
        style: {
          layout: { visibility: 'visible' }
        },
        zoom: 17,
        paint: {
          'raster-opacity': 1
        }
      }
      this.map.addLayer(wmtsLayer)
      this.map.setZoom(10)
      this.idList = ['yingxiang-layer', ...this.idList]
      this.changeZoom()
    },

    // 画点
    renderPoint () {
      this.removePoint()
      let geojson = {
        "type": "FeatureCollection",
        "features": [{
          "type": "Feature",
          "properties": {
            "name": "点一",
            "id": 1
          },
          "geometry": {
            "type": "Point",
            "coordinates": [120.12, 30.16]
          }
        },
        {
          "type": "Feature",
          "properties": {
            "name": "点二",
            "id": 2
          },
          "geometry": {
            "type": "Point",
            "coordinates": [120.2, 30.2]
          }
        }]
      }
      if (!this.map.hasImage('point-image')) {
        this.map.loadImage(
          require(`@/assets/chengshiguidao.png`),
          (error, image) => {
            if (error) throw error
            this.resizeImageBitmap(image, 22, 22).then((resizedBitmap) => {
              this.map.addImage('point-image', resizedBitmap)
            })
          }
        )
      }
      this.map.addSource('point-source', {
        'type': 'geojson',
        'data': geojson
      })
      this.map.addLayer({
        'id': 'point-layer',
        'type': 'symbol',
        'source': "point-source",
        'layout': {
          'visibility': 'visible',
          "icon-image": "point-image"
        },
        'paint': {}
      })
      this.idList = [...this.idList, 'point-layer']
      this.changeZoom()
      this.map.on('mouseenter', 'point-layer', () => {
        this.map.getCanvas().style.cursor = 'pointer' // 设置鼠标样式
      })
      this.map.on('mouseleave', 'point-layer', () => {
        this.map.getCanvas().style.cursor = ''
      })
      this.map.on('click', 'point-layer', (e) => {
        this.clickFn(e, 'point-layer')
      })
    },

    // 画线
    renderLine () {
      this.removeLine()
      let geojson = {
        "type": "FeatureCollection",
        "features": [
          {
            "id": 1,
            "type": "Feature",
            "properties": {
              "id": 1,
              "name": "line-1"
            },
            "geometry": {
              "type": "LineString",
              "coordinates": [
                [120.00, 30.00],
                [120.20, 30.32]
              ]
            }
          },
          {
            "id": 2,
            "type": "Feature",
            "properties": {
              "id": 2,
              "name": "line-2"
            },
            "geometry": {
              "type": "LineString",
              "coordinates": [
                [120.15, 30.10],
                [120.30, 30.12]
              ]
            }
          },
          {
            "id": 3,
            "type": "Feature",
            "properties": {
              "id": 3,
              "name": "line-3"
            },
            "geometry": {
              "type": "LineString",
              "coordinates": [
                [120.05, 30.00],
                [120.10, 30.20]
              ]
            }
          }
        ]
      }
      this.map.addSource('line-source', {
        'type': 'geojson',
        'data': geojson
      })
      this.map.addLayer({
        "layout": {
          "visibility": "visible"
        },
        "maxzoom": 24,
        "paint": {
          "line-width": 8,
          // "line-color": ['match', ['get', 'name'],
          //   'line-1', '#f61c13',
          //   'line-2', '#efa573',
          //   'line-3', '#f9a81a',
          //   'blue'
          // ]
          "line-color": 'red'
          // "line-color": [
          //   'case',
          //   ['boolean', ['feature-state', 'click'], false],
          //   'blue',
          //   'red'
          // ]
        },
        "id": "line-layer",
        "source": "line-source",
        "type": "line",
        "minzoom": 0
      })
      this.idList = [...this.idList, 'line-layer']
      this.changeZoom()
      this.map.on('mouseenter', 'line-layer', () => {
        this.map.getCanvas().style.cursor = 'pointer' // 设置鼠标样式
      })
      this.map.on('mouseleave', 'line-layer', () => {
        this.map.getCanvas().style.cursor = ''
      })
      this.map.on('click', 'line-layer', (e) => {
        this.clickFn(e, 'line-layer')
      })
    },

    // 画面
    renderFill () {
      this.removeFill()
      let geojson = {
        "type": "FeatureCollection",
        "features": [
          {
            "id": 1,
            "type": "Feature",
            "properties": {
              "id": 1,
              "name": "fill-1"
            },
            "geometry": {
              "type": "Polygon",
              "coordinates": [[[119.96681121640789, 30.318485110607], [120.08594421201235, 30.325694888439198], [120.07461456113357, 30.20141205152499], [119.92767242246117, 30.213085025157724]]]
            }
          }
        ]
      }
      this.map.addSource('fill-source', {
        'type': 'geojson',
        'data': geojson
      })
      this.map.addLayer({
        'id': 'fill-layer',
        'type': 'fill',
        'source': 'fill-source',
        'layout': {},
        'paint': {
          'fill-color': '#088',
          'fill-opacity': 0.8
        }
      })
      this.idList = [...this.idList, 'fill-layer']
      this.changeZoom()
      this.map.on('mouseenter', 'fill-layer', () => {
        this.map.getCanvas().style.cursor = 'pointer' // 设置鼠标样式
      })
      this.map.on('mouseleave', 'fill-layer', () => {
        this.map.getCanvas().style.cursor = ''
      })
      this.map.on('click', 'fill-layer', (e) => {
        this.clickFn(e, 'fill-layer')
      })
    },

    // 删除点
    removePoint () {
      if (this.map.getLayer('point-layer')) {
        this.map.removeLayer('point-layer')
        this.idList = this.idList.filter(item => {
          return item !== 'point-layer'
        })
      }
      this.map.getSource('point-source') && this.map.removeSource('point-source')
    },

    // 删除线
    removeLine () {
      if (this.map.getLayer('line-layer')) {
        this.map.removeLayer('line-layer')
        this.idList = this.idList.filter(item => {
          return item !== 'line-layer'
        })
      }
      this.map.getSource('line-source') && this.map.removeSource('line-source')
    },

    // 删除面
    removeFill () {
      if (this.map.getLayer('fill-layer')) {
        this.map.removeLayer('fill-layer')
        this.idList = this.idList.filter(item => {
          return item !== 'fill-layer'
        })
      }
      this.map.getSource('fill-source') && this.map.removeSource('fill-source')
    },

    // 点击事件
    clickFn (e, name) {
      console.log('e: ', e)
      let bbox = [
        [e.point.x - 5, e.point.y - 5],
        [e.point.x + 5, e.point.y + 5]
      ]
      let features = this.map.queryRenderedFeatures(bbox, {
        layers: [name]
      })
      console.log('features: ', features)
      // 创建信息窗体
      let coordinates = e.lngLat
      let description = features[0].properties.id + "<br/>" + features[0].properties.name
      let popup = new this.$mapboxgl.Popup()
        .setLngLat(coordinates)
        .setHTML(description)
        .addTo(this.map)
      popup.on('close', () => {
        console.log('Popup is closed')
        // 在这里编写你需要执行的代码
      })
      // this.map.setPaintProperty('图层id', 'key', 'value')
      // this.map.setLayoutProperty('图层id', 'key', 'value')
      // if (name === 'line-layer') {
      // this.map.setPaintProperty('line-layer', 'line-color', 'blue')
      // this.map.setFeatureState(
      //   {
      //     source: 'line-source',
      //     id: features[0].properties.id
      //   },
      //   { click: true }
      // )
      // }

      // 定位
      this.map.flyTo({ center: coordinates })
    },

    // 图层层级位置
    changeZoom () {
      this.idList.forEach((item, i) => {
        if (i !== 0) {
          this.map.moveLayer(item)
        }
      })
    },

    // 处理图片
    resizeImageBitmap (bitmap, newWidth, newHeight) {
      const canvas = new OffscreenCanvas(newWidth, newHeight);
      const ctx = canvas.getContext('2d')

      // 将ImageBitmap绘制到canvas上，并调整大小
      ctx.drawImage(bitmap, 0, 0, newWidth, newHeight)

      // 创建一个新的ImageBitmap
      return createImageBitmap(canvas)
    },

    // 添加控件
    addControl () {
      if (!this.controlFlag) {
        this.controlFlag = true
        this.navigationControl = new this.$mapboxgl.NavigationControl()
        this.scaleControl = new this.$mapboxgl.ScaleControl()
        this.fullscreenControl = new this.$mapboxgl.FullscreenControl()
        this.map.addControl(this.navigationControl)
        this.map.addControl(this.scaleControl)
        this.map.addControl(this.fullscreenControl)
      } else {
        this.controlFlag = false
        this.navigationControl && this.map.removeControl(this.navigationControl)
        this.scaleControl && this.map.removeControl(this.scaleControl)
        this.fullscreenControl && this.map.removeControl(this.fullscreenControl)
      }
    },

    // 基本绘制
    baseDraw () {
      if (!this.drawControlFlag) {
        this.drawControlFlag = true
        this.draw = new MapboxDraw()
        this.map.addControl(this.draw, 'top-left')
        this.map.on('draw.create', this.drawEvent)
        this.map.on('draw.update', this.drawEvent)
      } else {
        this.drawControlFlag = false
        this.draw && this.map.removeControl(this.draw)
        this.map.off('draw.create', this.drawEvent)
        this.map.off('draw.update', this.drawEvent)
      }
    },
    drawEvent (e) {
      console.log('e: ', e)
      const { coordinates, type } = e.features[0].geometry
      if (type === 'Point') {
        console.log('coordinates: ', coordinates)
      }
      if (type === 'LineString') {
        let line = turf.lineString(coordinates)
        let len = turf.length(line)
        console.log('turf计算长度', len + 'km')
        // 设置量算服务参数
        let measureParam = new MeasureParameters()
        // 设置要量算的矢量对象({Line}或{Polygon})，geometry可以通过直接初始化的方法获取
        measureParam.geometry = e.features[0].geometry
        measureParam.unit = Unit.METER
        let url = "https://iserver.supermap.io/iserver/services/map-world/rest/maps/World";
        // 初始化服务类，设置服务请求关键参数
        let measureService = new MeasureService(url)
        // 提交服务请求，传递服务查询参数，获取返回结果并按照用户需求进行处理
        measureService.measureDistance(measureParam).then(function (serviceResult) {
          // 获取服务器返回的结果
          let { result } = serviceResult
          console.log('超图返回长度结果', result)
        })
      }
      if (type === 'Polygon') {
        let polygon = turf.polygon(coordinates)
        let area = turf.area(polygon)
        console.log('turf计算面积', area + '平方米')
        let areaMeasureParam = new MeasureParameters()
        areaMeasureParam.geometry = e.features[0].geometry
        let urlOne = "https://iserver.supermap.io/iserver/services/map-world/rest/maps/World";
        // 提交服务请求，传递服务查询参数，获取返回结果并按照用户需求进行处理
        new MeasureService(urlOne).measureArea(areaMeasureParam).then(function (serviceResult) {
          // 获取服务器返回的结果
          let { result } = serviceResult
          console.log('超图返回面积结果', result)
        })

        // 设置几何查询范围
        let urlTwo = "https://iserver.supermap.io/iserver/services/data-world/rest/data"
        let queryPolygonGeometry = e.features[0].geometry
        // 设置任意几何范围查询参数
        let geometryParam = new GetFeaturesByGeometryParameters({
          datasetNames: ["World:Capitals"],
          geometry: queryPolygonGeometry,
          spatialQueryMode: "INTERSECT" // 相交空间查询模式
        })
        // 创建任意几何范围查询实例
        new FeatureService(urlTwo).getFeaturesByGeometry(geometryParam).then(function (serviceResult) {
          // 获取服务器返回的结果
          let { features } = serviceResult.result
          console.log('几何查询范围结果', features)
          window.localStorage.setItem('queryFeatures', JSON.stringify(features))
        })
      }
    },

    // 绘制查询图层
    addQuery () {
      let features = window.localStorage.getItem('queryFeatures')
      if (!features) return
      this.map.addSource("queryDatas", {
        "type": "geojson",
        "data": JSON.parse(features)
      })
      this.map.addLayer({
        "id": "queryDatas",
        "type": "circle",
        "source": "queryDatas",
        "paint": {
          "circle-radius": 6, /* 圆的直径，单位像素 */
          "circle-color": "blue", /* 圆的颜色 */
          "circle-opacity": 0.5  /* 圆的颜色 */
        }
      })
      this.idList = [...this.idList, 'queryDatas']
      this.changeZoom()
    },

    // 删除查询图层
    removeQuery () {
      if (this.map.getLayer('queryDatas')) {
        this.map.removeLayer('queryDatas')
        this.idList = this.idList.filter(item => {
          return item !== 'queryDatas'
        })
      }
      this.map.getSource('queryDatas') && this.map.removeSource('queryDatas')
      window.localStorage.setItem('queryFeatures', null)
    },

    // 筛选过滤
    filterLayer () {
      if (this.map.getLayer('line-layer')) {
        if (!this.filterLayerFlag) {
          this.filterLayerFlag = true
          this.map.setFilter('line-layer', [
            "any",
            [
              "==",
              [
                "get",
                "name"
              ],
              'line-1'
            ],
            [
              "==",
              [
                "get",
                "name"
              ],
              'line-2'
            ]
          ])
        } else {
          this.filterLayerFlag = false
          this.map.setFilter('line-layer', null)
        }
      } else {
        alert('请先画线，以线图层为demo')
      }
    },

    // 热力图
    heat () {
      if (!this.heatFlag) {
        this.heatFlag = true
        let heatMapLayer = new HeatMapLayer(
          "heatMap",
          {
            "map": this.map,
            "id": "heatmap",
            "radius": 45,
            //featureWeight指定以哪个属性值为热力权重值创建热力图
            "featureWeight": "value",
          }
        )
        let num = 200
        let radius = 50
        heatMapLayer.useGeoUnit = true
        heatMapLayer.radius = radius

        let features = [];
        for (let i = 0; i < num; i++) {
          features[i] =
          {
            "type": "feature",
            "geometry": {
              "type": "Point",
              "coordinates": [
                Math.random() * 360 - 180,
                Math.random() * 160 - 80]
            },
            "properties": {
              "value": Math.random() * 9,
            }
          }
        }

        let heatPoints = {
          "type": "FeatureCollection",
          "features": features
        }

        heatMapLayer.addFeatures(heatPoints)
        this.map.addLayer(heatMapLayer)
      } else {
        this.heatFlag = false
        this.map.getLayer('heatmap') && this.map.removeLayer('heatmap')
      }

    }
  }
}
</script>

<style lang='scss' scoped>
.lj-home {
  width: 100%;
  height: 100%;
  position: relative;
  .btn-one,
  .btn-two {
    position: absolute;
    right: 10px;
    bottom: 10px;
    width: 300px;
    height: 150px;
    background-color: #fff;
    border: 1px solid #ddd;
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
    align-content: flex-start;
    padding: 15px;
    z-index: 10;
    .btn-item {
      width: 80px;
      height: 30px;
      border: 1px solid #ddd;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 13px;
      cursor: pointer;
      &:nth-of-type(3n + 2) {
        margin: 0 14px;
      }
      &:nth-last-of-type(1),
      &:nth-last-of-type(2),
      &:nth-last-of-type(3) {
        margin-bottom: 0;
      }
      &:hover {
        color: #4178d6;
      }
      &:active {
        font-size: 16px;
      }
    }
  }
  .btn-one {
    bottom: 170px;
  }
}
</style>
