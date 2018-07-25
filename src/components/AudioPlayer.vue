 /* eslint-disable */
<template>
  <div id="player">
    <audio id="audio" ref="audio" style="display: none;"></audio>
  
    <el-container>
      <div id="cover" ref="cover"></div>
      <div id="info">
        <div id="title">{{ID3Tag.title}}-{{ID3Tag.band}}</div>
        <div id="subtitle">{{ID3Tag.artist}}</div>
        <div id="controls">
          <el-row>
            <el-col :span="19">
              <el-button id="prev" type="primary" @click="prev()" class="mediaPlayer icon-media-prev" circle></el-button>
              <el-button id="play" type="primary" @click="resume()" v-show="!isPlaying" class="mediaPlayer icon-media-play" circle></el-button>
              <el-button id="pause" type="primary" @click="pause()" v-show="isPlaying" class="mediaPlayer icon-media-pause" circle></el-button>
              <el-button id="next" type="primary" @click="next()" class="mediaPlayer icon-media-next" circle></el-button>
            </el-col>
            <el-col :span="5">
              <el-button id="mute" :type="isMute()?'info':'primary'" @click="mute()" :class="['mediaPlayer',isMute()?'icon-media-mute':'icon-media-unmute']" circle></el-button>
            </el-col>
          </el-row>
  
        </div>
      </div>
  
    </el-container>
    <div>
      <div id="timeline">
        <el-row :gutter="10"  > 
          <el-col :span="3" class="txt_curtime"> {{sec_to_time(currentTime)}} </el-col>  
          <el-col :span="18">
            <el-slider v-model="currentTime" :max="duration" :format-tooltip="formatTootip"></el-slider>
          </el-col>
          <el-col :span="3" class="txt_duration"> {{sec_to_time(duration)}} </el-col>
        </el-row>
      </div>
      <div>
        <ul id="list">
          <li v-for="(item,idx) in list" :key="idx" :class="{current:isCurrent(idx),fail:isFail(item)}">
            {{ (idx+1) +". "+item }}
          </li>
        </ul>
      </div>
  
    </div>
  
  </div>
</template>

<script>
import Vue from "vue";
import { Main, Container, Button, Slider, Col, Row } from "element-ui";
import { parse } from "id3-parser";
import { fetchFileAsBuffer } from "id3-parser/lib/universal/helpers";
import "../assets/icon/iconfont.css";

Vue.use(Container);
Vue.use(Button);
Vue.use(Slider);
Vue.use(Col);
Vue.use(Row);

export default {
  props: {
    list: {
      default: function() {
        return [];
      }
    }
  },
  data() {
    return {
      duration: 300,
      state: "play",
      currentIndex: 0,
      currentTime: 0,
      audioElement: null,
      ID3Tag: {},
      fails: new Set()
    };
  },
  computed: {
    isPlaying() {
      return this.audioElement && this.state == "play";
    }
  },
  watch: {
    currentTime(newValue, oldValue) {
      if (Math.abs(newValue - oldValue) > 0.9) {
        this.audioElement.currentTime = newValue;
      }
    },
    currentIndex(newValue, oldValue) {
      if (newValue != oldValue) {
        this.play();
      }
    }
  },
  methods: {
    isMute() {
      return this.audioElement && this.audioElement.volume == 0;
    },
    isCurrent(idx) {
      return this.currentIndex == idx;
    },
    isFail(filename) {
      return this.fails.has(filename);
    },
    formatTootip(v) {
      return this.sec_to_time(v);
    },
    sec_to_time(s) {
      var t = "";
      if (s > -1) {
        var hour = Math.floor(s / 3600);
        var min = Math.floor(s / 60) % 60;
        var sec = Math.floor(s % 60);
        if (hour >= 1) {
          if (hour < 10) {
            t = "0" + hour + ":";
          } else {
            t = hour + ":";
          }
        }
        if (min < 10) {
          t += "0";
        }
        t += min + ":";
        if (sec < 10) {
          t += "0";
        }
        t += sec;
      }
      return t;
    },
    play() {
      const filename = this.list[this.currentIndex];
      const url = "media/" + encodeURI(filename);
      fetchFileAsBuffer(url)
        .then(parse)
        .then(tag => {
          if (tag.image) {
            var base64String = "";
            for (var i = 0; i < tag.image.data.length; i++) {
              base64String += String.fromCharCode(tag.image.data[i]);
            }
            var dataUrl =
              "data:" + tag.image.mime + ";base64," + window.btoa(base64String);

            this.$refs["cover"].style.backgroundImage = `url('${dataUrl}')`;
          } else {
            this.$refs[
              "cover"
            ].style.backgroundImage = `url('/img/recorder.jpg')`;
          }

          this.ID3Tag = tag;
          this.fails.delete(filename);
        });
      this.ID3Tag = {
        title: filename
      };
      this.audioElement.src = url;
      this.audioElement.play().catch(error => {
        console.log(error);
        this.fails.add(filename);
        setTimeout(this.next, 1000);
      });
    },
    resume() {
      this.audioElement.play();
    },
    pause() {
      this.audioElement.pause();
    },
    prev() {
      if (this.currentIndex > 0) this.currentIndex--;
    },
    next() {
      if (this.currentIndex < this.list.length - 1) {
        this.currentIndex++;
      } else {
        this.currentIndex = 0;
      }
    },
    mute() {
      if (this.audioElement) {
        if (this.audioElement.volume > 0) {
          this.audioElement.volume = 0;
        } else {
          this.audioElement.volume = 1;
        }
      }
    },
    reset() {
      this.currentIndex = 0;
      this.play();
    }
  },
  mounted() {
    this.audioElement = this.$refs.audio;

    if (this.audioElement) {
      this.audioElement.addEventListener("play", () => {
        this.state = "play";
      });
      this.audioElement.addEventListener("pause", () => {
        this.state = "pause";
      });
      this.audioElement.addEventListener("loadedmetadata", () => {
        this.duration = this.audioElement.duration;
      });
      this.audioElement.addEventListener("ended", this.next);
      this.audioElement.addEventListener("timeupdate", () => {
        this.currentTime = this.audioElement.currentTime;
      });
      this.reset();
    }
  }
};
</script>

<style scoped lang="less">
div {
  /* border: 1px dashed darkgray; */
}

.current {
  font-weight: bold;
  background-color: #f0f8ff;
}

#player {
  width: 500px;
  border: 4px solid #f0f0f0;
  padding: 10px;
  #cover {
    width: 40%;
    height: 200px;
    box-shadow: 3px 3px 10px 2px gray;
    background-repeat: no-repeat;
    background-size: cover;
    border-radius: 10px;
    background-image: url("/img/recorder.jpg");
  }
  #info {
    width: 60%;
    position: relative;
    #title {
      font-size: 30px;
      font-weight: bold;
      padding-left: 30px;
      line-height: 50px;
      text-align: left;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
      text-shadow: 2px 2px 10px gray;
    }
    #subtitle {
      font-size: 25px;
      padding-left: 30px;
      line-height: 50px;
      text-align: left;
      text-shadow: 2px 2px 10px gray;
    }
    #controls {
      position: absolute;
      bottom: 0px;
      width: 300px;
      button {
        box-shadow: 2px 2px 10px 1px #aaa;
      }
    }
  }
  #timeline {
    line-height: 38px;
    margin: 10px 0px;
    box-shadow: 1px 1px 5px 1px #e8e8e8;
    .txt_curtime {
      float: left;
    }
    .txt_duration {
      float: right;
    }
  }
  #list {
    box-shadow: 1px 1px 5px 1px #e2e2e2;
    padding: 0px;
    li {
      padding: 15px;
      text-align-last: left;
      list-style: none;
      color: #429efd;
      & + li {
        border-top: dashed 1px #f0f0f0;
      }
    }

    .fail {
      text-decoration: line-through;
      color: rgb(168, 168, 168);
    }
  }
}
</style>
