<template>
  <b-row>
    <b-col cols="12" md="4" sm="12">
      <div class="px-2">
        <b-card class="mb-3">
          <b-card-body>
            <b-card-title>Details</b-card-title>
            <b-card-text>
              Latitude: {{this.coordinates.lat}}
            </b-card-text>
            <b-card-text>
              Longitude: {{this.coordinates.lng}}
            </b-card-text>

            <label for="range-1">Radius: {{this.radius}} Kilometers</label>
            <b-form-input id="range-1" v-model="radius" type="range" min="0" max="10"></b-form-input>
          </b-card-body>
        </b-card>

        <b-card class="mb-3">
          <b-card-body>
            <b-card-title>Population</b-card-title>
            <b-card-text>
              20,000 - 22,000
            </b-card-text>
          </b-card-body>
        </b-card>

        <b-card class="mb-3">
          <b-card-body>
            <b-card-title>Ave. Socio Economic Level</b-card-title>
            <b-card-text>
              Med High
            </b-card-text>
          </b-card-body>
        </b-card>

        <b-card class="mb-3">
          <b-card-body>
            <b-card-title>Registered businesses</b-card-title>
            <b-card-text>
              5,640
            </b-card-text>
          </b-card-body>
        </b-card>

        <b-button block>View full report</b-button>

      </div>
    </b-col>
    <b-col cols="12" md="8" sm="12">
      <div id="map" style="height:100vh"></div>
    </b-col>
  </b-row>
</template>

<script>
export default {
  name: 'IndexPage',
  mounted() {
    this.createMap();
    this.isMapLoaded();
  },
  watch: {
        radius: {
            handler(newRadius, old) {
              this.drawCircle(this.coordinates, newRadius);
            },
        },
    },
  data() {
    return {
      map: {},
      coordinates: {
        lat: 32.517209,
        lng: -117.026843
      },
      radius: 3,
      marker: null,
      zoomMarker: null,
      mapZoom: 19,
      mapPitch: 0,
      mapBearing: 0,
      access_token: "pk.eyJ1IjoiZWxnZXJhcmRvIiwiYSI6ImNsOG90NjFtMzFucG0zeWw1YWRheTV5ZmYifQ.87BCgCSXpjLIHkqGsWUW7g",
    }
  },
  methods: {
    createMap() {
      this.$mapboxgl.accessToken = this.access_token;
      this.map = new this.$mapboxgl.Map({
                container: "map",
                style: "mapbox://styles/javy3r18/cl9fvwqli000p15qskifof42o",
                zoom: 12,
                pitch: this.mapPitch, //inclination
                bearing: this.mapBearing, // rotation
                center: [-117.026843, 32.517209],
                antialias: true,
            });

    },
    isMapLoaded() {
      this.map.on("load", () => {
          console.log(this.coordinates)
          console.log("Radius: " + this.radius + " Kilometer")

          this.map.setCenter(this.coordinates);

          this.initMarker()

          this.drawCircle(this.coordinates, this.radius);
      })

    },
    createGeoJSONCircle(center, radius, points) {
      if(!points) points = 64;

      var coords = {
        latitude: center.lat,
        longitude: center.lng
      };

      var km = radius;
      var ret = [];
      var distanceX = km/(111.320*Math.cos(coords.latitude*Math.PI/180));
      var distanceY = km/110.574;
      var theta, x, y;

      for(var i=0; i<points; i++) {
          theta = (i/points)*(2*Math.PI);
          x = distanceX*Math.cos(theta);
          y = distanceY*Math.sin(theta);

          ret.push([coords.longitude+x, coords.latitude+y]);
      }
      ret.push(ret[0]);

      return {
        "type": "geojson",
        "data": {
            "type": "FeatureCollection",
            "features": [{
                "type": "Feature",
                "geometry": {
                    "type": "Polygon",
                    "coordinates": [ret]
                }
            }]
        }
      };
    },
    initMarker() {
            this.marker = new this.$mapboxgl.Marker({
                color: "black",
                draggable: false,
            }).setLngLat(this.coordinates).addTo(this.map);
      },
    drawCircle(center, radius) {
      if (this.map.getLayer('polygon')) {
        let circle = this.createGeoJSONCircle(center, radius)
        this.map.getSource('polygon').setData(circle.data);
      }
      else {
        this.map.addSource("polygon", this.createGeoJSONCircle(center, radius));
        this.map.addLayer({
          "id": "polygon",
          "type": "fill",
          "source": "polygon",
          "layout": {},
          "paint": {
              "fill-color": "blue",
              "fill-opacity": 0.6
          }
      });

      }

      
    }
  }
}
</script>
