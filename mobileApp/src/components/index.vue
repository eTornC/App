<template>
  <div class="py-3 px-3">
    <div id="token-indicator">
      <span v-if="hasToken" @click="showToken">✅</span>
      <span v-else>❌</span>
    </div>

    <div class="row" v-if="tiquet">
      <div class="col-12 mb-2 d-flex justify-content-center">
        <div class="card" style="width: 18rem;">
          <h1 class="border-bottom text-center">TIQUET</h1>
          <div class="card-body text-center">
            <h3 class="card-title">{{tiquet.StoreName}}</h3>
            <p v-if="tiquet.turn.type='normal'" class="m-0">T{{tiquet.turn.number}}</p>
            <p v-else class="m-0">T{{tiquet.turn.created_at}}</p>
          </div>
        </div>
      </div>
    </div>
    <template v-for="store in stores">
      <!--div :key="store.name" class="store-app my-3 mx-4">
        <img
          :src="urls.host + urls.routes.prefix + store.photo_path"
          class="img-card-app .md-image"
        >
        <h2 class="mb-3 mt-2">{{ store.name }}</h2>

        <div class="boton_Torn-app row justify-content-center">
          <div class="mx-2">
            <button class="btn btn-primary" @click="normalTurn(store)">Demanar Turn</button>
          </div>
        </div>
      </div-->
      <div class="row" :key="store.id">
        <div class="col-12 d-flex justify-content-center">
          <div class="card" style="width: 18rem;">
            <img
              class="card-img-top"
              :src="urls.host + urls.routes.prefix + store.photo_path"
              alt="Card image cap"
            >
            <div class="card-body">
              <h2 class="card-text">{{ store.name }}</h2>
            </div>
          </div>
        </div>
        <div class="col-6 mt-2">
          <button
            type="button"
            class="btn btn-warning"
            data-toggle="modal"
            data-target="#exampleModal"
            @click="getNextBuckets(store)"
          >Reserva</button>
          <!-- Modal -->
          <div
            class="modal fade"
            id="exampleModal"
            tabindex="-1"
            role="dialog"
            aria-labelledby="exampleModalLabel"
            aria-hidden="true"
          >
            <div class="modal-dialog" role="document">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 class="modal-title" id="exampleModalLabel">Selecciona la hora</h5>
                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                  </button>
                </div>
                <div class="modal-body">
                  <ul class="list-group">
                    <li
                      :class="{ bucketSelect: bucket ==  bucketsSelect}"
                      class="list-group-item m-0"
                      v-for="(bucket,index) in bucketsList"
                      :key="index"
                    >
                      <div v-if="bucket.filled">{{hour(bucket.hour_start)}} ple</div>
                      <div v-else @click="selectBucket(bucket)">{{hour(bucket.hour_start)}}</div>
                    </li>
                  </ul>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Cerrar</button>
                  <button
                    type="button"
                    class="btn btn-primary"
                    data-dismiss="modal"
                    @click="hourTurn(store)"
                  >Reserva</button>
                </div>
              </div>
            </div>
          </div>
          <!-- Modal End -->

          <buckets-list/>
        </div>
        <div class="col-6 mt-2">
          <button class="btn btn-primary" @click="normalTurn(store)">Turn Ahora</button>
        </div>
        <div class="col-12">
          <hr>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import urls from "../api/config";
import axios from "axios";
window.$ = window.jQuery = require("jquery");

import Vue from "vue";
import datePicker from "vue-bootstrap-datetimepicker";
import "bootstrap/dist/css/bootstrap.css";
import "pc-bootstrap4-datetimepicker/build/css/bootstrap-datetimepicker.css";
Vue.use(datePicker);
import bucketsList from "./bucketsList.vue";
export default {
  components: {
    "buckets-list": bucketsList
  },
  data() {
    return {
      stores: [],
      urls: urls,
      showTurnsModal: false,
      storeModal: {},
      tiquet: null,
      bucketsList: null,
      bucketsSelect: null
    };
  },

  computed: {
    hasToken() {
      return (
        typeof this.$route.query.token !== "undefined" &&
        this.$route.query.token !== ""
      );
    }
  },

  mounted() {
    const url = urls.host + urls.routes.prefix + urls.routes.stores;
    axios
      .get(url)
      .then(res => {
        this.stores = res.data;
        console.log(this.stores);
      })
      .catch(err => {
        console.log(err);
      });
  },
  methods: {
    hour: function(text) {
      return text.split(" ")[1];
    },
    showTurnsListModal(store) {
      this.storeModal = store;
      this.showTurnsModal = true;
    },
    closeModal() {
      this.showTurnsModal = false;
    },
    selectBucket(bucket) {
      this.bucketsSelect = bucket;
    },
    getNextBuckets(store) {
      const url =
        urls.host +
        urls.routes.prefix +
        urls.routes.store +
        "/" +
        store.id +
        urls.routes.bucketsNextHour;
      console.log(url);
      axios
        .get(url, {})
        .then(res => {
          this.bucketsList = res.data;
          console.log(this.bucketsList);
        })
        .catch(err => {
          this.$swal("Failako");
        });
    },
    normalTurn(store) {
      const url =
        urls.host +
        urls.routes.prefix +
        urls.routes.store +
        "/" +
        store.id +
        urls.routes.turn;
      if (this.hasToken) {
        axios
          .post(url, {
            token: this.hasToken ? this.$route.query.token : ""
          })
          .then(res => {
            this.tiquet = res.data;
            this.tiquet.StoreName = store.name;
            console.log(this.tiquet);
            this.$swal({
              type: "success",
              title: "Turn demanat T" + res.data.turn.number,
              showConfirmButton: false,
              timer: 2000
            });
            console.log("Torn demanat T" + res.data.turn.number);
            //this.$swal('Imprimir tiquet' + JSON.stringify(res.data));
          })
          .catch(err => {
            this.$swal("Failako");
          });
      } else {
        this.$swal("Failako Token");
      }
    },
    hourTurn(store) {
      const url =
        urls.host +
        urls.routes.prefix +
        urls.routes.store +
        "/" +
        store.id +
        urls.routes.hourTurn;
      console.log(this.bucketsSelect);

      if (this.hasToken) {
        if (this.bucketsSelect != null) {
          var data = new Date(this.bucketsSelect.hour_start);
          console.log(parseInt(data.getTime() / 1000));
          console.log(url);
          axios
            .post(url, {
              hour: parseInt(data.getTime() / 1000),
              token: this.hasToken ? this.$route.query.token : ""
            })
            .then(res => {
              console.log(res.data);
              //this.tiquet = res.data;
              //this.tiquet.StoreName = store.name;
              this.$swal({
                type: "success",
                title:
                  "Turn demanat a " + this.hour(this.bucketsSelect.hour_start),
                showConfirmButton: false,
                timer: 2000
              });
            })
            .catch(err => {
              this.$swal("Failako");
            });
        } else {
          this.$swal("selecciona l'hora");
        }
      } else {
        this.$swal("Failako Token");
      }
    },
    showToken() {
      this.$swal(this.$route.query.token);
    }
  }
};
$(function() {
  $("#datetimepicker3").datetimepicker({
    format: "LT"
  });
});
</script>

<style>
hr {
  margin-top: 1rem;
  margin-bottom: 1rem;
  border: 0;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

#token-indicator {
  position: absolute;
  top: 10px;
  right: 10px;
}

.bucketSelect {
  border: solid 3px rgb(52, 132, 236);
}
</style>
