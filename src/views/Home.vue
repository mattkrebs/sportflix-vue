<template>
  <v-container fluid>
    <v-row align="start" justify="start">
      <v-col cols="8">
        <video :src="videoSource" ref="videoPlayer" id="video" autoplay></video>
        <v-list>
          <v-list-item v-for="device in deviceList" :key="device.id">
            {{ device.label }}
          </v-list-item>
        </v-list>
      </v-col>
      <v-col cols="3">
        <v-row>
          <v-col cols="12">
            <canvas id="myCanvas" />
          </v-col>
        </v-row>
        <v-row align="start" justify="start">
          <v-col>
          <v-btn large color="primary" ref="stream" @click="startStream">Start Stream</v-btn>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>
<style scoped>
.scoreboard div {
  width: 50%;
  float: left;
  background-color: transparent;
  margin: 0;
}
</style>
<script>
//import Vue from "vue";
import { exec }  from 'child_process';
const fs = require('fs');
//import html2canvas from "html2canvas";


export default {
  name: "VideoPlayer",
  data: () => ({
    videoSource: [],
    myvar: "hello",
    scoreboard: {
      home :{
        color: "red",
        score: 0,
        name: "Home 1"
      },
      away:{
        color:"blue",
        score: 0,
        name: "Away 1"
      }
    },
    banner: require("../assets/scoreboard.png"),
    deviceList: [],
  }),
  mounted() {
  //  let vm = this;
    this.createScoreboard();
  },
  created() {
    var deviceName = this.findSourceByName("USB Video");
    console.log(deviceName);
    this.displayVideo(deviceName);
    //this.streamVideo();
   
   
  },
  methods: {
    checkVideoSource: function() {},
    findSourceByName: function(name) {
      navigator.mediaDevices.enumerateDevices().then(function(devices) {
        return devices.filter((device) => device.label === name).id;
      });

      return "";
    },
    startStream(){
      this.saveCanvas();
      this.stopCapture();
      this.streamVideo();
    },
    saveCanvas(){
       var canvas = document.querySelector("#myCanvas");
        const url = canvas.toDataURL("image/png", 1);

      // remove Base64 stuff from the Image
      const base64Data = url.replace(/^data:image\/png;base64,/, "");
      fs.writeFile(
        "./src/assets/scoreboard.png",
        base64Data,
        "base64",
        function(err) {
          console.log(err);
        }
      );
    },
    createScoreboard() {
      var canvas = document.querySelector("#myCanvas");
      var ctx = canvas.getContext("2d");
      //left score
      ctx.globalAlpha= 0.2
      ctx.font = "14px Arial";
      ctx.fillStyle= this.scoreboard.home.color;
      ctx.fillRect(20,0,100,40);
      ctx.beginPath();
      ctx.arc(20, 20, 20, 0.5 * Math.PI, 1.5 * Math.PI);
      ctx.fillStyle = this.scoreboard.home.color;
      ctx.fill();
      ctx.globalAlpha= 1
      ctx.fillStyle = "white";
      ctx.fillText(this.scoreboard.home.score, 10, 25);
       ctx.fillText(this.scoreboard.home.name, 50, 25);

       //right score
      ctx.globalAlpha= 0.2
      ctx.font = "14px Arial";
      ctx.fillStyle=this.scoreboard.away.color;
      ctx.fillRect(120,0,100,40);
      ctx.beginPath();
      ctx.arc(220, 20, 20, 1.5 * Math.PI, 0.5 * Math.PI);
      ctx.fillStyle = this.scoreboard.away.color;
      ctx.fill();
      ctx.globalAlpha= 1
      ctx.fillStyle = "white";
      ctx.fillText(this.scoreboard.away.score, 210, 25);
      ctx.fillText(this.scoreboard.away.name, 130, 25);
      
      
    },
    stopCapture(){
      let tracks = this.$refs.videoPlayer.srcObject.getTracks();

      tracks.forEach(track => track.stop());
      this.$refs.videoPlayer.srcObject = null;
    },
    streamVideo: function() {
    
      var out = exec('ffmpeg -f dshow -i video="USB Video" -f flv rtmp://192.168.1.196:1935/stream/mystream');
      console.log(out);
    },
    displayVideo: function(device) {
      let vm = this;
      navigator.mediaDevices
        .getUserMedia({ video: { deviceId: device } })
        .then(function(stream) {
          console.log(stream);
          vm.$refs.videoPlayer.srcObject = stream;
          vm.$refs.videoPlayer.play();
        })
        .catch(function(err) {
          /* handle the error */

          console.log(err);
        });
    },
  },

  computed: {},
};

//
</script>
