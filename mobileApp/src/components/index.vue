<template>
  <div class="py-3 px-3">
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
                  <input
                    id="appt-time"
                    type="time"
                    name="appt-time"
                    min="12:00"
                    max="18:00"
                    required
                    pattern="[0-9]{2}:[0-9]{2}"
                  >
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Cerrar</button>
                  <button type="button" class="btn btn-primary">Reserva</button>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="col-6 mt-2">
          <button class="btn btn-primary">Turn Ahora</button>
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

export default {
  components: {},
  data() {
    return {
      stores: [],
      urls: urls,
      showTurnsModal: false,
      storeModal: {}
    };
  },

  mounted() {
    const url = urls.host + urls.routes.prefix + urls.routes.stores;
    axios
      .get(url)
      .then(res => {
        this.stores = res.data;
      })
      .catch(err => {
        console.log(err);
      });
  },
  methods: {
    showTurnsListModal(store) {
      this.storeModal = store;
      this.showTurnsModal = true;
    },
    closeModal() {
      this.showTurnsModal = false;
    },
    normalTurn(store) {
      const url =
        urls.host +
        urls.routes.prefix +
        urls.routes.store +
        "/" +
        store.id +
        "/turn";

      axios
        .post(url)
        .then(res => {
          this.$swal({
            type: "success",
            title: "Turn demanat A" + res.data.number,
            showConfirmButton: false,
            timer: 2000
          });
          console.log("Torn demanat A" + res.data.number);
          //this.$swal('Imprimir tiquet' + JSON.stringify(res.data));
        })
        .catch(err => {
          this.$swal("Failako");
        });
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
</style>
