<template>
  <div :class="isFull ? 'Rtsp-item-video-full' : 'Rtsp-item-video'" :style="RtspStyle">
    <div class="rtsp-item">
      <video
        v-if="!isError"
        class="rtsp-item"
        ref="rtspItem"
        id="videoElement"
        muted
      ></video>
      <div class="rtsp-item-error" v-else>{{ isErrorTitle }}</div>
    </div>
    <div class="control">
      <div class="control-left">{{ name }}</div>
      <div class="control-right">
        <div @click="refresh"><span>刷新</span></div>
        <div @click="controlFull">
          <span>{{ isFull ? "还原" : "全屏" }}</span>
        </div>
        <div @click="playClose"><span>关闭</span></div>
        <!--        <slot name="pushMe" :hasError="hasError" />-->
      </div>
    </div>
  </div>
</template>

<script>
import flvjs from "flv.js";
export default {
  name: "Rtsp",
  data() {
    return {
      isFull: false,
      isError: false,
      isErrorTitle: "",
      name: "这个是大桥的视频",
      loading: false
    };
  },
  props: {
    url: {
      type: String
    },
    width: {
      type: String,
      default: "800"
    },
    height: {
      type: String,
      default: "400"
    }
  },
  computed: {
    RtspStyle() {
      if (this.isFull) {
        /*
        *   position: absolute;
  width: 100%;
  height: 100%;
  background: red;
  z-index: 3;*/
        return {
          position: "absolute",
          width: "100%",
          height: "100%",
          background: "red",
          zIndex: "3"
        };
      } else {
        return {
          width: `${this.width}px`,
          height: `${this.height}px`
        };
      }
    }
  },
  beforeCreate() {
    //isSupported
    console.log(window.flvjs);
    if (window.flvjs || typeof flvjs !== "undefined")
      this.flvjs = window.flvjs || flvjs;
  },
  mounted() {
    this.initData();
  },
  methods: {
    initData() {
      this.loading = true;
      this.isError = false;
      this.isErrorTitle = " ";
      console.log(this.$refs.rtspItem);
      this.flvPlayer = this.flvjs.createPlayer({
        type: "flv",
        cors: "no-cors",
        isLive: true, // 如果是直播流需要设置这个值为 true
        url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test"
        // url: "http://img.ksbbs.com/asset/Mon_1704/15868902d399b87.flv"
        // ↑ 拉流示例地址，stream参数一定要和推流时所设置的流密钥一致
      });
      console.log(this.flvPlayer.Events, this.flvPlayer);
      // this.flvPlayer.Events(err => {
      //   console.log(err, 6666666666666666666);
      // });
      this.flvPlayer.on(this.flvjs.Events.ERROR, (errType, errDetail) => {
        // errType是 NetworkError时，对应errDetail有：Exception、HttpStatusCodeInvalid、ConnectingTimeout、EarlyEof、UnrecoverableEarlyEof
        // errType是 MediaError时，对应errDetail是MediaMSEError
        console.log(errType, errDetail, 6666);
        switch (errType) {
          case "NetworkError":
            this.isError = true;
            this.isErrorTitle = "网络错误";
            console.log("网络错误");
            break;
          case "MediaError":
            this.isError = true;
            this.isErrorTitle = "与媒体有关的错误（格式错误，解码问题等）";
            console.log("与媒体有关的错误（格式错误，解码问题等）");
            break;
          case "OtherError":
            this.isError = true;
            this.isErrorTitle = "其他错误";
            console.log("其他错误");
            break;
        }
      });
      this.flvPlayer.on(this.flvjs.Events.LOADING_COMPLETE, res => {
        console.log(res);
        this.loading = false;
      });
      try {
        this.flvPlayer.attachMediaElement(this.$refs.rtspItem);
        this.flvPlayer.load();
        this.flvPlayer.play();
      } catch (e) {
        console.log(e, 888);
      }
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
      this.isError = false;
      this.$nextTick(() => {
        this.initData();
      });
    },
    playClose() {
      this.isFull = false
      this.isError = true;
      this.isErrorTitle = "视频被关闭了";
      this.flvDestroy();
    },
    controlFull() {
      // if(!this.isError){
        this.isFull ? (this.isFull = false) : (this.isFull = true);
      // }

    }
  }
};
</script>

<style scoped>
.rtsp-item {
  width: 100%;
  height: 100%;
}
.Rtsp-item-video {
  border: 1px solid red;
  position: relative;
}
.Rtsp-item-video:nth-child(2n) {
  border-left: 0;
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
  justify-content: space-between;
  align-items: center;
  z-index: 2;
}
.control-left {
  display: flex;
  height: 28px;
  line-height: 28px;
  width: calc(400px - 150px);
  padding-left: 6px;
  font-size: 14px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.control-right {
  display: flex;
  width: 150px;
  height: 28px;
  line-height: 28px;
  justify-content: space-around;
  color: aliceblue;
}
.control-right div {
  cursor: pointer;
}
.Rtsp-item-video-full {
}
.rtsp-item-error {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
