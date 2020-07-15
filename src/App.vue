<template>
  <v-app>
    <v-main>
      <v-container>
        <v-row class="text-center">
          <v-col
            cols="12"
            class="text-h1 font-weight-black"
            :class="{ 'red--text': isComplete }"
          >
            {{ min }}:{{ sec }}
          </v-col>
          <v-col cols="12">
            <v-btn
              v-if="!playing"
              fab
              large
              color="primary"
              @click="startTimer"
            >
              <v-icon>mdi-play</v-icon>
            </v-btn>
            <v-btn v-if="playing" fab large @click="stopTimer">
              <v-icon dense>mdi-stop</v-icon>
            </v-btn>
          </v-col>
          <v-col cols="12">
            <v-btn fab small @click="resetTimer">
              <v-icon dense>mdi-replay</v-icon>
            </v-btn>
          </v-col>
        </v-row>
        <v-row class="mt-5" justify="center">
          <v-col cols="10" sm="6" class="pa-0">
            <v-slider
              v-model="minTime"
              min="0"
              max="60"
              label="分"
              dense
              thumb-label
              :thumb-size="24"
            ></v-slider>
          </v-col>
        </v-row>
        <v-row justify="center">
          <v-col cols="10" sm="6" class="pa-0">
            <v-slider
              v-model="secTime"
              min="0"
              max="60"
              label="秒"
              dense
              thumb-label
              :thumb-size="24"
            ></v-slider>
          </v-col>
        </v-row>
        <v-row justify="center">
          <v-switch
            v-model="alartSound"
            label="アラーム音"
            class="ma-0"
          ></v-switch>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import completeAlertSound from "./mixin/completeAlertSound";
export default {
  mixins: [completeAlertSound],
  data() {
    return {
      timerIntervalId: null,
      soundIntervalId: null,
      totalSec: null,
      minTime: null,
      secTime: null,
      playing: false,
      alartSound: true,
      isComplete: false,
    };
  },
  computed: {
    // 分表示
    min() {
      const min = Math.floor(this.totalSec / 60);
      return this.zeroPadding(min);
    },
    // 秒表示
    sec() {
      const sec = this.totalSec - this.min * 60;
      return this.zeroPadding(sec);
    },
  },
  watch: {
    minTime() {
      // リセットする
      if (this.isComplete) this.resetTimer();
      this.totalSec = this.secTime + this.minTime * 60;
    },
    secTime() {
      // リセットする
      if (this.isComplete) this.resetTimer();
      this.totalSec = this.secTime + this.minTime * 60;
    },
  },
  mounted() {
    // localStorageから以前設定した値を取得/設定
    if (localStorage.totalSec) {
      this.minTime = Math.floor(localStorage.totalSec / 60);
      this.secTime = localStorage.totalSec - this.minTime * 60;
    }
    // pagetitle設定
    document.title = "シンプルタイマー";
  },
  methods: {
    startTimer() {
      if (!this.totalSec) return;
      // ローカルストレージに保存
      localStorage.totalSec = this.totalSec;
      // 再生中フラグON
      this.playing = true;
      // 終了フラグOFF
      this.isComplete = false;
      // タイマー開始
      this.timerIntervalId = setInterval(() => {
        if (this.totalSec >= 1) {
          this.totalSec--;
        } else {
          this.resetTimer();
          this.isComplete = true;
          if (this.alartSound) this.soundIntervalId = this.soundPlay();
        }
      }, 1000);
    },
    stopTimer() {
      clearInterval(this.timerIntervalId);
      this.playing = false;
      this.timerIntervalId = null;
    },
    resetTimer() {
      clearInterval(this.timerIntervalId);
      clearInterval(this.soundIntervalId);
      this.playing = false;
      this.totalSec = null;
      this.isComplete = false;
    },
    zeroPadding(val) {
      return ("0" + val).slice(-2);
    },
  },
};
</script>
<style lang="stylus">
.container
  position: absolute
  top: 25%
  right: 0
  bottom: 0
  left: 0
.v-slider
  cursor: pointer !important
.v-slider__thumb
  cursor: grabbing
</style>
