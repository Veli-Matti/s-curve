<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <div>
      <b-button-group>
        <b-button class="mr-2" @click="reset" variant="outline-secondary">Reset</b-button>
        <b-button class="mr-2" @click="stop" variant="outline-danger">Stop</b-button>
        <b-button @click="start" variant="outline-primary">Start</b-button>
      </b-button-group>
    </div>
    <div class="small">
      <TrendLine :trendData="trendData"></TrendLine>
    </div>
  </div>
</template>

<script>
import TrendLine from "./components/TrendLine.vue";
const periodMs = 100;

export default {
  name: "app",
  components: {
    TrendLine
  },
  data() {
    return {
      trendData: [],
      accConsts: {
        acceleration: 500, // ums^2
        jerk: 100 // acc per adjust cycle
      },
      runtime: {
        startTime: undefined,
        position: 0
      }
    };
  },
  methods: {
    reset() {
      this.trendData = [];
    },
    start() {
      this.stop();
      this.runtime.startTime = new Date();
      this.timerId = setInterval(() => {
        this.trendData.push(this.calculatePosition(20000));
      }, periodMs);
    },
    stop() {
      if (this.timerId) {
        clearInterval(this.timerId);
        this.timerId = null;
      }
    },
    calculatePosition(max) {
      const a = this.accConsts.acceleration; // um/s2
      const j = this.accConsts.jerk;

      const v0 = 0;
      const t = this.deltaTimeMs();
      const p = this.runtime.position;

      const position = v0 * t + (1 / 2) * (a * (t * t));
      const speed = v0 * t + a * t;

      this.position = position;

      console.log(
        `v0 (um/s)=${v0}, t (ms)=${t}, a (um/s^2)=${a}, j (um/s^2)=${j}, position=${position}, speed=${speed}`
      );
      return position;

      /*
      const position = p * t + v * t + (1 / 2) * (a * t) + (1 / 6) * j;
      console.log(`v=${v}, t=${t}, a=${a}, j=${j}, position=${position}`);
*/
      // return Math.floor(Math.random() * Math.floor(max));
    },
    deltaTimeMs() {
      let delta = 0;
      if (this.runtime.startTime) {
        const currentDate = new Date();
        delta = currentDate - this.runtime.startTime;
        delta = delta / 1000;
      }
      return delta;
    }
  },
  timerId: null
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.small {
  max-width: 600px;
  margin: 150px auto;
}
</style>
