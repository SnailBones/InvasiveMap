<template>
    <div id="map-container">
      <nav id="menu" class="menu">
        {{currentYear}}
        <div class="range-slider">
          <input class = "range-slider__range" id="slider" type="range" v-model.number="currentYear" v-bind:min="startYear" v-bind:max="endYear" step="1" value="2000" />
        </div>
      </nav>
      <div id="map"></div>
    </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
// import dataPoints from "@/assets/data/zebra_mussel.geo.json";
import dataPoints from "@/assets/data/olive.geo.json"

export default {
  name: "Map",
  name: "Visualization",
  data() {
      return {
          // map: null,
          endYear: 0,
          startYear: 3000,
          currentYear: 2000
      };
  },
  methods: {
    prepareData(data){
      let min = 0
      let m
      data.features.forEach(point => {
        console.log("point.geometry is", point.geometry)
        // if (!"ObsDate" in point.properties){
        if (!point.properties.ObsDate){
          // console.log("no observation date :(");
          point.properties.Time = 0;
          return;
        }
        let year = parseInt(point.properties.ObsDate.slice(-4))
        point.properties.Time = year
        if (year < this.startYear ){
          this.startYear = year
        }
        else if (year > this.endYear ){
          this.endYear = year
        }
      })
      return data
    },
    updateMap(){
      let self = this
      // See data in a range of 60 years
      let backTime = 60
      // Filter out points in the future
      this.map.setFilter('point', ['<', 'Time', self.currentYear]);
      this.map.setFilter('heatmap', ['<', 'Time', self.currentYear]);
      this.map.setPaintProperty("point", "circle-opacity",
      [
          "interpolate",
          ["linear"],
          ["get", "Time"],
          self.currentYear-backTime,
          0,
          self.currentYear,
          .2
      ]
    )
      this.map.setPaintProperty("point",
              "circle-radius",
              [

                      "*",
                      [
                          "-",
                          ["get", "Time"],
                          self.currentYear - backTime,
                      ],
                      .1
                      // 0
                ]
              )
      this.map.setPaintProperty("heatmap",
      "heatmap-weight",
      [
          "-",
          ["get", "Time"],
          (self.currentYear - backTime)
      ]
    )
    }
  },
  mounted() {
    let points = this.prepareData(dataPoints)
    mapboxgl.accessToken =
      "pk.eyJ1Ijoic25haWxib25lcyIsImEiOiJjamJvdGlueG41bzB5MzRxZnlsbTM4OGNpIn0.g4DLrr29EfW3otLvTfzt9Q";
    this.map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/mapbox/light-v9",
      center: [-106.629181, 35.106766],
      zoom: 4,
      interactive: true
    });
    let self = this
    let map = this.map

    map.on("load", function() {
        map.addSource("pointData", {
          type: "geojson",
          data: points
        });
        let heatmap_layer =
          {
            id: "heatmap",
            type: "heatmap",
            source: "pointData",
            maxzoom: 13,
            paint: {
              "heatmap-intensity": [
                "interpolate",
                ["linear"],
                ["zoom"],
                0,
                .01,
                9,
                .03
              ],
              "heatmap-color": ["interpolate",["linear"],["heatmap-density"],
              0,"rgba(0, 0, 0, .0)",0.1,"rgba(255, 233, 53, 0.4)",0.3,"#feb24c",0.5,"#fd8d3c",0.7,"#fc4e2a",1,"#e31a1c"],

              // Adjust the heatmap radius by zoom level
              "heatmap-radius": [
                "interpolate",
                ["linear"],
                ["zoom"],
                0,
                2,
                9,
                20
              ],
              // Transition from heatmap to circle layer by zoom level
              "heatmap-opacity": [
                "interpolate",
                ["linear"],
                ["zoom"],
                6,
                0.5,
                13,
                0
              ]
            }
          }

        let point_layer =
          {
            id: "point",
            type: "circle",
            source: "pointData",
            minzoom: 4,
            paint: {
              "circle-color":     [
                      "interpolate",
                      ["linear"],
                      ["get", "Time"], // color based on load
                      self.startYear,
                      "#a61",
                      self.endYear,
                      "#41a"
                ],
              "circle-opacity": [
                "interpolate",
                ["linear"],
                ["zoom"],
                4,
                0.0,
                8,
                1.0
              ],
              "circle-stroke-color": "rgba(255,255,255,.75)",
              "circle-stroke-width": 1,
              "circle-stroke-opacity": [
                "interpolate",
                ["linear"],
                ["zoom"],
                12,
                0,
                20,
                1
              ]
            }
          }

        map.addLayer(point_layer)
        map.addLayer(heatmap_layer)
        self.updateMap()
      })

      map.on("click", "point", function(e) {
        let clicked = e.features[0];
        new mapboxgl.Popup()
          .setLngLat(clicked.geometry.coordinates)
          .setHTML(
            Object.entries(clicked.properties)
              .map((k, v) => "<p>" + k[0] + ": " + k[1] + "</p>")
              .join("")
          )
          .addTo(map);
      });
  },
  watch:{
    currentYear(year){
       this.updateMap()
    }

  }
};
</script>
<style lang="scss">
#map {
    position: absolute;
    height: 100%;
    width: 100%;
}
#menu {
  background: #fff;
  position: absolute;
  z-index: 1;
  top: 10px;
  left: 10px;
  border-radius: 3px;
  width: 220px;
  border: 1px solid rgba(0, 0, 0, 0.4);
  font-family: "Open Sans", sans-serif;

    .slidecontainer {
      width: 100%;
    }

    .slider {
      -webkit-appearance: none;
      width: 100%;
      height: 25px;
      background: #d3d3d3;
      outline: none;
      opacity: 0.7;
      -webkit-transition: .2s;
      transition: opacity .2s;
    }

    .slider::slider-thumb {
      width: 25px;
      height: 25px;
      background: #4CAF50;
      cursor: pointer;
    }

    .slider:hover {
      opacity: 1;
    }
}
.mapboxgl-popup-content p {
  margin: 0;
  opacity: 0.5;
}

.range-slider {
  margin: 60px 0 0 0%;
}


// Range Slider
$range-width: 100% !default;

$range-handle-color: rgb(73, 66, 103);
$range-handle-color-hover: rgb(171, 145, 93);
$range-handle-size: 20px !default;

$range-track-color: rgb(45, 50, 66);
$range-track-height: 10px !default;

$range-label-color: rgb(45, 50, 66);
$range-label-width: 60px !default;

.range-slider {
  width: $range-width;
}

.range-slider__range {
  -webkit-appearance: none;
  width: calc(100% - (#{$range-label-width + 13px}));
  height: $range-track-height;
  border-radius: 5px;
  background: $range-track-color;
  outline: none;
  padding: 0;
  margin: 0;

  // Range Handle
  &::-webkit-slider-thumb {
    appearance: none;
    width: $range-handle-size;
    height: $range-handle-size;
    border-radius: 50%;
    background: $range-handle-color;
    cursor: pointer;
    transition: background .15s ease-in-out;

    &:hover {
      background: $range-handle-color-hover;
    }
  }

  &:active::-webkit-slider-thumb {
    background: $range-handle-color-hover;
  }

  &::-moz-range-thumb {
    width: $range-handle-size;
    height: $range-handle-size;
    border: 0;
    border-radius: 50%;
    background: $range-handle-color;
    cursor: pointer;
    transition: background .15s ease-in-out;

    &:hover {
      background: $range-handle-color-hover;
    }
  }

  &:active::-moz-range-thumb {
    background: $range-handle-color-hover;
  }

  // Focus state
  &:focus {

    &::-webkit-slider-thumb {
      box-shadow: 0 0 0 3px #000,
                  0 0 0 6px #257;
    }
  }
}


// Range Label
.range-slider__value {
  display: inline-block;
  position: relative;
  width: $range-label-width;
  // color: #113;
  color: #eef;
  line-height: 20px;
  text-align: center;
  border-radius: 3px;
  background: $range-label-color;
  padding: 5px 10px;
  margin-left: 8px;

  &:after {
    position: absolute;
    top: 8px;
    left: -7px;
    width: 0;
    height: 0;
    border-top: 7px solid transparent;
    border-right: 7px solid $range-label-color;
    border-bottom: 7px solid transparent;
    content: '';
  }

}
</style>
