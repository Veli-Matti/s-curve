<template>
  <b-container id="app">
    <b-row align-h="center">
      <b-col cols="6">
        <!--
        <img alt="Vue logo" src="./assets/logo.png" />
        -->
        <b-jumbotron variant="info" header-tag="h3">
          <template v-slot:header>
            {{runtimeTimeTxt}}
          </template>
          <template v-slot:lead>
            {{runtimePosTxt}}
          </template>
        </b-jumbotron>

        <div>
          <b-input-group prepend="Initial speed" append="um/s">
            <b-form-input v-model.number="accConsts.v0"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Acceleration" append="um/s^2">
            <b-form-input v-model.number="accConsts.acceleration"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Target" append="um">
            <b-form-input v-model.number="accConsts.targetPos"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Jerk" append="%">
            <b-form-input v-model.number="accConsts.jerk"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Period" append="ms">
            <b-form-input v-model.number="accConsts.period"></b-form-input>
          </b-input-group>
          <b-form-radio-group class="mt-2" v-model="accConsts.phase">
            <b-form-radio v-model="accConsts.phase" value="concave">Concave</b-form-radio>
            <b-form-radio v-model="accConsts.phase" value="linear">Linear</b-form-radio>
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
    <b-row class="mt-5" align-h="center">
      <b-col>
        <TrendLine :trendData="trendData"></TrendLine>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import TrendLine from "./components/TrendLine.vue";
const CONCAVE = "concave";
const LINEAR = "linear";
const CONVEX = "convex";

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
        jerk: 25, // percentage from max
        phase: "linear",
        period: 100,
        targetPos: 2000 // um
      },
      runtime: {
        startTime: undefined,
        position: 0,
        time: 0
      },
      timerId: null
    };
  },
  computed: {
    isActive() {
      return this.timerId ? true : false;
    },
    runtimeTimeTxt () {
      const timeTxt = parseFloat(this.runtime.time).toFixed(3)
      return `${timeTxt} s`
    },
    runtimePosTxt () {
      const dataTxt = parseFloat(this.runtime.position).toFixed(1)
      return `${dataTxt} um`
    }

  },
  methods: {
    reset() {
      this.trendData = [];
      this.runtime.position = 0
      this.runtime.time = 0
    },
    start() {
      this.stop();
      this.runtime.startTime = new Date();
      this.timerId = setInterval(() => {
        const pos = this.calculatePosition();
        if (pos >= this.accConsts.targetPos) {
          this.stop()
        } else {
          this.trendData.push(pos);
          this.runtime.position = pos;
          this.runtime.time = this.deltaTimeMs();
        }
      }, this.accConsts.period);
    },
    stop() {
      if (this.timerId) {
        clearInterval(this.timerId);
        this.timerId = null;
      }
    },
    calculatePosition() {
      let retVal;
      if (this.accConsts.phase === CONCAVE) {
        retVal = this.calculatePositionConcave();
      } else if (this.accConsts.phase === LINEAR) {
        retVal = this.calculatePositionlinear();
      } else if (this.accConsts.phase === CONVEX) {
        retVal = this.calculatePositionConvex();
      } else {
        // Just for testing purpose
        retVal = Math.floor(Math.random() * Math.floor(20000));
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
    calculatePositionConcave() {
      // TODO. Calculation
      return 1000;
    },
    calculatePositionlinear() {
      const a = this.accConsts.acceleration;
      const j = this.accConsts.jerk;

      const v0 = 0;
      const t = this.deltaTimeMs();
      const p = this.runtime.position;

      const position = v0 * t + (1 / 2) * (a * (t * t));
      const speed = v0 * t + a * t;

      // this.position = position;

      console.log(
        `v0 (um/s)=${v0}, t (ms)=${t}, a (um/s^2)=${a}, j (um/s^2)=${j}, position=${position}, speed=${speed}`
      );
      return position;
    },
    calculatePositionConvex() {
      // TODO. Calculation
      return 1900;
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
