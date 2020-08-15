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
              v-if="!playing && !isComplete"
              fab
              large
              color="primary"
              @click="startTimer"
            >
              <v-icon>mdi-play</v-icon>
            </v-btn>
            <v-btn
              v-if="playing && !isComplete"
              fab
              large
              class="white--text"
              color="secondary"
              @click="stopTimer"
            >
              <v-icon dense>mdi-stop</v-icon>
            </v-btn>
            <v-btn
              v-if="isComplete"
              fab
              large
              color="error"
              @click="completeTimer"
            >
              <v-icon dense>mdi-check-bold</v-icon>
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
          <v-col cols="10" sm="6" class="pa-0">
            <v-btn-toggle
              borderless
              color="primary"
              class="d-flex justify-center"
            >
              <v-btn @click="setMinTime(1)">
                1分
              </v-btn>
              <v-btn @click="setMinTime(3)">
                3分
              </v-btn>
              <v-btn @click="setMinTime(5)">
                5分
              </v-btn>
              <v-btn @click="setMinTime(10)">
                10分
              </v-btn>
              <v-btn @click="setMinTime(15)">
                15分
              </v-btn>
            </v-btn-toggle>
          </v-col>
        </v-row>
        <v-row class="mt-5" justify="center">
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
      alartSound: false,
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
      this.changeSlider();
    },
    secTime() {
      this.changeSlider();
    },
  },
  mounted() {
    this.initData();
    // pagetitle設定
    document.title = "シンプルタイマー";
  },
  methods: {
    startTimer() {
      if (!this.totalSec) return;
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
      this.minTime = null;
      this.secTime = null;
      this.isComplete = false;
    },
    completeTimer() {
      clearInterval(this.timerIntervalId);
      clearInterval(this.soundIntervalId);
      this.playing = false;
      this.totalSec = null;
      this.isComplete = false;
      // startTimer実行時の値を代入
      this.initData();
    },
    zeroPadding(val) {
      return ("0" + val).slice(-2);
    },
    initData() {
      // localStorageから以前設定した値を取得/設定
      if (sessionStorage.getItem("totalSec")) {
        this.minTime = Math.floor(sessionStorage.getItem("totalSec") / 60);
        this.secTime = sessionStorage.getItem("totalSec") - this.minTime * 60;
      }
    },
    // スライダーの変化時の共通処理、watch監視している
    changeSlider() {
      this.totalSec = this.secTime + this.minTime * 60;
      // ローカルストレージに設定した時間を保存
      sessionStorage.setItem("totalSec", this.totalSec);
      // 完了後、スライドを変化させた時は完了フラグをfalseに
      if (this.totalSec !== 0) this.isComplete = false;
    },
    // 1~15分ボタン
    setMinTime(val) {
      this.minTime = val;
      this.secTime = null;
    },
  },
};
</script>
<style lang="stylus">
.container
  position: absolute
  top: 20%
  right: 0
  bottom: 0
  left: 0
.v-slider
  cursor: pointer !important
.v-slider__thumb
  cursor: grabbing
</style>
