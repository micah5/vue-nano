<template>
  <div class="center">
    <vs-dialog v-model="active" style="text-align: center;">
      <div style="max-width: 360px;">
        <img src="./static/logo.png" width="50" style="margin-bottom: 10px;" />
        <p style="color: #4A90E2; font-size: small;">
          Waiting for payment ({{ timeLeft }})
        </p>
        <vs-card
          type="3"
          style="width: 250px; margin: 0 auto;"
          @click="refresh"
        >
          <template #img ref="qrcode">
            <canvas id="qr"></canvas>
          </template>
        </vs-card>
        <div ref="loadingSpinner" />
        <div class="metadata">
          <h4>ACCOUNT ADDRESS</h4>
          <p>
            {{ address }}
          </p>

          <h4>AMOUNT</h4>
          <p>
            <span :style="{ color: received > 0 ? '#2C3F51' : '#CCCCCC' }">{{
              received == 0 ? received.toFixed(2) : received
            }}</span>
            / {{ amount }} NANO
          </p>
        </div>
      </div>
    </vs-dialog>
  </div>
</template>

<script>
import Vue from "vue";
import Vuesax from "vuesax";
import "vuesax/dist/vuesax.css";
import QRious from "qrious";
import BigNumber from "bignumber.js";
import Timer from "easytimer.js";
Vue.use(Vuesax, {
  colors: {
    primary: "#4A90E2"
  }
});

export default /*#__PURE__*/ {
  name: "VueNano", // vue component name
  props: {
    value: Boolean,
    time: {
      type: Number,
      default: 60 * 120
    },
    amount: Number,
    checkReceived: Function,
    received: {
      type: Number,
      default: 0.0
    },
    address: String
  },
  data: () => ({
    active: false,
    timeLeft: null
  }),
  mounted: function() {
    const timer = new Timer();
    timer.start({
      precision: "seconds",
      countdown: true,
      startValues: { seconds: this.time }
    });
    timer.addEventListener("secondsUpdated", e => {
      this.timeLeft = timer.getTimeValues().toString();
    });
  },
  methods: {
    loadQrCode: function() {
      const loading = this.$vs.loading({
        target: this.$refs.qrcode
      });
      setTimeout(() => {
        new QRious({
          element: document.getElementById("qr"),
          value: this.genNanoUrl(this.address, this.amount),
          size: 250
        });
        loading.close();
      }, 1000);
    },
    genNanoUrl: function(addr, amount) {
      let x = new BigNumber(amount * 10 ** 30);
      return `nano:${addr}?amount=${x.toFixed()}`;
    },
    refresh: async function() {
      const loading = this.$vs.loading({
        target: this.$refs.qrcode,
        type: "points"
      });
      if (this.checkReceived != null) {
        const res = await this.checkReceived();
        loading.close();
      } else {
        setTimeout(() => {
          loading.close();
        }, 1000);
      }
    }
  },
  watch: {
    active: function(val) {
      if (val) {
        this.loadQrCode();
      }
      this.$emit("input", val);
    },
    value: function(val) {
      this.active = val;
    }
  }
};
</script>

<style scoped>
* {
  font-family: Arial;
}

.metadata {
  font-size: small;
  word-wrap: break-word;
}
</style>
