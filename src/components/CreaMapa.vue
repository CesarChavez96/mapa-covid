<template>
  <div style="height: 100%" align="center">
    <head>
      <link
        rel="stylesheet"
        href="https://unpkg.com/leaflet@1.1.0/dist/leaflet.css"
      />
    </head>
    <div style="height: 200px overflow: auto;">
      <p>Mapa COVID-19 {{currentCenter}}  {{currentZoom}}</p>
      <span v-if="loading">Loading...</span>
    </div>
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 400px; width: 700px;"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker 
        v-for="pais in covid"
        :key="pais.pais"
        :lat-lng="pais.localizacion"
        :icon="icon">
          <l-popup>
            <div>
              <p>Pais:{{pais.pais}}</p>
              <p>Casos confirmados:{{pais.confirmados}}</p>
              <p>Recuperados:{{pais.recuperados}}</p>
              <p>Defunciones:{{pais.defunciones}}</p>
              <p>Fecha de actualizacion:{{pais.fecha}}:</p>
            </div>
          </l-popup>
      </l-marker>
      <l-marker
        v-for="estado in estados"
        :key="estado.estado"
        :lat-lng="estado.localizacion"
        :icon="icon">
          <l-popup>
            <div>
              <p>Pais:{{estado.estado}}</p>
              <p>Casos confirmados:{{estado.confirmados}}</p>
              <p>Recuperados:{{estado.recuperados}}</p>
              <p>Defunciones:{{estado.defunciones}}</p>
              <p>Fecha de actualizacion:{{estado.fecha}}:</p>
            </div>
          </l-popup>
      </l-marker>
      <l-marker
        v-for="pais in paises"
        :key="pais.pais"
        :lat-lng="pais.localizacion"
        :icon="icon2">
        <div>{{currentLugar}}</div>
          <l-popup>
            <div>
              <p>Pais:{{pais.pais}}</p>
              <p>Mensaje:{{pais.mensaje}}</p>
              <p>Fecha de actualizacion:{{pais.fecha}}:</p>
            </div>
          </l-popup>
      </l-marker>
    </l-map>
  </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
import { latLng, icon } from "leaflet";
import { LMap, LTileLayer, LMarker, LPopup, LTooltip } from "vue2-leaflet";

Vue.use(VueAxios, axios);
var options = {
  headers: {
    "x-rapidapi-key": "a38d2d83acmshd6d851d19ec0b6cp1090f6jsn6c9549ad67ab",
  },
};

export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LTooltip,
  },
  data() {
    return {
      zoom: 1.3,
      center: latLng(23.220975, 9.427278),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      withPopup: latLng(47.41322, -1.219482),
      currentZoom: 0,
      currentCenter: latLng(0, 0),
      loading: true,
      mapOptions: {
        zoomSnap: 0.3,
      },
      showMap: true,
      icon: icon({
        iconUrl: require("@/assets/biohazard.png"),
        iconSize: [20, 20],
      }),
      icon2: icon({
        iconUrl: require("@/assets/biohazard-256.png"),
        iconSize: [20, 20],
      }),
      paises: [],
      estados: [],
      location: [],
      lugar: "Guanajuato",
      currentLugar: "",
      covid: [],
    };
  },
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
  },
  async created() {
    try {
      var meses = new Array(
        "Enero",
        "Febrero",
        "Marzo",
        "Abril",
        "Mayo",
        "Junio",
        "Julio",
        "Agosto",
        "Septiembre",
        "Octubre",
        "Noviembre",
        "Diciembre"
      );
      let f = new Date();
      const esta = await Vue.axios({
        url: `https://covid-1931.p.rapidapi.com/`,
        headers: {
          "x-rapidapi-key":
            "a38d2d83acmshd6d851d19ec0b6cp1090f6jsn6c9549ad67ab",
        },
      });
      const infoestados = esta.data[0].states.fields;
      console.log(esta);
      let loc = esta.data[0].loc;
      let estado = [];
      for (let i = 0; i < infoestados.length; i++) {
        let estad = infoestados[i].mapValue.fields;
        let loca = infoestados[i].mapValue.fields.state.stringValue;
        loca = loca.trim();
        estado[i] = {
          estado: estad.displayName.stringValue,
          confirmados: estad.confirmed.integerValue,
          recuperados: estad.recovered.integerValue,
          defunciones: estad.deaths.integerValue,
          localizacion: loc[loca],
          fecha: `${f.getDate()} de ${meses[f.getMonth()]}`,
        };
      }

      for (let es = 0; es < estado.length; es++) {
        estado[es].localizacion = latLng(
          estado[es].localizacion[0].LATITUDE,
          estado[es].localizacion[0].LONGITUDE
        );
      }
      this.estados = estado;

      const { data } = await Vue.axios({
        url: `https://covid19-data.p.rapidapi.com/all`,
        headers: {
          "x-rapidapi-key":
            "a38d2d83acmshd6d851d19ec0b6cp1090f6jsn6c9549ad67ab",
        },
      });
      this.paises = data;
      console.log(data);
      const arreglo2 = this.paises;
      let promises1 = [];
      let promises = [];
      let const1 = 0;
      let const2 = 0;

      for (let i = 0; i < arreglo2.length; i++) {
        if (arreglo2[i].country === "Mexico") {
          promises1[const1] = {
            pais: arreglo2[i].country,
            confirmados: arreglo2[i].confirmed,
            recuperados: arreglo2[i].recovered,
            defunciones: arreglo2[i].deaths,
            localizacion: latLng(arreglo2[i].latitude, arreglo2[i].longitude),
            fecha: `${f.getDate()} de ${meses[f.getMonth()]}`,
          };
          const1++;
        } else {
          if (arreglo2[i].confirmed != 0 && arreglo2[i].latitude != null) {
            promises1[const1] = {
              pais: arreglo2[i].country,
              confirmados: arreglo2[i].confirmed,
              recuperados: arreglo2[i].recovered,
              defunciones: arreglo2[i].deaths,
              localizacion: latLng(arreglo2[i].latitude, arreglo2[i].longitude),
              fecha: `${f.getDate()} de ${meses[f.getMonth()]}`,
            };
            const1++;
          } else if (arreglo2[i].latitude != null) {
            promises[const2] = {
              pais: arreglo2[i].country,
              mensaje: "Libre de covid",
              localizacion: latLng(arreglo2[i].latitude, arreglo2[i].longitude),
              fecha: `${f.getDate()} de ${meses[f.getMonth()]}`,
            };
            const2++;
          }
        }
      }

      this.covid = promises1;
      this.paises = promises;
      console.log(this.paises);

      //this.currentCenter = this.currentCenter + 20;
      console.log(this.currentCenter.lat + 20);

      /*async function getMultiple(objectsToGet) {
        let users = [];
        let promises = [];
        for (let i = 0; i < objectsToGet.length; i++) {
          let pais = encodeURI(objectsToGet[i].pais);
          promises.push(
            axios.get(
              `https://covid-19-data.p.rapidapi.com/country?name=${pais}`,
              options
            )
          );
        }
        await Promise.all(promises).then(response => users.push(response));
        return users;
      }

      console.log(await getMultiple(promises1));
      Promise.all(promi)
        .then(response => {
          this.covid = response;
          console.log(this.covid);
        })
        .catch(er => {
          console.log(er);
        });*/
    } catch (e) {
      console.log(e.message);
    } finally {
      console.log("La peticon para obtener los todos ha finalizado");
      this.loading = false;
    }
  },
};
</script>
