<template>
  <div class="timer-container" @click="toggle()" :diameter="diameter">
    <radial-progress-bar
      :diameter="diameter"
      :stroke-width="thickness"
      :inner-stroke-width="thickness-10"
      :start-color="color"
      :stop-color="color"
      :total-steps="max"
      :completed-steps="value"
      :timing-func="'ease'">
      <span class="timer-center-text" :style="textStyle">{{ remaining }}</span>
    </radial-progress-bar>
    <div class="circle" v-if="this.timer !== null" :style="circleStyle"></div>
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
      default: 50
    },
    color: {
      type: String,
      default: '#ffc400'
    },
    max: {
      type: Number,
      default: 3
    },
    resetTrigger: {
      type: Boolean,
      default: false
    },
    state: {
      type: String,
      default: 'off'
    }
  },
  components: {
    RadialProgressBar
  },
  data() {
    return {
      value: 0,
      timer: null,
      audio: new Audio('alarm.mp3')
    }
  },
  computed: {
    remaining() {
      return this.secondsToTime(this.max - this.value);
    },
    circleStyle() {
      return {
        borderColor: this.color,
        borderWidth: (this.thickness * 2/5) + 'px'
      }
    },
    textStyle() {
      return {
        color: this.color,
        fontSize: (this.thickness*3) + 'px'
      }
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
            this.audio.play();
            this.stop(true);
          }
        }, 1000)
        this.$emit('update:state', 'on')
      } else {
        this.reset();
      }
    },
    stop(finish=false) {
      clearInterval(this.timer);
      this.timer = null;
      this.$emit('update:state', finish ? 'finished' : 'off')
    },
    reset() {
      if (this.timer !== null) {
        this.stop();
      }
      this.$emit('update:state', 'off')
      this.audio.pause();
      this.audio.currentTime = 0;
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
      return `${hours > 0 ? hours+'h ' : (minutes > 0 ? minutes+'m ' : ((seconds>0 || (hours==0 && minutes==0)) ? seconds+'s' : ''))}`
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
.timer-center-text {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-weight: bold;
  white-space: nowrap;
  text-align: center;
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
  height: 108%;
  width: 108%;
  border-radius: 53%;
  border-style: solid;
}

.hidden {
  visibility: hidden;
}
</style>