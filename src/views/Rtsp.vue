<template>
  <div class="main">
    <div class="rtsp" :style="{width:`${width}px`,height:`${height}px`,gridTemplateColumns:`${width/2}px ${width/2}px`}">
      <RtspItem v-for="item in listData" :width="`${width/2}`" :height="`${height/2}`" :key="item"></RtspItem>
    </div>
    <div class="handleSwitch" @click="handleSwitch">切换</div>
  </div>
</template>

<script>
import RtspItem from "./Rtsp-item.vue";
export default {
  name: "Rtsp",
  props: {
    listData: {
      type: Array
    },
    width: {
      type: String,
      default: '800'
    },
    height:{
      type:String,
      default: "400"
    }
  },
  components: {
    RtspItem
  },
  data() {
    return {
      list: 4
    };
  },
  methods: {
    handleSwitch() {
      this.list === 4 ? (this.list = 12) : (this.list = 4);
    }
  }
};
</script>

<style scoped>
.main {
  text-align: center;
}
.rtsp {
  position: relative;
  display: inline-grid;
  justify-content: center;
  justify-items: center;
}
.handleSwitch {
  cursor: pointer;
}
</style>
