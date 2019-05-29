<template>
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
                    min="8:00"
                    max="23:00"
                    required
                    v-model="time"
                    pattern="[0-9]{2}:[0-9]{2}"
                  >
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Cerrar</button>
                  <button type="button" class="btn btn-primary" data-dismiss="modal" @click="hourTurn(store)">Reserva</button>
                </div>
              </div>
            </div>
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
      storeModal: {},
      time: null,
      tiquet: null,
      
    };
  },

  computed: {
    hasToken() {
      return typeof this.$route.query.token !== 'undefined' && this.$route.query.token !== '';
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
        urls.routes.turn;

      axios
        .post(url,{
            token: (this.hasToken ? this.$route.query.token : '')
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
    },
    hourTurn(store) {
      const url =
        urls.host +
        urls.routes.prefix +
        urls.routes.store +
        "/" +
        store.id +
        urls.routes.hourTurn;
      if (this.time) {
        var hoursMinutes = this.time.split(":");

        var date = new Date();
        date.setHours(hoursMinutes[0]);
        date.setMinutes(hoursMinutes[1]);
        console.log(date);
        console.log(parseInt(date.getTime() / 1000));
        console.log(url);
        axios
          .post(url, {
            hour: parseInt(date.getTime() / 1000),
            token: (this.hasToken ? this.$route.query.token : '')
          })
          .then(res => {
            this.tiquet = res.data;
            this.tiquet.StoreName = store.name;
            console.log(res)
            console.log(this.tiquet);
            this.$swal({
              type: "success",
              title: "Turn demanat a " + this.time,
              showConfirmButton: false,
              timer: 2000
            });
            //this.$swal('Imprimir tiquet' + JSON.stringify(res.data));
          })
          .catch(err => {
            this.$swal("Failako");
          });
      } else {
        this.$swal("selecciona l'hora");
      }
    },
    showToken() {
      this.$swal(this.$route.query.token)
    }
  }
};
$(function() {
  $("#datetimepicker3").datetimepicker({
    format: "LT"
  });
});
</script>