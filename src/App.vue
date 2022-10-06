<template>
  <div class="app">
    <elevator
      :floors="floors"
      v-for="elevator in elevators"
      :key="elevator"
      :callQueue="callQueue"
      @arrived="arrived"
      @started="started"
      ref="elevator"
      :idleElevators="idleElevators"
    ></elevator>
    <controls
      :floors="floors"
      :handleButtonPress="handleButtonPress"
      :callQueue="callQueue"
      :targetFloors="targetFloors"
    ></controls>
  </div>
  <div style="text-align: left">CallQueue {{ callQueue }}</div>
  <div style="text-align: left">Target Floors {{ targetFloors }}</div>
  <div style="text-align: left">Idle floors {{ idleFloors }}</div>
  <div style="text-align: left">Idle elevators {{ idleElevators }}</div>
</template>

<script>
import Elevator from "./components/Elevator.vue";
import Controls from "./components/Controls.vue";

export default {
  name: "App",
  components: {
    Elevator,
    Controls,
  },
  data() {
    return {
      callQueue: [],
      targetFloors: [],
      idleFloors: [],
      idleElevators: [],
      floors: 7,
      elevators: 5,
    };
  },
  watch: {
    callQueue() {
      if (this.idleElevators.length && this.callQueue.length) {
        this.sendClosestElevator()
      }
    },
    idleElevators() {
      if (this.idleElevators.length && this.callQueue.length) {
        this.sendClosestElevator()
      }
    }
  },
  methods: {
    handleButtonPress(floor) {
      if (
        !this.callQueue.includes(floor) &&
        !this.idleFloors.includes(floor) &&
        !this.targetFloors.includes(floor)
      ) {
        this.callQueue = [...this.callQueue, floor];
      }
    },
    sendClosestElevator() {
      const targetFloor = this.callQueue[this.callQueue.length - 1];
      const minDistance = this.idleElevators
        .map((key) => {
          return this.$refs.elevator[key - 1].floor;
        })
        .reduce((acc, curr) => {
          const currDistance = Math.abs(curr - targetFloor);
          return currDistance < acc ? currDistance : acc;
        }, Number.POSITIVE_INFINITY);
      const closestElevator = this.idleElevators.find(
        (key) =>
          Math.abs(this.$refs.elevator[key - 1].floor - targetFloor) ===
          minDistance
      );
      this.$refs.elevator[closestElevator - 1].goToFloor(
        this.callQueue.shift()
      );
    },
    started(key, [sourceFloor, targetFloor]) {
      const sourceIndex = this.idleFloors.indexOf(sourceFloor);
      this.idleFloors.splice(sourceIndex, 1);
      const keyIndex = this.idleElevators.indexOf(key);
      this.idleElevators.splice(keyIndex, 1);
      this.targetFloors = [...this.targetFloors, targetFloor];
    },
    arrived(key, floor) {
      const index = this.targetFloors.indexOf(floor);
      this.targetFloors.splice(index, 1);
      this.idleFloors = [...this.idleFloors, floor];
      this.idleElevators = [...this.idleElevators, key];
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.app {
  display: flex;
  gap: 5px;
}
</style>
