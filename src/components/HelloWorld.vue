<template>
  <div id="mapfo">
    <div id="header">
      <div id="logoandtitle">
        GISpace动态标绘演示系统 V1.0.2（for OL3 & vue）
        <span id="aboutLabel" @click="showAbout()">关 于</span>
      </div>
      <div id="version">— powered by Plot API V1.0.2 for OL3 & vue</div>
    </div>
    <div id="menu">
      <el-row>
        点标
        <el-button class="el-row" size="small" type="primary" plain @click="activate('marker')">点</el-button>
        <!-- <button type="button" @click="activate('marker')">点</el-button> -->
      </el-row>
      <el-row>
        线标
        <el-button class="el-row" size="small" type="primary" plain @click="activate('arc')">弧线</el-button>
        <el-button class="el-row" size="small" type="primary" plain @click="activate('curve')">曲线</el-button>
        <el-button class="el-row" size="small" type="primary" plain @click="activate('polyline')">折线</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('freehandline')"
        >自由线</el-button>
      </el-row>
      <el-row>
        面标
        <el-button class="el-row" size="small" type="primary" plain @click="activate('circle')">圆</el-button>
        <el-button class="el-row" size="small" type="primary" plain @click="activate('ellipse')">椭圆</el-button>
        <el-button class="el-row" size="small" type="primary" plain @click="activate('lune')">弓形</el-button>
        <el-button class="el-row" size="small" type="primary" plain @click="activate('sector')">扇形</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('closedcurve')"
        >曲线面</el-button>
        <el-button class="el-row" size="small" type="primary" plain @click="activate('polygon')">多边形</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('rectangle')"
        >矩形</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('freehandpolygon')"
        >自由面</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('gatheringplace')"
        >聚集地</el-button>
      </el-row>
      <el-row>
        箭头
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('doublearrow')"
        >钳击</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('straightarrow')"
        >直箭头</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('finearrow')"
        >细直箭头</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('assaultdirection')"
        >突击方向</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('attackarrow')"
        >进攻方向</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('tailedattackarrow')"
        >进攻方向（尾）</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('squadcombat')"
        >分队战斗行动</el-button>
        <el-button
          class="el-row"
          size="small"
          type="primary"
          plain
          @click="activate('tailedsquadcombat')"
        >分队战斗行动（尾）</el-button>
      </el-row>
    </div>
    <div id="aboutContainer">
      <div id="aboutTitle">
        关于...
        <span id="closeBtn" @click="hideAbout()">X</span>
      </div>
      <div id="aboutContent">
        <p class="aboutText">
          plot4ol3是GISpace动态标绘的OpenLayers3版本，旨在为基于开源GIS技术的项目提供标绘API（
          <a
            href="http://7xlbfq.com1.z0.glb.clouddn.com/plot4flex/index.html"
          >ArcGIS Flex版本(链接已失效)</a>）。当前版本实现了常用符号的绘制，下一版本将实现符号编辑、序列化和反序列化。
          在此版本基础上实现本Demo在vue下的集成
        </p>
        <p class="aboutText">
          欢迎使用和反馈，开发者：@平凡的世界 （QQ：21587252） <br/>vue集成：小迷茫
          <img id="earth" src="../assets/earth.png" />（QQ:1399138561）  天空（QQ:1468707445)
        </p>
      </div>
    </div>
    <div id="delete-wrapper">
      <el-button
        class="el-row"
        size="small"
        type="primary"
        plain
        id="btn-delete"
        style="display:none;"
      >删 除</el-button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      map: {},
      plotDraw: null,
      plotEdit: null,
      drawOverlay: null,
      drawStyle: null,
      center: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      let _this = this;
      this.center = ol.proj.transform([116.32, 29.88], "EPSG:4326", "EPSG:3857");
      // 初始化地图，底图使用openstreetmap在线地图
      this.map = new ol.Map({
        target: "mapfo",
        layers: [
          new ol.layer.Tile({
            //source: new ol.source.MapQuest({layer: 'sat'})
            source: new ol.source.OSM(),
          }),
        ],
        view: new ol.View({
          center: this.center,
          zoom: 4,
        }),
      });
      this.map.on("click", function (e) {
        if (_this.plotDraw.isDrawing()) {
          return;
        }
        var feature = _this.map.forEachFeatureAtPixel(e.pixel, function (
          feature,
          layer
        ) {
          return feature;
        });
        if (feature) {
          // 开始编辑
          _this.plotEdit.activate(feature);
          _this.activeDelBtn();
        } else {
          // 结束编辑
          _this.plotEdit.deactivate();
          _this.deactiveDelBtn();
        }
      });
      this.plotDraw = new P.PlotDraw(this.map);
      this.plotDraw.on(
        P.Event.PlotDrawEvent.DRAW_END,
        this.onDrawEnd,
        false,
        this
      );
      this.plotEdit = new P.PlotEdit(this.map);
      // 设置标绘符号显示的默认样式
      var stroke = new ol.style.Stroke({
        color: "#FF0000",
        width: 2,
      });
      var fill = new ol.style.Fill({ color: "rgba(0,255,0,0.4)" });
      var image = new ol.style.Circle({
        fill: fill,
        stroke: stroke,
        radius: 8,
      });
      this.drawStyle = new ol.style.Style({
        image: image,
        fill: fill,
        stroke: stroke,
      });
      // 绘制好的标绘符号，添加到FeatureOverlay显示。
      this.drawOverlay = new ol.layer.Vector({
        source: new ol.source.Vector(),
      });
      this.drawOverlay.setStyle(this.drawStyle);
      this.drawOverlay.setMap(this.map);
      this.get("btn-delete").onclick = function () {
        if (_this.drawOverlay && _this.plotEdit && _this.plotEdit.activePlot) {
          _this.drawOverlay
            .getSource()
            .removeFeature(_this.plotEdit.activePlot);
          _this.plotEdit.deactivate();
          _this.deactiveDelBtn();
        }
      };
      // this.get("mapfo").onclick = function (e) {
      //   // alert('X ; '+ e.clientX  + 'Y: '+e.clientY);
      //   var t = _this.map.getEventCoordinate(e)
      //   alert(t);
      // };
      //绘制扇形实例
      var sector = new P.PlotFactory.createPlot(P.PlotTypes.SECTOR, [
        this.center,
        [11442317.386177745,3776600.693513987],
        [11853242.850238852,2817774.610704737],
      ]);
      var feature = new ol.Feature({
        geometry: sector,
      });
      this.drawOverlay.getSource().addFeature(feature);
    },
    onDrawEnd(event) {
      var feature = event.feature;
      this.drawOverlay.getSource().addFeature(feature);
      // 开始编辑
      this.plotEdit.activate(feature);
      this.activeDelBtn();
    },
    activate(type) {
      this.plotEdit.deactivate();
      this.plotDraw.activate(type);
    },
    get(domId) {
      return document.getElementById(domId);
    },
    showAbout() {
      document.getElementById("aboutContainer").style.visibility = "visible";
    },
    hideAbout() {
      document.getElementById("aboutContainer").style.visibility = "hidden";
    },
    activeDelBtn() {
      document.getElementById("btn-delete").style.display = "inline-block";
    },
    deactiveDelBtn() {
      document.getElementById("btn-delete").style.display = "none";
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a:hover {
  color: #f00;
}
#earth {
  width: 5%;
  height: 5%;
}
#map {
  height: 100%;
  z-index: -1;
}

.el-row {
  color: black;
  margin-bottom: 5px;
  display: flex;
  flex-wrap: wrap;
}
/*隐藏ol的一些自带元素*/
.ol-attribution,
.ol-zoom {
  display: none;
}
#header,
#menu,
#aboutContainer,
#delete-wrapper {
  z-index: 1;
}
</style>
