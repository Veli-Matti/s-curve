
<script>
import Trend from "vuetrend";

export default {
  name: "TrendLine",
  props: {
    trendData: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  components: {
    Trend
  }
};
</script>

<template>
  <div>
    <trend
      :data="trendData"
      :gradient="['#6fa8dc', '#42b983', '#2c3e50']"
      auto-draw
      autoDrawDuration="1000"
    ></trend>
  </div>
</template>