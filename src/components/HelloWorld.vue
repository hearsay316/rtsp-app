<template>
  <div class="hello">
    <button @click="startSession()">Start Session</button>
    <div id="remoteVideos"></div>
    <div id="div"></div>
    <div id="dplayer" ref="dplayer"></div>
  </div>
</template>

<script>
import axios from "axios";
let log = msg => {
  document.getElementById("div").innerHTML += msg + "<br>";
};
// let sendChannel = null;
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      config: {
        iceServers: [
          {
            urls: ["stun:stun.l.google.com:19302"]
          }
        ]
      },
      sendChannel: null,
      pc: null
    };
  },
  async mounted() {
    await this.$nextTick();
    this.init();
    await this.$nextTick();
    this.getCodecInfo();
  },
  destroyed() {
    clearInterval(this.time);
  },
  methods: {
    async init() {
      const _this = this;
      this.pc = new RTCPeerConnection(this.config);
      this.pc.onnegotiationneeded = this.handleNegotiationNeededEvent;

      this.pc.ontrack = function(event) {
        console.log(event.track.kind);
        log(event.streams.length + " track is delivered");
        _this.dp = new window.DPlayer({
          container: _this.$refs.dplayer,
          live: true,
          autoplay: true,
          mutex: false,
          video: {
            url: event.streams[0]
          }
        });
        console.log(_this.dp, 1111111111);
        // var el = document.createElement(event.track.kind);
        // console.log(typeof event.streams[0],'6666666666666')
        // el.srcObject = event.streams[0];
        // el.muted = true;
        // el.autoplay = true;
        // el.controls = true;
        // el.width = 600;
        // document.getElementById("remoteVideos").appendChild(el);
      };
      // eslint-disable-next-line no-unused-vars
      this.pc.oniceconnectionstatechange = e => {
        log(this.pc.iceConnectionState);
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
      window.$.get("http://127.0.0.1:8083/codec/demo1", data => {
        try {
          console.log(data);
          // data = JSON.parse(data);
          if (data.length > 1) {
            log("add audio Transceiver");
            this.pc.addTransceiver("audio", {
              direction: "sendrecv"
            });
          }
        } catch (e) {
          console.log(e);
        } finally {
          log("add video Transceiver");
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
              console.log("ping");
            }, 1000);
          };
          this.sendChannel.onmessage = e =>
            log(
              `Message from DataChannel '${this.sendChannel.label}' payload '${e.data}'`
            );
        }
      });
    },
    getRemoteSdp() {
      var formData = new FormData();
      // var fileField = document.querySelector("input[type='file']");

      formData.append("suuid", "demo1");
      formData.append("data", btoa(this.pc.localDescription.sdp));
      axios.defaults.headers.post["Content-Type"] =
        "application/x-www-form-urlencoded";
      // "http://127.0.0.1:8083/recive",
      window.$.post(
        "http://127.0.0.1:8083/recive",
        {
          suuid: "demo1",
          data: btoa(this.pc.localDescription.sdp)
        },
        data => {
          console.log(typeof data);
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
        }
      );
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus">
h3
  margin 40px 0 0

ul
  list-style-type none
  padding 0

li
  display inline-block
  margin 0 10px

a
  color #42b983
#dplayer
  width 320px
</style>
