<template>
  <b-container id="app">
    <b-row align-h="center">
      <b-col cols="4">
        <img alt="Vue logo" src="./assets/logo.png" />
        <div>
          <b-input-group prepend="Initial speed" append="um/s">
            <b-form-input v-model.number="accConsts.v0"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Acceleration" append="um/s^2">
            <b-form-input v-model.number="accConsts.acceleration"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Jerk" append="um/s^2">
            <b-form-input v-model.number="accConsts.jerk"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Period" append="ms">
            <b-form-input v-model.number="accConsts.period"></b-form-input>
          </b-input-group>
          <b-form-radio-group class="mt-2" v-model="accConsts.phase">
            <b-form-radio v-model="accConsts.phase" value="convey">Convey</b-form-radio>
            <b-form-radio v-model="accConsts.phase" value="steady">Steady</b-form-radio>
            <b-form-radio v-model="accConsts.phase" value="convex">Convex</b-form-radio>
          </b-form-radio-group>
          <b-form-radio class="em" v-model="accConsts.phase" value="random">Random (for testing)</b-form-radio>
          <b-button-group class="mt-3">
            <b-button class="mr-2" @click="reset" variant="outline-secondary">Reset</b-button>
            <b-button class="mr-2" @click="stop" variant="outline-danger" :disabled="!isActive">Stop</b-button>
            <b-button @click="start" variant="outline-primary" :disabled="isActive">Start</b-button>
          </b-button-group>
        </div>
      </b-col>
    </b-row>
    <b-row align-h="center">
      <b-col cols="8">
        <div class="small">
          <TrendLine :trendData="trendData"></TrendLine>
        </div>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import TrendLine from "./components/TrendLine.vue";
const CONVEY = "convey";
const CONVEX = "convex";
const STEADY = "steady";

export default {
  name: "app",
  components: {
    TrendLine
  },
  data() {
    return {
      trendData: [],
      accConsts: {
        v0: 0,
        acceleration: 500, // ums^2
        jerk: 100, // acc per adjust cycle
        phase: "steady",
        period: 100
      },
      runtime: {
        startTime: undefined,
        position: 0
      },
      timerId: null
    };
  },
  computed: {
    isActive() {
      return this.timerId ? true : false;
    }
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
      }, this.accConsts.period);
    },
    stop() {
      if (this.timerId) {
        clearInterval(this.timerId);
        this.timerId = null;
      }
    },
    calculatePosition(max) {
      let retVal;
      if (this.accConsts.phase === CONVEY) {
        retVal = this.calculatePositionConvey();
      } else if (this.accConsts.phase === STEADY) {
        retVal = this.calculatePositionSteady();
      } else if (this.accConsts.phase === CONVEX) {
        retVal = this.calculatePositionConvex();
      } else {
        // Just for testign purpose
        retVal = Math.floor(Math.random() * Math.floor(max));
      }
      return retVal;
    },
    deltaTimeMs() {
      let delta = 0;
      if (this.runtime.startTime) {
        const currentDate = new Date();
        delta = currentDate - this.runtime.startTime;
        delta = delta / 1000;
      }
      return delta;
    },
    calculatePositionConvey() {
      // TODO. Calculation
      return 1000;
    },
    calculatePositionSteady() {
      const a = this.accConsts.acceleration;
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
    },
    calculatePositionConvex() {
      // TODO. Calculation
      return 3000;
    }
  }
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

.em {
  font-style: italic;
}
</style>
