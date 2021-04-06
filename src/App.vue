<template>
  <div id="app">
    <b-modal :width="'fit-content'" v-model="isAddModalOpen">
      <div class="card">
        <header class="card-header">
          <p class="card-header-title">
            Add a Timer
          </p>
        </header>
        <div class="card-content">
          <b-field label="Name">
              <b-input value="Timer" v-model="newTimer.name"></b-input>
          </b-field>
          <b-field label="Color" :style="{position: 'relative'}">
            <div class="circle" :style="{backgroundColor: newTimer.color}"></div>
            <verte value="#ffc400" id="color-picker" display="widget" :show-history="false" model="hex" @input="(color) => newTimer.color = color"></verte>
          </b-field>
          <b-field label="Hours">
            <b-numberinput value="0" min="0" max="99" v-model="newTimer.hours"></b-numberinput>
          </b-field>
          <b-field label="Minutes">
            <b-numberinput value="0" min="0" max="59" v-model="newTimer.minutes"></b-numberinput>
          </b-field>
          <b-field label="Seconds">
            <b-numberinput value="0" min="0" max="59" v-model="newTimer.seconds"></b-numberinput>
          </b-field>
        </div>
        <div class="card-footer">
          <a class="card-footer-item" @click="closeAddModal">Close</a>
          <a class="card-footer-item" @click="addTimer">Add</a>
        </div>
      </div>
    </b-modal>
    <div id="sidebar" v-if="showKegiatan">
      <b-table id="activity-table"
        :data="tableData"
        :columns="tableColumns"
        :selected.sync="selectedData"
        height="95vh"
        sticky-header
        :scrollable="false"
        :mobile-cards="false">
      </b-table>
      <div class="load-save-container">
        <label class="button is-text is-fullwidth" for="f" id="l">Load</label>
        <a id="save" class="button is-text is-fullwidth" @click="saveJSON">Save</a>
      </div>
    </div>
    <div id="content">
      <b-tabs id="timer-tabs" type="is-boxed" v-model="selectedTimer">
        <b-tab-item v-for="timer in timers" :key="timer.name" :visible="timer.visible">
          <template #header>
            <b-icon 
              v-if="timer.state == 'on'"
              pack="fas"
              icon="spinner"
              custom-class="fa-pulse">
            </b-icon>
            <b-icon 
              v-if="timer.state == 'finished'"
              pack="fas"
              icon="exclamation">
            </b-icon>
            <span>{{timer.name}}</span>
          </template>
          <Timer :max="timer.time" :resetTrigger="timer.resetTrigger" :diameter="windowMin*0.6" :thickness="windowMin*0.05" :state.sync="timer.state" :color="timer.color"></Timer>
          <b-button class="reset-button" @click="timer.resetTrigger = !timer.resetTrigger">Reset</b-button>
        </b-tab-item>
      </b-tabs>
      <div class="modify-timer">
        <b-button type="is-text" @click="openAddModal">Add Timer</b-button>
        <b-button type="is-text" @click="deleteTimer">Delete Timer</b-button>
      </div>
      <div v-if="windowDimensions[1] > 1000">
        <b-button type="is-text" @click="showKegiatan = !showKegiatan" id="solo-load-button">{{showKegiatan ? 'Hide' : 'Show'}} Kegiatan</b-button>
      </div>
    </div>
    <input id="f" type="file" @change="loadFunction"/>
  </div>
</template>

<script>
import Timer from './components/Timer.vue'
import verte from 'verte'
import 'verte/dist/verte.css';

export default {
  name: 'App',
  components: {
    Timer,
    verte
  },
  data() {
    return {
      tableData: [
        {id: 1, event: "Kegiatan A", displayTime: "", time: 10},
        {id: 2, event: "Kegiatan B", displayTime: "", time: 5},
      ],
      tableColumns: [
        {
          field: 'id',
          label: 'No.',
          width: 40,
          numeric: true,
        },
        {
          field: 'event',
          label: 'Kegiatan',
          width: 300,
        },
        {
          field: 'displayTime',
          label: 'Waktu',
          width: 160
        },
      ],
      timers: [
        {
          name: 'Kegiatan',
          time: 30,
          state: 'off',
          resetTrigger: false,
          color: '#24dcdc',
          visible: false,
        },
        {
          name: '30',
          time: 30,
          state: 'off',
          resetTrigger: false,
          color: '#ffc400',
          visible: true,
        },
        {
          name: '20',
          time: 20,
          state: 'off',
          resetTrigger: false,
          color: '#ffc400',
          visible: true,
        },
        {
          name: '15',
          time: 15,
          state: 'off',
          resetTrigger: false,
          color: '#ffc400',
          visible: true,
        },
        {
          name: '10',
          time: 10,
          state: 'off',
          resetTrigger: false,
          color: '#ffc400',
          visible: true,
        },
        {
          name: '5',
          time: 5,
          state: 'off',
          resetTrigger: false,
          color: '#ffc400',
          visible: true,
        }
      ],
      newTimer: {
        name: '',
        hours: 0,
        minutes: 0,
        seconds: 0,
        color: '#ffc400'
      },
      selectedData: null,
      selectedTimer: 1,
      isAddModalOpen: false,
      showKegiatan: false,
      windowMin: Math.min(window.innerHeight, window.innerWidth),
      windowDimensions: [window.innerHeight, window.innerWidth]
    }
  },
  mounted() {
    this.selectedData = this.tableData[0];
    this.processDataTime();
    window.addEventListener('resize', this.onResize)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    onResize() {
      this.windowMin = Math.min(window.innerHeight, window.innerWidth);
      this.windowDimensions = [window.innerHeight, window.innerWidth];
    },
    processDataTime() {
      for (const row of this.tableData) {
        row.displayTime = this.secondsToTime(row.time)
      }
    },
    secondsToTime(totalSeconds) {
      let hours = Math.floor(totalSeconds / 3600);
      let minutes = Math.floor(totalSeconds / 60) % 60;
      let seconds = totalSeconds % 60;
      return `${hours > 0 ? hours+'h ' : ''}${minutes > 0 ? minutes+'m ' : ''}${(seconds>0 || (hours==0 && minutes==0)) ? seconds+'s' : ''}`
    },
    timeToSeconds(time) {
      let clean = time.replace(' ', '');
      let s1 = clean.includes('h') ? clean.split('h') : ['0', clean];
      let hours = parseInt(s1[0]);
      let s2 = s1[1].includes('m') ? s1[1].split('m') : ['0', s1[1]];
      let minutes = parseInt(s2[0])
      let s3 = s2[1].includes('s') ? s2[1].split('s') : ['0', s2[1]];
      let seconds = parseInt(s3[0]);
      return hours*3600 + minutes*60 + seconds;
    },
    resetAddModal() {
      this.newTimer.name = "";
      this.newTimer.hours = 0;
      this.newTimer.minutes = 0;
      this.newTimer.seconds = 0;
    },
    openAddModal() {
      this.isAddModalOpen = true;
    },
    closeAddModal() {
      this.isAddModalOpen = false;
    },
    addTimer() {
      let time = this.newTimer.hours*3600 + this.newTimer.minutes*60 + this.newTimer.seconds;
      if (this.newTimer.name !== '' && time > 0) {
        this.timers.push({
          name: this.newTimer.name,
          time,
          state: 'off',
          resetTrigger: false,
          color: this.newTimer.color,
          visible: true,
        });
      }
      this.closeAddModal();
      this.resetAddModal();
    },
    deleteTimer() {
      if (this.selectedTimer > 0) {
        this.timers.splice(this.selectedTimer, 1);
        this.selectedTimer--;
      }
    },
    loadFunction() {
      let loadFile = document.getElementById("f");
      loadFile.files[0].text().then((text) => {
        this.loadJSON(text);
      })
    },
    loadJSON(jsonString) {
      let events = JSON.parse(jsonString);
      this.tableData = [];
      let id = 1;
      for (const event of events) {
        this.tableData.push({
          id,
          event: event.name,
          displayTime: '',
          time: this.timeToSeconds(event.duration),
        });
        id++;
      }
      this.selectedData = this.tableData[0];
      this.processDataTime();
    },
    saveJSON() {
      let events = []
      for (const event of this.tableData) {
        events.push({
          name: event.event,
          duration: this.secondsToTime(event.time)
        });
      }
      var json = JSON.stringify(events, null, 2);

      let dataUri = "data:application/json;charset=utf-8," + encodeURIComponent(json);
      let defaultName = 'activity_data.json';
      let element = document.getElementById("save");

      element.href = dataUri;
      element.download = defaultName;
    }
  },
  watch: {
    selectedData: function() {
      this.timers[0].time = this.selectedData.time;
      this.timers[0].resetTrigger = !this.timers[0].resetTrigger;
    },
    showKegiatan: function() {
      this.timers[0].visible = this.showKegiatan;
      if (!this.showKegiatan && this.selectedTimer == 0 && this.timers.length > 1) {
        this.selectedTimer = 1;
      }
    }
  }
}
</script>

<style lang="scss">
$primary: #ffc400;
@import "~bulma/sass/elements/button.sass";

$primary: #24dcdc;
$link: #4a4a4a;
@import "~bulma/sass/utilities/_all";
@import "~bulma";
@import "~buefy/src/scss/buefy";

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  display: flex;
  flex-direction: row;
}
#sidebar {
  display: flex;
  flex-direction: column;
  height: 100vh;
}
#content {
  position: relative;
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100%;
  padding: 20px 40px;
  background-color: #f8f8f8;
}
#timer-tabs {
  width: 100%
}
.tab-item {
  display: flex;
  position: relative;
  padding-top: 25px;
  height: 85vh;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.tab-item:focus {
  outline: none;
}
.load-save-container {
  display: flex;
  flex-direction: row;
  width: 100%;
  max-height: 5vh;
}
.reset-button {
  margin-top: 20px;
}
.modify-timer {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  position: absolute;
  bottom: 20px;
  right: 40px;
}
.circle {
  position: absolute;
  top: 5px;
  right: 0px;
  width: 16px; 
  height: 16px; 
  border-radius: 8px;
  border: #4a4a4a solid 1px;
}
#f {
  display: none;
}
#solo-load-button {
  position: absolute;
  bottom: 20px;
  left: 40px;
}
tr:hover {
  cursor: pointer;
}
</style>
