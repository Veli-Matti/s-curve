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
      timeMs: 0
    };
  },
  methods: {
    reset() {
      this.trendData = [];
    },
    start() {
      console.log("start");
      this.stop();
      this.timerId = setInterval(() => {
        this.timeMs = this.timeMs + periodMs;
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
      return Math.floor(Math.random() * Math.floor(max));
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
