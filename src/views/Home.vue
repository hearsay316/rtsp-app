<template>
  <div class="home">
    <Rtsp :listData="listData" width="800" height="400"></Rtsp>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from "@/components/HelloWorld.vue";
import Rtsp from "./Rtsp";
import axios from "axios";
import Qs from "qs";
export default {
  name: "Home",
  data() {
    return {
      listData: [
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        },
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        },
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        },
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        },
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        },
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        },
        {
          url: "http://47.103.47.160:8080/live?port=1985&app=live&stream=test",
          name: "这个是大桥视频"
        }
      ]
    };
  },
  components: {
    // HelloWorld,
    Rtsp
  },
  async mounted() {
    // await this.$nextTick();
    // this.init();
    // await this.$nextTick();
    // this.getCodecInfo();
  },
  methods: {
    async init() {
      this.pc = new RTCPeerConnection(this.config);
      this.pc.onnegotiationneeded = this.handleNegotiationNeededEvent;
      const _this = this;
      this.pc.ontrack = function(event) {
        // el.srcObject = event.streams[0];
        // el.muted = true;
        // el.autoplay = true;
        // el.controls = true;
        // el.width = 600;
        // document.getElementById("remoteVideos").appendChild(el);
        try {
          _this.dp = new window.DPlayer({
            container: document.getElementById("dplayer1"),
            live: true,
            autoplay: true,
            mutex: false,
            video: {
              url: event.streams[0]
            }
          });
        } catch (e) {
          console.log(e);
          console.log(
            document.querySelector(".dplayer-video .dplayer-video-current")
          );
        }
        console.log(_this.dp, "ddddddd");
        // _this.dp.container.querySelector('video').srcObject = event.streams[0]
        // console.log();
      };
      // eslint-disable-next-line no-unused-vars
      this.pc.oniceconnectionstatechange = e => {
        console.log(this.pc.iceConnectionState, e);
        // if(this.pc.iceConnectionState==='disconnected'){
        //   this.getCodecInfo()
        // }
      };
    },
    async handleNegotiationNeededEvent() {
      let offer = await this.pc.createOffer();
      await this.pc.setLocalDescription(offer);
      this.getRemoteSdp();
    },
    getCodecInfo() {
      axios({
        method: "get",
        url: `http://127.0.0.1:8083/codec/demo1`,
        headers: {
          "Content-type": "application/x-www-form-urlencoded"
        }
      }).then(({ data }) => {
        try {
          // data = JSON.parse(data);
          if (data.length > 1) {
            console.log("add audio Transceiver");
            this.pc.addTransceiver("audio", {
              direction: "sendrecv"
            });
          }
        } catch (e) {
          console.log(e);
        } finally {
          console.log("add video Transceiver");
          this.pc.addTransceiver("video", {
            direction: "sendrecv"
          });
          //send ping becouse PION not handle RTCSessionDescription.close()
          this.sendChannel = this.pc.createDataChannel("foof");
          this.sendChannel.onclose = () =>
            console.log("sendChannel has closed");
          this.sendChannel.onopen = () => {
            console.log("sendChannel has opened");
            this.sendChannel.send("ping");
            this.time = setInterval(() => {
              this.sendChannel.send("ping");
            }, 1000);
          };
          this.sendChannel.onmessage = e =>
            console.log(
              `Message from DataChannel '${this.sendChannel.label}' payload '${e.data}'`
            );
        }
      });
    },
    getRemoteSdp() {
      const data = Qs.stringify({
        suuid: "demo1",
        data: btoa(this.pc.localDescription.sdp)
      });
      // "http://127.0.0.1:8083/recive",
      axios({
        method: "post",
        url: `http://127.0.0.1:8083/recive`,
        headers: {
          "Content-type": "application/x-www-form-urlencoded"
        },
        data
      }).then(({ data }) => {
        try {
          this.pc.setRemoteDescription(
            new RTCSessionDescription({
              type: "answer",
              sdp: atob(data)
            })
          );
        } catch (e) {
          console.warn(e);
        }
      });
    }
  }
};
</script>
<style scoped>
#dplayer1 {
  width: 200px;
}
</style>
