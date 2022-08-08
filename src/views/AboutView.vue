<template>
  <div class="contenedor_principal">
    <div class="contenedor_formulario">
      <div class="noticia">
        <img class="centro" src="@/assets/logo_cruzdelsur.png">
        <aside class="parrafo">PUNTO ORIGEN</aside>
        <b-form-select v-model="select" :options="this.city"></b-form-select>
        <aside class="parrafo">PUNTO DESTINO</aside>
        <b-form-select v-model="select_2" :options="this.city"></b-form-select>
        <div class="reset"></div>
        <button v-on:click="generar" class="btn">Generar</button>
      </div>
      <div class="noticia">
        <aside>RUTA GENERADA</aside>
        <div class="noticia_2">
          <p class="parrafo_resizable" v-for="(c) in generate" v-bind:key="c.value" >{{c.city_00 + '-' + c.city_01}}</p>
          <div class="reset"></div>
        </div>
        <div class="reset"></div>
      </div>
      <div class="noticia">
        <aside>LEYENDA</aside>
        <div class="noticia_2">
          <img class="izquierda" src="@/assets/marcador-de-posicion-principal.png">
          <aside>PUNTO PRINCIPAL</aside>
          <div class="reset"></div>
        </div>
        <div class="noticia_2">
          <img class="izquierda" src="@/assets/marcador-de-posicion-secundario.png">
          <aside>PUNTO INTERMEDIO</aside>
          <div class="reset"></div>
        </div>
        <div class="noticia_2">
          <img class="izquierda" src="@/assets/marcador-de-posicion-elegido.png">
          <aside>PUNTO SELECCIONADO</aside>
          <div class="reset"></div>
        </div>
        <div class="noticia_2">
          <img class="izquierda" src="@/assets/barra_negra.webp">
          <aside>RUTA GENERAL</aside>
          <div class="reset"></div>
        </div>
        <div class="noticia_2">
          <img class="izquierda" src="@/assets/barra_roja.png">
          <aside>RUTA SELECCIONADA</aside>
          <div class="reset"></div>
        </div>
        <div class="reset"></div>
      </div>
    </div>
    <div id="contenedor_mapa">
      <l-map :zoom="zoom" :center="center">
        <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
        <div v-for="(cty,ctys) in cities" v-bind:key="ctys">
          <l-marker :lat-lng="[cty.lat,cty.lon]" :icon="icon" v-if="cty.tip === 'P'">
            <l-tooltip>{{cty.text}}</l-tooltip>
          </l-marker>
          <l-marker :lat-lng="[cty.lat,cty.lon]" :icon="icon_2" v-if="cty.tip === 'I'">
            <l-tooltip>{{cty.text}}</l-tooltip>
          </l-marker>
          <l-marker :lat-lng="[cty.lat,cty.lon]" :icon="icon_3" v-if="cty.tip === 'E'">
            <l-tooltip>{{cty.text}}</l-tooltip>
          </l-marker>
        </div>
        <l-polyline v-for="(grha) in grhap" v-bind:key="grha.value" :lat-lngs="[[grha.lat_1,grha.lon_1],[grha.lat_2,grha.lon_2]]" :color="grha.color"></l-polyline>
      </l-map>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import { LMap, LTileLayer, LMarker,LPolyline,LTooltip } from 'vue2-leaflet';

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPolyline,
    LTooltip
  },
  data () {
    return {
      auxiliar: 0,
      select: null,
      select_2: null,
      cities: [],
      grhap: [],
      generate: [],
      city: [
        {
          value: null,
          text: '---------- Seleccione Punto de Origen ----------'
        }
      ],
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
          '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 10,
      center: [-4.90389, -80.68528],
      markerLatLng: [-5.19449, -80.63282],
      icon: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/3237/3237519.png',
        iconSize: [32, 37],
        iconAnchor: [16, 37]
      }),
      icon_2: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/3237/3237510.png',
        iconSize: [32, 37],
        iconAnchor: [16, 37]
      }),
      icon_3: L.icon({
        iconUrl: 'https://cdn-icons-png.flaticon.com/512/6339/6339679.png',
        iconSize: [32, 37],
        iconAnchor: [16, 37]
      }),
      datosCity: {
        ciudadOrigen: null,
        ciudadDestino: null
      },
    };
  },
  created() {
    this.state = true
    this.state_2 = false
    axios.get("/getcity")
        .then((response) =>
        {
          let data = response.data;
          if (response.status === 200)
          {
            let cities = []
            data.forEach(element =>{
              let obj = {
                value: this.auxiliar,
                text: element.name,
                lat: element.latitud,
                lon: element.longitud,
                tip: element.type
              }
              let auxiliar = {
                value: element.name,
                text: element.name
              }
              this.city.push(auxiliar)
              cities.push(obj);
              this.auxiliar = this.auxiliar + 1
            });
            this.cities = cities;
          }
          else
          {
            console.log("No se pudo cargar Ciudades");
          }
        })
        .catch(() =>
        {
          console.log("Error Interno");
        });

    axios.get("/getgrhaps")
        .then((response) =>
        {
          let data = response.data;
          if (response.status === 200)
          {
            let grhap = []
            data.forEach(element =>{
              let obj = {
                value: this.auxiliar,
                city_1: element.city_1,
                lat_1: element.latitud_1,
                lon_1: element.longitud_1,
                city_2: element.city_2,
                lat_2: element.latitud_2,
                lon_2: element.longitud_2,
                distancia: element.distancia,
                color: 'black'
              }
              grhap.push(obj);
              this.auxiliar = this.auxiliar + 1
            });
            this.grhap = grhap;
          }
          else
          {
            console.log("No se pudo cargar Ciudades");
          }
        })
        .catch(() =>
        {
          console.log("Error Interno");
        });
    console.log(this.auxiliar)
  },
  methods: {
    generar(){
      this.datosCity.ciudadOrigen = this.select
      this.datosCity.ciudadDestino = this.select_2
      if(this.datosCity.ciudadOrigen == null || this.datosCity.ciudadDestino == null){
        alert("Tiene que seleccionar los dos puntos")
      }else{
        var a = 0
        console.log(this.datosCity)

          let data = {
            origen: this.datosCity.ciudadOrigen,
            destino: this.datosCity.ciudadDestino
          }

          axios.post("/getroute",data)
            .then((response)=>{
              let date = response.data;
              if (response.status === 200)
              {
                let aux = []
                date.forEach(element =>{
                  let obj = {
                    value: a,
                    city_00: element.city_00,
                    city_01: element.city_01
                  }
                  aux.push(obj)
                  a = a + 1
                });
                this.generate = aux
                console.log(this.generate)
              }
              else
              {
                console.log("No se puede Generar la Ruta");
              }
            })
            .catch(() =>
            {
              console.log("Error Interno");
            });

          axios.post("/setcity",data)
              .then((response) =>
              {
                let datee = response.data;
                console.log(response.status)
                if (response.status === 200)
                {
                  let c = []
                  datee.forEach(element =>{
                    let obj = {
                      value: a,
                      text: element.name,
                      lat: element.latitud,
                      lon: element.longitud,
                      tip: element.type
                    }
                    c.push(obj);
                    a = a + 1
                  });
                  this.cities = c
                  console.log(this.cities)
                }
                else
                {
                  console.log("No se puede Generar la Ruta");
                }
              })
              .catch(() =>
              {
                console.log("Error Interno");
              });

          axios.post("/setgraph",data)
              .then((response) =>
              {
                let date = response.data;
                console.log(response.status)
                if (response.status === 200)
                {
                  let g = []
                  date.forEach(element =>{
                    let obj = {
                      value: a,
                      city_1: element.city_1,
                      lat_1: element.latitud_1,
                      lon_1: element.longitud_1,
                      city_2: element.city_2,
                      lat_2: element.latitud_2,
                      lon_2: element.longitud_2,
                      distancia: element.distancia,
                      color: element.color
                    }
                    g.push(obj);
                    a = a + 1
                  });
                  this.grhap = []
                  this.grhap = g
                  console.log(this.grhap)
                }
                else
                {
                  console.log("No se pudo cargar Ciudades");
                }
              })
              .catch(() =>
              {
                console.log("Error Interno");
              });
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.contenedor_principal{
  display: flex;
}
.contenedor_formulario{
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 54vh;
  background: #006a7c;
  overflow-y: scroll;
}
#contenedor_mapa{
  height: 100vh;
  width: 146vh;
  margin: 0;
  padding: 0;
}

div.noticia {
  margin: 10px 5px 10px 5px;
  background: #006a7c;
  color: white;
  padding: 10px;
  font-size: 20px;
  border: 2px solid;
  border-radius: 10px;
  width: 490px;
}

div.noticia img.centro{
  float: top;
  width: 400px;
  height: 150px;
}

div.noticia_2 {
  width: 100%;
  margin: 10px auto;
  background: #eabd00;
  color: black;
  font-size: 20px;
  border-radius: 10px;
}

div.noticia_2 img.izquierda {
  float: left;
  margin-right: 15px;
  width: 30px;
  height: 30px;
}

div.reset {
  clear: both;
}


.parrafo{
  color: white;
  text-align: left;
  margin: 10px;
}

.parrafo_resizable{
  box-sizing: border-box;
  width: 100%;
  padding: 0;
  margin: 0;
}

.btn{
  width: 100px;
  margin-top: 15px;
  background: #42b983;
}
.btn:hover{
  background: aquamarine;
}
</style>
