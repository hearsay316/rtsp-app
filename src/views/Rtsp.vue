<template>
  <div class="rtsp">
    <div class="rtsp-item">
      <video ref="rtspItem" id="videoElement" muted></video>
    </div>
  </div>
</template>

<script>
import flvjs from "flv.js";
export default {
  name: "Rtsp",
  props: {
    Url: {
      type: String
    }
  },
  beforeCreate() {
    if (flvjs.isSupported()) this.flvjs = flvjs;
  },
  mounted() {
    this.initData();
  },
  methods: {
    initData() {
      this.flvPlayer = this.flvjs.createPlayer({
        type: "flv",
        cors: "no-cors",
        isLive: true, // 如果是直播流需要设置这个值为 true
        // url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test"
        url: "http://img.ksbbs.com/asset/Mon_1704/15868902d399b87.flv"
        // ↑ 拉流示例地址，stream参数一定要和推流时所设置的流密钥一致
      });
      this.flvPlayer.attachMediaElement(this.$refs.rtspItem);
      this.flvPlayer.load();
      this.flvPlayer.play();
    },
    flvDestroy() {
      // 关闭
      this.flvPlayer.pause();
      this.flvPlayer.unload();
      this.flvPlayer.detachMediaElement();
      this.flvPlayer.destroy();
      this.flvPlayer = null;
    }
  }
};
</script>

<style scoped>
.rtsp-item {
  width: 400px;
  height: 200px;
  border: 1px solid red;
}
</style>
