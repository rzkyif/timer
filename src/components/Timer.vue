<template>
  <div class="timer-container" @click="toggle()" :diameter="diameter">
    <radial-progress-bar
      :diameter="diameter"
      :stroke-width="thickness"
      :inner-stroke-width="thickness-10"
      :start-color="'#ffd000'"
      :stop-color="'#ffd000'"
      :total-steps="max"
      :completed-steps="value"
      :timing-func="'ease'">
      <span class="timer-center-text">{{ remaining }}</span>
    </radial-progress-bar>
    <div class="circle" v-if="this.timer !== null"></div>
  </div>
</template>

<script>
import RadialProgressBar from 'vue-radial-progress'

export default {
  name: "Timer",
  props: {
    diameter: {
      type: Number,
      default: 400
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
    },
    running: {
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
        this.$emit('update:running', true)
      } else {
        this.reset();
      }
    },
    stop() {
      clearInterval(this.timer);
      this.timer = null;
      this.$emit('update:running', false)
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

.timer-container {
  display: flex;
  position: relative;
  margin: 20px;
}

.timer-container:hover {
  cursor: pointer;
}

.circle {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  height: 106%;
  width: 106%;
  border-radius: 53%;
  border: $primary solid 10px;
}

.hidden {
  visibility: hidden;
}
</style>