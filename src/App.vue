<template>
  <div class="app">
    <elevator
      :floors="floors"
      v-for="elevator in elevators"
      v-bind="elevatorsState[elevator - 1]"
      :key="elevator"
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
import config from "./config.json";

const elevatorState = {
  floor: 1,
  isIdle: true,
  direction: "still",
  targetFloor: null,
};

export default {
  name: "App",
  components: {
    Elevator,
    Controls,
  },
  setup() {
    return {
      ...config,
    };
  },
  data() {
    return {
      callQueue: [],
      targetFloors: [],
      idleFloors: [],
      idleElevators: [],
      elevatorsState: Array(this.elevators)
        .fill()
        .map(() => {
          return { ...elevatorState };
        }),
    };
  },
  watch: {
    callQueue: {
      handler(newValue) {
        localStorage.setItem("callQueue", JSON.stringify(newValue));
        if (this.idleElevators.length && this.callQueue.length) {
          this.sendClosestElevator();
        }
      },
      deep: true,
    },
    targetFloors: {
      handler(newValue) {
        localStorage.setItem("targetFloors", JSON.stringify(newValue));
      },
      deep: true,
    },
    idleFloors: {
      handler(newValue) {
        localStorage.setItem("idleFloors", JSON.stringify(newValue));
      },
      deep: true,
    },
    idleElevators: {
      handler(newValue) {
        localStorage.setItem("idleElevators", JSON.stringify(newValue));
        if (this.idleElevators.length && this.callQueue.length) {
          this.sendClosestElevator();
        }
      },
      deep: true,
    },
    elevatorsState: {
      handler(newValue) {
        localStorage.setItem("elevatorsState", JSON.stringify(newValue));
      },
      deep: true,
    },
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
          return this.elevatorsState[key - 1].floor;
        })
        .reduce((acc, curr) => {
          const currDistance = Math.abs(curr - targetFloor);
          return currDistance < acc ? currDistance : acc;
        }, Number.POSITIVE_INFINITY);
      const closestElevator = this.idleElevators.find(
        (key) =>
          Math.abs(this.elevatorsState[key - 1].floor - targetFloor) ===
          minDistance
      );
      this.goToFloor(closestElevator - 1, this.callQueue.shift());
    },
    async goToFloor(elevator, targetFloor) {
      if (targetFloor) {
        this.elevatorsState[elevator].targetFloor = targetFloor;
      }
      if (this.elevatorsState[elevator].isIdle) {
        this.started(elevator + 1, [
          this.elevatorsState[elevator].floor,
          this.elevatorsState[elevator].targetFloor,
        ]);
      }
      this.elevatorsState[elevator].isIdle = false;
      if (this.elevatorsState[elevator].floor === targetFloor) {
        await new Promise((r) => setTimeout(r, 3000));
        this.elevatorsState[elevator].isIdle = true;
        this.elevatorsState[elevator].direction = "still";
        this.arrived(elevator + 1, this.elevatorsState[elevator].targetFloor);
        this.elevatorsState[elevator].targetFloor = null;
        return null;
      } else {
        if (this.elevatorsState[elevator].floor > targetFloor) {
          this.elevatorsState[elevator].direction = "down";
          this.elevatorsState[elevator].floor -= 1;
          await new Promise((r) => setTimeout(r, 1000));
          this.goToFloor(elevator, targetFloor);
        } else {
          this.elevatorsState[elevator].direction = "up";
          this.elevatorsState[elevator].floor += 1;
          await new Promise((r) => setTimeout(r, 1000));
          this.goToFloor(elevator, targetFloor);
        }
      }
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
  mounted() {
    if (
      !this.callQueue.length &&
      !this.targetFloors.length &&
      !this.idleFloors.length &&
      !this.idleElevators.length
    ) {
      this.elevatorsState.forEach((_, index) => this.arrived(index + 1, 1));
    }
    if (localStorage.callQueue) {
      this.callQueue = JSON.parse(localStorage.callQueue);
    }
    if (localStorage.targetFloors) {
      this.targetFloors = JSON.parse(localStorage.targetFloors);
    }
    if (localStorage.idleFloors) {
      this.idleFloors = JSON.parse(localStorage.idleFloors);
    }
    if (localStorage.idleElevators) {
      this.idleElevators = JSON.parse(localStorage.idleElevators);
    }
    if (localStorage.elevatorsState) {
      this.elevatorsState = JSON.parse(localStorage.elevatorsState);
    }
    this.elevatorsState.forEach((elevator, index) => {
      if (!elevator.isIdle) {
        this.goToFloor(index, elevator.targetFloor)
      }
    })
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
