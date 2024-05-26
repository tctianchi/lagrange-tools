<template>
  <v-card>
    <v-card-title>AirBorne</v-card-title>
    <v-card-text>
      <v-container>
        <v-row>
          <v-col cols="6">
            <v-text-field
              ref="refBegin"
              v-model="beginLabel"
              label="Begin"
              :hint="distanceLabel"
              persistent-hint
              append-inner-icon="mdi-clipboard"
              @click:append-inner="pasteToBegin"
            >
            </v-text-field>
          </v-col>
          <v-col cols="6">
            <v-text-field
              ref="refEnd"
              v-model="endLabel"
              label="End"
              append-inner-icon="mdi-clipboard"
              @click:append-inner="pasteToEnd"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="6">
            <v-text-field
              v-model="speed"
              label="Speed"
              type="number"
              :hint="durationLabel"
              persistent-hint
            ></v-text-field>
          </v-col>
          <v-col cols="6">
            <v-text-field v-model="endTimeLabel" label="End Time" :hint="beginTimeLabel" persistent-hint></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-divider></v-divider>
          </v-col>
        </v-row>
        <v-row> </v-row>
      </v-container>
    </v-card-text>
  </v-card>
</template>

<script>
export default {
  name: 'AirBorne',
  props: {},
  data() {
    return {
      beginLabel: '',
      endLabel: '',
      speed: '',
      endTimeLabel: '',
    }
  },
  computed: {
    distance() {
      try {
        let [x1, y1] = this.beginLabel.replace('(', '').replace(')', '').split(',', 2)
        let [x2, y2] = this.endLabel.replace('(', '').replace(')', '').split(',', 2)
        x1 = parseInt(x1)
        y1 = parseInt(y1)
        x2 = parseInt(x2)
        y2 = parseInt(y2)
        return Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2) * 1e4
      } catch {
        return 0
      }
    },
    distanceLabel() {
      return `Distance: ${this.distance.toFixed(0)}`
    },
    duration() {
      return this.distance / this.speed
    },
    durationLabel() {
      return `Duration: ${this.convertSecondsToTime(this.duration)}`
    },
    beginTime() {
      // prepend date of today in front of endTimeLabel, and parse this date
      const cur = new Date()
      let [hh, mm, ss] = this.endTimeLabel.split(':', 3)
      if (!mm) {
        mm = 0
      }
      if (!ss) {
        ss = 0
      }
      let t = new Date(cur.getFullYear(), cur.getMonth(), cur.getDate(), hh, mm, ss)
      // sub t by duration
      t.setSeconds(t.getSeconds() - this.duration)
      return t
    },
    beginTimeLabel() {
      return `Begin time: ${this.formatTime(this.beginTime)}`
    },
  },
  mounted() {
    this.setDefaultValues()
  },
  methods: {
    setDefaultValues() {
      this.beginLabel = '(0,0)'
      this.endLabel = '(3000,3000)'
      this.speed = 4000
      const cur = new Date()
      cur.setHours(cur.getHours() + 1)
      cur.setMinutes(0)
      cur.setSeconds(0)
      this.endTimeLabel = this.formatTime(cur)
    },
    async pasteToBegin() {
      try {
        this.beginLabel = await navigator.clipboard.readText()
      } catch (err) {
        alert('can not paste from the clipboard')
      }
    },
    async pasteToEnd() {
      try {
        this.endLabel = await navigator.clipboard.readText()
      } catch (err) {
        alert('can not paste from the clipboard')
      }
    },
    convertSecondsToTime(seconds) {
      if (isNaN(seconds)) {
        return '--'
      }
      const hh = Math.floor(seconds / 3600)
      const mm = Math.floor((seconds % 3600) / 60)
      const ss = Math.floor(seconds % 60)
      return `${hh.toString().padStart(2, '0')}:${mm.toString().padStart(2, '0')}:${ss.toString().padStart(2, '0')}`
    },
    formatTime(t) {
      return t.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false })
    },
  },
}
</script>

<style scoped></style>
