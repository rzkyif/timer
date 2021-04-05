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
          <div class="card-horizontal-content">
            <div>
              <b-field label="Hours">
                <b-numberinput value="0" min="0" max="99" v-model="newTimer.hours"></b-numberinput>
              </b-field>
            </div>
            <div>
              <b-field label="Minutes">
                <b-numberinput value="0" min="0" max="59" v-model="newTimer.minutes"></b-numberinput>
              </b-field>
            </div>
            <div>
              <b-field label="Seconds">
                <b-numberinput value="0" min="0" max="59" v-model="newTimer.seconds"></b-numberinput>
              </b-field>
            </div>
          </div>
        </div>
        <div class="card-footer">
          <a class="card-footer-item" @click="closeAddModal">Close</a>
          <a class="card-footer-item" @click="addTimer">Add</a>
        </div>
      </div>
    </b-modal>
    <div id="sidebar">
      <b-table id="activity-table"
        :data="tableData"
        :columns="tableColumns"
        :selected.sync="selected"
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
      <div class="add-timer">
        <b-button type="is-text" @click="openAddModal">Add Timer</b-button>
      </div>
      <b-tabs id="timer-tabs" type="is-boxed">
        <b-tab-item v-for="timer in timers" :key="timer.id" :label="timer.name">
          <Timer :max="timer.time" :resetTrigger="timer.resetTrigger"></Timer>
          <b-button class="reset-button" @click="timer.resetTrigger = !timer.resetTrigger">Reset</b-button>
        </b-tab-item>
      </b-tabs>
    </div>
    <input id="f" type="file" @change="loadFunction"/>
  </div>
</template>

<script>
import Timer from './components/Timer.vue'

export default {
  name: 'App',
  components: {
    Timer
  },
  data() {
    return {
      tableData: [
        {id: 1, event: "A", displayTime: "", time: 10},
        {id: 2, event: "B", displayTime: "", time: 5},
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
          resetTrigger: false
        },
        {
          name: '30',
          time: 30,
          resetTrigger: false
        },
        {
          name: '20',
          time: 20,
          resetTrigger: false
        },
        {
          name: '15',
          time: 15,
          resetTrigger: false
        },
        {
          name: '10',
          time: 10,
          resetTrigger: false
        },
        {
          name: '5',
          time: 5,
          resetTrigger: false
        }
      ],
      newTimer: {
        name: '',
        hours: 0,
        minutes: 0,
        seconds: 0,
        time: 0,
        resetTrigger: false
      },
      selected: null,
      isAddModalOpen: false,
    }
  },
  mounted() {
    this.selected = this.tableData[0];
    this.processDataTime();
  },
  methods: {
    processDataTime() {
      for (const row of this.tableData) {
        row.displayTime = this.secondsToTime(row.time)
      }
    },
    secondsToTime(totalSeconds) {
      let hours = Math.floor(totalSeconds / 3600);
      let minutes = Math.floor(totalSeconds / 60) % 60;
      let seconds = totalSeconds % 60;
      return `${hours > 0 ? hours+'h ' : ''}${minutes > 0 ? minutes+'m ' : ''}${seconds+'s'}`
    },
    timeToSeconds(time) {
      let clean = time.replace(' ', '');
      let s1 = clean.includes('h') ? clean.split('h') : ['0', clean];
      let hours = parseInt(s1[0]);
      let s2 = s1[1].includes('m') ? s1[1].split('m') : ['0', s1[1]];
      let minutes = parseInt(s2[0])
      let seconds = parseInt(s2[1].split('s')[0]);
      return hours*3600 + minutes*60 + seconds;
    },
    resetAddModal() {
      this.newTimer.name = "";
      this.newTimer.hours = 0;
      this.newTimer.minutes = 0;
      this.newTimer.seconds = 0;
      this.newTimer.time = 0;
    },
    openAddModal() {
      this.isAddModalOpen = true;
    },
    closeAddModal() {
      this.isAddModalOpen = false;
    },
    addTimer() {
      this.newTimer.time = this.newTimer.hours*3600 + this.newTimer.minutes*60 + this.newTimer.seconds
      this.timers.push({
        name: this.newTimer.name,
        time: this.newTimer.time,
        resetTrigger: false,
      });
      this.closeAddModal();
      this.resetAddModal();
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
      this.selected = this.tableData[0];
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
    selected: function() {
      this.timers[0].time = this.selected.time;
      this.timers[0].resetTrigger = !this.timers[0].resetTrigger;
    }
  }
}
</script>

<style lang="scss">
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
  margin: -1px;
}
#content {
  display: flex;
  flex-direction: column;
  width: 100%;
  padding: 20px 40px;
  background-color: #f8f8f8;
}
#timer-tabs {
  width: 100%
}
.tab-item {
  display: flex;
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
}
.reset-button {
  margin-top: 50px;
}
.add-timer {
  position: absolute;
  top: 20px;
  right: 40px;
}
.card-horizontal-content {
  display: flex;
  flex-direction: row;
  justify-content: center;
}
.card-horizontal-content > div {
  padding: 0px 10px;
}
#f {
  display: none;
}
</style>
