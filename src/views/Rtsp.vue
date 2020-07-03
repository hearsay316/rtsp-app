<template>
  <div>
    <div class="rtsp">
      <RtspItem v-for="item in list" :key="item"></RtspItem>
    </div>
    <div class="handleSwitch" @click="handleSwitch">切换</div>
  </div>
</template>

<script>
import RtspItem from "./Rtsp-itme.vue";
export default {
  name: "Rtsp",
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
      this.list === 4 ? (this.list = 8) : (this.list = 4);
    }
  }
};
</script>

<style scoped>
.rtsp {
  display: grid;
  justify-content: center;
  grid-template-columns: 400px 400px;
  justify-items: center;
}
.handleSwitch {

  cursor: pointer;
}
</style>
