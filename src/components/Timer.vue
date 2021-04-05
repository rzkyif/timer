<template>
  <div class="timer-container" @click="toggle()">
    <radial-progress-bar
      :diameter="diameter"
      :stroke-width="thickness"
      :inner-stroke-width="thickness-10"
      :start-color="'#ffd000'"
      :stop-color="'#ffd000'"
      :total-steps="max"
      :completed-steps="value"
      :timing-func="'ease'">
      <div class="circle" v-if="this.timer !== null"></div>
      <span class="timer-center-text">{{ remaining }}</span>
    </radial-progress-bar>
  </div>
</template>

<script>
import RadialProgressBar from 'vue-radial-progress'

export default {
  name: "Timer",
  props: {
    diameter: {
      type: Number,
      default: 600
    },
    thickness: {
      type: Number,
      default: 25
    },
    max: {
      type: Number,
      default: 3
    },
    resetTrigger: {
      type: Boolean,
      default: false
    }
  },
  components: {
    RadialProgressBar
  },
  data() {
    return {
      value: 0,
      timer: null,
    }
  },
  computed: {
    remaining() {
      return this.secondsToTime(this.max - this.value);
    }
  },
  watch: {
    resetTrigger: function() {
      this.reset()
    }
  },
  methods: {
    start() {
      if (this.value < this.max) {
        this.timer = setInterval(()=>{
          if (this.value < this.max) {
            this.value += 1;
          } else {
            this.stop();
          }
        }, 1000)
      } else {
        this.reset();
      }
    },
    stop() {
      clearInterval(this.timer);
      this.timer = null;
    },
    reset() {
      if (this.timer !== null) {
        this.stop();
      }
      this.value = 0;
    },
    toggle() {
      if (this.timer === null) {
        this.start();
      } else {
        this.stop();
      }
    },
    secondsToTime(totalSeconds) {
      let hours = Math.floor(totalSeconds / 3600);
      let minutes = Math.floor(totalSeconds / 60) % 60;
      let seconds = totalSeconds % 60;
      return `${hours > 0 ? hours+'h ' : ''}${minutes > 0 ? minutes+'m ' : ''}${(seconds>0 || (hours==0 && minutes==0)) ? seconds+'s' : ''}`
    },
  },
  destroyed() {
    if (this.timer !== null) {
      clearInterval(this.timer);
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../scss/index.scss';

.timer-center-text {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-weight: bold;
  font-size: 80px;
  white-space: nowrap;
  text-align: center;
  color: $primary;
  user-select: none;
}

.timer-container:hover {
  cursor: pointer;
}

$circleWidth: 625px;
.circle {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  height: $circleWidth;
  width: $circleWidth;
  border-radius: $circleWidth / 2;
  border: $primary solid 10px;
}

.hidden {
  visibility: hidden;
}
</style>