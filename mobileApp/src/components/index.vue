<template>
  <div class="py-3 px-3">
    <div id="token-indicator">
      <span v-if="hasToken" @click="showToken">✅</span>
      <span v-else>❌</span>
    </div>

    <div>
      <span>VIP</span>
      <input type="checkbox" v-model="vip" name="isVip" id>
    </div>
    <!--div class="row" v-if="tiquet">
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
    </div-->
    <template v-for="store in storesAndTiquets">
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
                  <div v-for="(bucket,index) in bucketsList" :key="index">
                    <li
                      :class="{ bucketSelect: bucket ==  bucketsSelect}"
                      class="list-group-item m-0 list-group-item-secondary"
                      v-if="bucket.filled"
                    >{{hour(bucket.hour_start)}}</li>
                    <li
                      :class="{ bucketSelect: bucket ==  bucketsSelect}"
                      class="list-group-item m-0"
                      v-else
                      @click="selectBucket(bucket)"
                    >{{hour(bucket.hour_start)}}</li>
                  </div>
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

        <!-- pedir turn-->
        <div v-if="!store.tiquet" class="col-12">
          <div class="row footer">
            <div class="col-4 mt-2">
              <button
                type="button"
                class="btn btn-warning"
                data-toggle="modal"
                data-target="#exampleModal"
                @click="getNextBuckets(store)"
              >Reserva</button>
            </div>
            <div class="col-4 mt-2">
              <button v-if="vip" class="btn btn-success" @click="vipTurn(store)">Turn Vip</button>
            </div>
            <div class="col-4 mt-2">
              <button class="btn btn-primary" @click="normalTurn(store)">Turn Ahora</button>
            </div>
          </div>
        </div>

        <!-- mostrar tiquet-->
        <div v-else class="col-12">
          <div class="row footer">
            <div class="col-12 p-2">{{turnLeter(store.tiquet)}}</div>
          </div>
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
      tiquets: null,
      bucketsList: null,
      bucketsSelect: null,
      vip: false,
      storesAndTiquets: null
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
    this.getTiquetOfToken();
  },
  methods: {
    getStores() {
      const url = urls.host + urls.routes.prefix + urls.routes.stores;
      axios
        .get(url)
        .then(res => {
          this.stores = res.data;

          this.stores.forEach(store => {
            this.tiquets.forEach(tiquet => {
              if (store.name == tiquet.name) {
                store.tiquet = tiquet;
              }
            });
          });
          this.storesAndTiquets = this.stores;
          console.log(this.stores);
        })
        .catch(err => {
          console.log(err);
        });
    },
    getTiquetOfToken() {
      const url = urls.host + urls.routes.prefix + urls.routes.tokenTurns;
      console.log(url);
      console.log(this.$route.query.token);

      axios
        .post(url, {
          token: this.hasToken ? this.$route.query.token : ""
        })
        .then(res => {
          console.log(res.data);
          this.tiquets = res.data;
          this.getStores();
          //this.$swal('Imprimir tiquet' + JSON.stringify(res.data));
        })
        .catch(err => {
          this.$swal("Failako");
        });
    },
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
            console.log(this.tiquet);
            this.$swal({
              type: "success",
              title: "Turn demanat T" + res.data.turn.number,
              showConfirmButton: false,
              timer: 1500
            });
            this.getTiquetOfToken();

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
    vipTurn(store) {
      const url =
        urls.host +
        urls.routes.prefix +
        urls.routes.store +
        "/" +
        store.id +
        urls.routes.vipTurn;
      console.log(url);
      if (this.hasToken) {
        axios
          .post(url, {
            token: this.hasToken ? this.$route.query.token : ""
          })
          .then(res => {
            console.log(res);
            this.getTiquetOfToken();

            /*  this.$swal({
              type: "success",
              title: "Turn demanat V" + res.data.turn.number,
              showConfirmButton: false,
              timer: 1500
            });
            console.log("Torn demanat V" + res.data.turn.number);*/
            //this.$swal('Imprimir tiquet' + JSON.stringify(res.data));
          })
          .catch(err => {
            console.log(err);
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
              this.getTiquetOfToken();
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
    },
    turnLeter: function(turn) {
      if (turn.number != null) {
        if (turn.number.toString().length == 1) {
          turn.number = "0" + turn.number.toString();
        }
      }
      switch (turn.type) {
        case "normal":
          return "T" + turn.number;
          break;
        case "hour":
          if (turn.number) {
            return "A" + turn.number;
          } else {
            return "unsing";
          }
          break;
        case "vip":
          return "V" + turn.number;
          break;
      }
    }
  }
};
$(function() {
  $("#datetimepicker3").datetimepicker({
    format: "LT"
  });
});
</script>

<style scoped>
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
li {
  box-sizing: border-box;
}
</style>
