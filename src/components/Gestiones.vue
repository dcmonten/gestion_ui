<template>
  <div class="gestiones">
    <h1>Gestiones</h1>
    <div v-if="test">
      <small class="d-block">Cache connected: {{ test.redis_up }}</small>
      <small class="d-block">Status: {{ test.msg }}</small>
    </div>
    <p class="w-100 lead">
      Al hacer clic en "Cargar desde ...", el programa envía 10 solicitudes
      consecutivas, ya sea a la cache o a la base de datos, y guarda los tiempos
      en un arreglo.
    </p>
    <div class="btn-group py-3">
      <button
        class="btn btn-primary"
        @click="getGestionesDB()"
        :disabled="loading"
      >
        Cargar desde base de datos
      </button>
      <button
        class="btn btn-info"
        @click="getGestionesCached()"
        :disabled="loading"
      >
        Cargar desde caché (redis)
      </button>

      <button class="btn btn-light" v-if="start == 0" @click="resetStart()">
        Resetear contador
      </button>
    </div>
    <div v-if="loading == false">
      <div class="py-4" v-if="gestiones.length == 0">
        Haga clic en cualquiera de los botones para empezar.
      </div>
    </div>
    <div v-else>
      <div v-if="start > 0">Quedan {{ start }} solicitudes por enviar...</div>
      <div class="spinner-border text-primary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>
    <div class="container">
      <div class="row">
        <div class="col-lg-6 col-md-12" v-if="cache_times.length > 0">
          <h4>Caché</h4>
          <table class="table">
            <thead>
              <tr>
                <th scope="col">Tiempo (ms)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(time, index) in cache_times" :key="index">
                <th scope="row">{{ time }}</th>
              </tr>
              <tr>
                <th scope="row">Promedio: {{ avg_cache }}</th>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="col-lg-6 col-md-12" v-if="db_times.length > 0">
          <h4>BD Solamente</h4>
          <table class="table">
            <thead>
              <tr>
                <th scope="col">Tiempo (ms)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(time, index) in db_times" :key="index">
                <th scope="row">{{ time }}</th>
              </tr>
              <tr>
                <th scope="row">Promedio: {{ avg_db }}</th>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="col-12">
          <p class="lead" v-if="db_times.length > 0 && cache_times.length > 0">
            Las solicitudes directo a la base de datos toman {{compare_db_cache * 100}} más tiempo que las solicitudes con cache.  
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "Gestiones",
  data: function() {
    return {
      gestiones: [],
      test: {},
      loading: false,
      start: 10,
      cache_times: [],
      db_times: []
    };
  },
  computed: {
    baseURL() {
      return "http://dist-sys-gestiones.herokuapp.com/api/";
    },
    db() {
      return this.baseURL + "gestion";
    },
    cache() {
      return this.baseURL + "gestion/cache";
    },
    test_cache() {
      return this.baseURL + "cache";
    },
    avg_cache() {
      return (
        this.cache_times.reduce((a, b) => a + b, 0) / this.cache_times.length
      );
    },
    avg_db() {
      return this.db_times.reduce((a, b) => a + b, 0) / this.db_times.length;
    },
    compare_db_cache(){
      return (this.avg_db-this.avg_cache)/this.avg_cache;
    }
  },
  methods: {
    getGestionesCached() {
      let $vm = this;
      this.loading = true;
      let start_time = new Date().getTime();
      axios
        .get(this.cache)
        .then(response => {
          let request_time = new Date().getTime() - start_time;
          $vm.cache_times.splice(1, 0, request_time);
          if ($vm.start > 0) {
            $vm.start--;
            $vm.getGestionesCached();
          }
          $vm.gestiones = response.data;
        })
        .catch(error => {
          alert(error);
        })
        .finally(() => {
          if ($vm.start == 0) $vm.loading = false;
        });
    },
    getGestionesDB() {
      this.loading = true;
      let $vm = this;
      let start_time = new Date().getTime();
      axios
        .get(this.db)
        .then(response => {
          let request_time = new Date().getTime() - start_time;
          $vm.db_times.splice(1, 0, request_time);
          if ($vm.start > 0) {
            $vm.start--;
            $vm.getGestionesDB();
          }
          $vm.gestiones = response.data;
        })
        .catch(error => {
          alert(error);
        })
        .finally(() => {
          if ($vm.start == 0) $vm.loading = false;
        });
    },
    resetStart() {
      this.start = 10;
    }
  },
  mounted() {
    let $vm = this;
    axios
      .get(this.test_cache)
      .then(response => {
        $vm.test = response.data;
      })
      .catch(error => {
        alert(error);
      });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss"></style>
