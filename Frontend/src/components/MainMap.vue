<template>
  <div>
      <div class="md-layout" style="padding-top:0px; margin-top:0px; padding-left:20px">
      <div class="md-layout-item md-size-25">
        <p>Plotted Values
        <md-radio v-model="range" value="Average" style="padding-left:20px; margin-top:0px">
          <md-tooltip md-direction="top">Average day/week/year</md-tooltip>Average</md-radio>
        <md-radio v-model="range" value="Maximum" style="margin-top:0px"><md-tooltip md-direction="top">
          Highest past day/week/year</md-tooltip>Maximum</md-radio>  
        </p>
      </div>
      <div class="md-layout-item md-size-25">
        <md-tooltip md-direction="top">Preiod of time</md-tooltip>
        <p>Heatmap Period
        <md-radio v-model="mode" value="Day" style="padding-left:20px; margin-top:0px">Day</md-radio>
        <md-radio v-model="mode" value="Week" style="margin-top:0px">Week</md-radio>  
        <md-radio v-model="mode" value="Year" style="margin-top:0px">Year</md-radio>  
        </p>
      </div>
      <div class="md-layout-item md-sise-50" style="padding-right:50px">
        <div class="colourGrad" style="width:100%"/>
        <div class="md-layout">    
          <div class="md-layout-item md-size-50">
        <p style="margin-bottom:0px">Min: <strong>0</strong></p>
          </div>
        <div class="md-layout-item md-size-50">
          <p style="float:right; margin-bottom:0px">Max: <strong>{{heatMapMaximum}}</strong></p>
          </div>
        </div>
      </div>
  
      </div>
      <l-map
        :zoom="zoom"
        :center="center"
        style="height: 900px; width: 100%"
        :options="options">
        <l-tile-layer
          :url="url"
          :attribution="attribution"/>
        <!-- <l-geo-json
          :geojson="bus.geojson"
          :options="bus.options"/> -->
        <l-geo-json
          :geojson="campusData.geojson"
          :options="campusData.options"
          
          />
      </l-map>
    </div>
</template>

<script>
import Vue from "vue";
import { LMap, LTileLayer, LGeoJson } from "vue2-leaflet";
import PopupContent from "./GeoJsonPopup";
// var baseballIcon = L.icon({
//   iconUrl: 'static/images/baseball-marker.png',
//   iconSize: [32, 37],
//   iconAnchor: [16, 37],
//   popupAnchor: [0, -28]
// });
function onEachFeature(feature, layer) {
  let PopupCont = Vue.extend(PopupContent);
  let popup = new PopupCont({
    propsData: {
      buildingId: feature.properties.buildingId,
      buildingName: feature.properties.buildingName
    }
  });
  layer.bindPopup(popup.$mount().$el, {
    maxWidth: "auto",
    maxHeight: 1000,
    autoPan: true
  });
}
export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LGeoJson
  },
  methods: {},
  mounted() {
    this.$store.dispatch("loadMapGeoJson");
  },
  data() {
    // 'https://api.tiles.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}.png?access_token=
    return {
      campus: "MainCampus",
      range: "Average",
      mode: "Week",
      zoom: 16.5,
      center: [-26.1893, 28.0266],
      url:
        "https://api.tiles.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoiY2hyaXNtYXJlZSIsImEiOiJjam1sdW5tMHAwOHlxM2tudWJtMGVnOXltIn0.FdRDSOeZfQ2cQEeEzHwyvw",
      attribution:
        'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, ' +
        '<a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
        'Imagery © <a href="http://mapbox.com">Mapbox</a>',
      options: {
        zoomControl: false,
        scrollWheelZoom: false,
        attributionControl: false
      }
    };
  },
  computed: {
    heatMapMaximum() {
      try {
        if (
          this.$store.state.databaseStore.campusInfo.MaximumLastPeriodAverage
        ) {
          if (this.range == "Average") {
            return Math.floor(
              this.$store.state.databaseStore.campusInfo
                .MaximumLastPeriodAverage[this.mode]
            );
          } else {
            return Math.floor(
              this.$store.state.databaseStore.campusInfo.Maximums[this.mode]
            );
          }
        }
      } catch (e) {}
    },
    campusData() {
      let campusDataObject = {};
      try {
        // console.log(this.mode)
        let chartMode = this.mode + "Style_" + this.range;
        console.log(chartMode);
        campusDataObject = {
          geojson: [this.$store.state.databaseStore.mapGeoJson],
          options: {
            style: function(feature) {
              return feature.properties && feature.properties[chartMode];
            },
            onEachFeature: onEachFeature
          }
        };
      } catch (e) {}
      return campusDataObject;
    }
  }
};
</script>

<style>
.colourGrad {
  height: 25px;
  width: 100%;
  background-color: red; /* For browsers that do not support gradients */
  background-image: linear-gradient(
    to right,
    blue,
    green,
    red
  ); /* Standard syntax (must be last) */
}
</style>
