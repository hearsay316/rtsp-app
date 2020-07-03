<template>
  <div class="hello">
    <div id="dplayer" ref="dplayer"></div>
  </div>
</template>

<script>
import axios from "axios";
import Qs from "qs";

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
        _this.dp = new window.DPlayer({
          container: _this.$refs.dplayer,
          live: true,
          autoplay: true,
          mutex: false,
          video: {
            url: event.streams[0]
          }
        });
      };
      // eslint-disable-next-line no-unused-vars
      this.pc.oniceconnectionstatechange = e => {
        console.log(this.pc.iceConnectionState);
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
          this.sendChannel.onmessage = e => {
            console.log(
              `Message from DataChannel '${this.sendChannel.label}' payload '${e.data}'`
            );
          };
        }
      });
    },
    getRemoteSdp() {
      const data = Qs.stringify({
        suuid: "demo1",
        data: btoa(this.pc.localDescription.sdp)
      });
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
