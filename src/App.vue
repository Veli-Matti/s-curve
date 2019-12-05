<template>
  <b-container id="app">
    <b-row align-h="center">
      <b-col cols="6">
        <!--
        <img alt="Vue logo" src="./assets/logo.png" />
        -->
        <b-jumbotron header-level="4">
          <template v-slot:header>
            {{runtimeTimeTxt}}
          </template>
          <template v-slot:lead>
            <b-table-simple stacked small>
              <b-tbody>
                <b-tr>
                  <b-td stacked-heading="Position"
                    :class="{'text-danger': runtime.stopReason==='position'}">
                    {{runtimePosTxt}}
                  </b-td>
                </b-tr>
                <b-tr>
                  <b-td stacked-heading="Speed"
                    :class="{'text-danger': runtime.stopReason==='speed'}">
                    {{runtimeSpeedTxt}}
                  </b-td>
                </b-tr>
                <b-tr>
                  <b-td stacked-heading="Avg.spd">{{runtimeAverageSpeedTxt}}</b-td>
                </b-tr>
              </b-tbody>
            </b-table-simple>
          </template>
        </b-jumbotron>

        <div>
          <b-input-group append="um/s">
            <template v-slot:prepend>
              <b-input-group-text class="bg-light">
                Initial speed (convex)
              </b-input-group-text>
            </template>
            <b-form-input v-model.number="accConsts.convexV0"></b-form-input>
          </b-input-group>
          <b-input-group append="s">
            <template v-slot:prepend>
              <b-input-group-text class="bg-light">
                Initial time (convex)
              </b-input-group-text>
            </template>
            <b-form-input v-model.number="accConsts.convexInitTime"></b-form-input>
          </b-input-group>
        </div>

        <div class="mt-4">
          <b-input-group prepend="Target speed" append="um/s">
            <b-form-input v-model.number="accConsts.targetSpeed"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Max acceleration" append="um/s^2">
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
const POSITION = "position"
const SPEED = "speed"

class runtimeResult {
  constructor(time, speed, position) {
    this.time = time;
    this.speed = speed;
    this.position = position;
  }
}

export default {
  name: "app",
  components: {
    TrendLine
  },
  data() {
    return {
      trendData: [],
      accConsts: {
        convexV0: 500, // um/s
        convexInitTime: 1, // s
        acceleration: 500, // ums^2
        jerk: 25, // percentage from max
        phase: CONVEX,
        period: 100, // ms
        targetPos: 2000, // um
        targetSpeed: 1000 // ums/s
      },
      runtime: {
        startTime: undefined,
        position: 0,
        speed: 0,
        time: 0,
        stopReason: ""
      },
      timerId: null
    };
  },
  computed: {
    isActive() {
      return this.timerId ? true : false;
    },
    jerk () {
      return this.accConsts.acceleration * (this.accConsts.jerk / 100)
    },
    runtimeTimeTxt () {
      const timeTxt = parseFloat(this.runtime.time).toFixed(3)
      return `${timeTxt} s`
    },
    runtimePosTxt () {
      const dataTxt = parseFloat(this.runtime.position).toFixed(3)
      return `${dataTxt} um`
    },
    runtimeSpeedTxt() {
      let dataTxt = parseFloat(this.runtime.speed).toFixed(3)
      return `${dataTxt} um/s`
    },
    runtimeAverageSpeedTxt () {
      let data
      if (this.runtime.time) {
        data = parseFloat(this.runtime.position/this.runtime.time)
      } else {
        data = parseFloat(0)
      }
      const dataTxt = parseFloat(data).toFixed(3)
      return `${dataTxt} um/s`
    }
  },
  methods: {
    reset() {
      this.trendData = [];
      this.runtime.position = 0
      this.runtime.time = 0
      this.runtime.speed = 0
      this.runtime.stopReason = ""
    },
    start() {
      this.stop();
      this.runtime.stopReason = ""
      this.runtime.startTime = new Date();
      this.timerId = setInterval(() => {
        const runtimeObj = this.resolveRuntime();
        const pos = runtimeObj.position
        // Stop when we have reached
        // ... position or speed limits
        let stopReason
        if (pos >= this.accConsts.targetPos || pos <= 0) {
          stopReason = POSITION
        } else if (runtimeObj.speed >= this.accConsts.targetSpeed) {
          stopReason = SPEED
        }
        if (stopReason) { // This works for acceleration only
          this.stop()
          this.runtime.stopReason = stopReason
        } else {
          this.trendData.push(pos);
          this.runtime.position = pos;
          this.runtime.speed = runtimeObj.speed;
          this.runtime.time = runtimeObj.time;
        }
      }, this.accConsts.period);
    },
    stop() {
      if (this.timerId) {
        clearInterval(this.timerId);
        this.timerId = null;
      }
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
    resolveRuntime() {
      let retVal;
      const timeStamp = this.deltaTimeMs();
      if (this.accConsts.phase === CONCAVE) {
        retVal = this.resolveRuntimeConcave(timeStamp);
      } else if (this.accConsts.phase === LINEAR) {
        retVal = this.resolveRuntimeLinear(timeStamp);
      } else if (this.accConsts.phase === CONVEX) {
        retVal = this.resolveRuntimeConvex(timeStamp);
      } else {
        retVal = this.resolveRuntimeRandom(timeStamp)
      }
      return retVal;
    },
    resolveRuntimeConcave(t) {
      const v0 = 0;
      const delta = this.resolveRuntimeconvXXDelta(t)

      const position = (v0 * t) + delta
      const speed = v0 + (this.jerk*(t*t)/2);

      return new runtimeResult(t, speed, position)
    },
    resolveRuntimeLinear(t) {
      const a = this.accConsts.acceleration;

      const v0 = 0;
      const p = this.runtime.position;

      const position = v0 * t + (1 / 2) * (a * (t * t));
      const speed = (v0 * t) + (a * t);
      return new runtimeResult(t, speed, position);
    },
    resolveRuntimeConvex(t) {
      const position = 900;
      const speed = 10;
      const a = this.accConsts.acceleration;
      const v0 = this.accConsts.convexV0;
      const tmpPos = v0*t + a*(t*t)/2 - this.resolveRuntimeconvXXDelta(t)
      const tmpSpeed = v0 + a*t - this.jerk*(t*t)/2
      console.log(`a=${a}, v0=${v0}, tmpPos=${tmpPos}, tmpSpeed=${tmpSpeed}`);
      return new runtimeResult(t, tmpSpeed, tmpPos);
    },
    resolveRuntimeconvXXDelta(t) {
      const j = this.jerk;
      return j * (t * t * t) / 6;
    },
    resolveRuntimeRandom(t) {
      const position = Math.floor(Math.random() * Math.floor(this.accConsts.targetPos));
      return new runtimeResult(t, 0, position)
    }
  }
}
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

.em {
  font-style: italic;
}

</style>
