<template>
<div>
  <div class="flex-r">
    <input type="text" placeholder="ip?" v-model="ip">
    <button @click="connect">
      connect
    </button>
  </div>
  <div >
    <select class="selected-device" @change="selectCum($event)" name="cums">
      <option v-for="(cum, index) in cums" :key="index">{{cum.label}}</option>
    </select>
  </div>
  <video autoplay class="video"> </video>
  <!-- <vue-webrtc ref="webrtc" width="100%" roomId="sample-room">
  </vue-webrtc> -->
</div>

</template>

<script>
export default {
  
  data(){
    return{
      ip:'',
      cums:[],
      selectedDevice:[],
      video:HTMLVideoElement,
      videoSource: HTMLSelectElement,
      srcVideo:MediaStream
    }
  },
  methods:{
    connect(){
      if(this.ip.length){
        this.srcVideo = this.video.srcObject;
        const peer = new RTCPeerConnection({
          iceServers:[
            {urls:this.ip}
          ]
        })
        this.srcVideo.getTracks().forEach(track => peer.addTrack(track, this.srcVideo))
        .then(function() {
          return peer.createAnswer();
        })
        console.log(peer);
      }else{
        console.error("ip empty?")
      }
    },
    selectCum(event){
      this.selectedDevice = this.cums.filter(cum => cum.label === event.target.value)
      this.getPerm();
    },
    callbackVideos(stream){
      this.video.srcObject = stream
      this.video.play();
    },
    getPerm(){
      navigator.mediaDevices.getUserMedia({
        video: {
          deviceId: this.selectedDevice[0]?.deviceId ? {exact: this.selectedDevice[0]?.deviceId} : undefined
        },
        audio: true
        })
        .then(this.callbackVideos)
        .then(navigator.mediaDevices.enumerateDevices().then(devices => {
          this.cums = devices.filter(cums => cums.kind === 'videoinput')}))
        .catch(error => {console.log('navigator.MediaDevices.getUserMedia error: ', error.message, error.name)})
    },

  },
  mounted(){
    this.video = document.querySelector("video");
    this.getPerm()

  }
}
</script>
<style scoped>

.flex-r{
  display: flex;
}
.video{
  width: 50%;
}
.selected-device{
  width: 13%;
  height: 30px;
}
</style>
