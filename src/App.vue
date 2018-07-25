<template>
  <div id="app">
    <AudioPlayer :list="list" />
  </div>
</template>

<script>
import AudioPlayer from "./components/AudioPlayer";

export default {
  name: "App",
  data() {
    return {
      list: ["lxdyy.mp3", "jay.mp3", "刘若英-为爱痴狂.mp3"]
    };
  },
  components: {
    AudioPlayer
  }
};
</script>

<style>
html,
body {
  margin: 0px;
  padding: 0px;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px;
}
</style>
