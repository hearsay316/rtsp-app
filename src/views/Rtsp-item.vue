<template>
  <div
    class="Rtsp-item-video"
    :class="isFull ? 'Rtsp-item-video-full' : 'Rtsp-item-video'"
  >
    <video class="rtsp-item" ref="rtspItem" id="videoElement" muted></video>
    <div class="control">
      <div class="control-main">
        <div @click="refresh"><span>刷新</span></div>
        <div @click="controlFull">
          <span>{{ isFull ? "还原" : "全屏" }}</span>
        </div>
        <div @click="playClose"><span>关闭</span></div>
        <!--      <li class="monitor-name">{{ name }}</li>-->
<!--        <slot name="pushMe" :hasError="hasError" />-->
      </div>

      <!--      <li v-if="hasError" title="刷新" @click="refresh">-->
      <!--        <i class="el-icon-refresh" />-->
      <!--      </li>-->
      <!--      <li-->
      <!--        v-if="!hasError || isFull !== '全屏'"-->
      <!--        :title="isFull"-->
      <!--        @click="playFull"-->
      <!--      >-->
      <!--        <span :class="isFull == '全屏' ? 'isFull' : 'notFull'" />-->
      <!--      </li>-->
      <!--      <li v-if="width !== 600" title="关闭" @click="playClose">-->
      <!--        <i class="el-icon-circle-close" />-->
      <!--      </li>-->
    </div>
  </div>
</template>

<script>
// import flvjs from "flv.js";
export default {
  name: "Rtsp",
  data() {
    return {
      isFull: false
    };
  },
  props: {
    Url: {
      type: String
    }
  },
  beforeCreate() {
    //isSupported
    console.log(window.flvjs)
    if (window.flvjs) this.flvjs = window.flvjs;
  },
  mounted() {
    // this.initData();
  },
  methods: {
    initData() {
      this.flvPlayer = this.flvjs.createPlayer({
        type: "flv",
        cors: "no-cors",
        isLive: true, // 如果是直播流需要设置这个值为 true
        url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test"
        // url: "http://img.ksbbs.com/asset/Mon_1704/15868902d399b87.flv"
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
    },
    refresh() {
      console.log(666);
      this.initData();
    },
    playClose() {
      this.flvDestroy();
    },
    controlFull() {
      this.isFull ? (this.isFull = false) : (this.isFull = true);
    }
  }
};
</script>

<style scoped>
.rtsp-item {
  width: 100%;
  height: 100%;
  border: 1px solid red;
}
.Rtsp-item-video {
  width: 400px;
  height: 200px;
  border: 1px solid red;
  position: relative;
}
video {
  object-fit: fill;
}
.control {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 28px;
  background: rgba(0, 0, 0, 0.55);
  display: flex;
  justify-content: flex-end;
  align-items: center;
  z-index: 2;
}
.control-main {
  display: flex;
  width: 150px;
  justify-content: space-around;
  color: aliceblue;
}
.control-main div {
  cursor: pointer;
}
.Rtsp-item-video-full {
  position: absolute;
  width: 100%;
  height: 100%;
  background: red;
  z-index: 3;
}
ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
</style>
