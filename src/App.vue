<template>
  <v-app>
    <v-main>
      <v-container>
        <v-row class="text-center">
          <v-col cols="12" class="text-h1 font-weight-black">
            {{ min }}:{{ sec }}
          </v-col>
          <v-col cols="12">
            <v-btn v-if="!playing" fab large @click="startTimer">
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
          <v-col cols="10" sm="6" class="pa-0">
            <v-switch label="アラーム音"></v-switch>
          </v-col>
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
      intervalId: null,
      totalSec: null,
      minTime: null,
      secTime: null,
      playing: false,
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
      this.totalSec = this.secTime + this.minTime * 60;
    },
    secTime() {
      this.totalSec = this.secTime + this.minTime * 60;
    },
  },
  methods: {
    startTimer() {
      if (!this.totalSec) return;
      this.playing = true;
      this.intervalId = setInterval(() => {
        if (this.totalSec >= 1) {
          this.totalSec--;
        } else {
          this.resetTimer();
          this.soundPlay();
        }
      }, 1000);
    },
    stopTimer() {
      clearInterval(this.intervalId);
      this.playing = false;
      this.intervalId = null;
    },
    resetTimer() {
      clearInterval(this.intervalId);
      this.playing = false;
      this.totalSec = null;
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
