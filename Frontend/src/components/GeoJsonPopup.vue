<template>
  <div style="width:800px; height: 550px">
    <div class="md-layout">
      <div class="md-layout-item md-size-60 md-alignment-top-right">
        <div class="md-title"><h2 style="padding-top:0px; margin-top:0px;">{{buildingName}}</h2></div>
        </div>
        <div class="md-layout-item md-size-40 md-alignment-top-right">
          <p style="padding-top:0px; margin-top:10px; float:right;">
            This is the {{buildingId}}th biggest Consumer on average</p>
        </div>
    </div >
   
    <div class="md-layout">
      <div class="md-layout-item md-size-50 md-alignment-top-right">
        <md-radio v-model="mode" value="Day" style="margin-top:0px">Day</md-radio>
        <md-radio v-model="mode" value="Week" style="margin-top:0px">Week</md-radio>  
        <md-radio v-model="mode" value="Year" style="margin-top:0px">Year</md-radio>  
      </div>

      <div class="md-layout-item md-size-50 md-alignment-top-right">
        <md-tooltip md-direction="top">Add building to comparison</md-tooltip>
        <md-switch @change="compareBuilding" v-model="addSetToCompare" style="float:right;" class="md-primary md-alignment-center-right">Compare Building</md-switch>
      </div>
    </div >

    <relative-building-bar-chart :buildingId="buildingId" :chartRange="mode" v-if="isVisible"/>
        
    <div v-observe-visibility="visibilityChanged"></div>

  </div>
</template>

<script>
import RelativeBuildingBarChart from "./charts/RelativeBuildingBarChart";

import store from "./../store/";
import Vue from "vue";

export default {
  name: "GeoJson2Popup",
  components: {
    RelativeBuildingBarChart
  },
  data: function() {
    return {
      mode: "Week",
      addSetToCompare: false,
      isVisible: false,
      normalized: false
    };
  },
  props: {
    buildingId: {
      type: Number,
      default: 0
    },
    buildingName: {
      type: String,
      default: ""
    }
  },
  methods: {
    visibilityChanged(isVisible, entry) {
      this.isVisible = isVisible;
      this.toggleSwitch()
      if (this.isVisible == true) {
        console.log(
          "loading building information for building",
          this.buildingId
        );
        store.dispatch("loadBuildingInformation", this.buildingId);
      }
    },
    compareBuilding() {
      console.log("I SWITCHED", this.buildingId);
      if (this.addSetToCompare) {
        store.dispatch("addComparisonBuilding", this.buildingId);
      } else {
        store.dispatch("removeComparisonBuilding", this.buildingId);
      }
    },
    toggleSwitch() {
      if (store.state.databaseStore.comparisonArray.includes(this.buildingId)) {
        this.addSetToCompare = true;
      }
      if (
        !store.state.databaseStore.comparisonArray.includes(this.buildingId)
      ) {
        this.addSetToCompare = false;
      }
    }
  },
  mounted() {
    this.toggleSwitch();
  },
  computed: {}
};
</script>