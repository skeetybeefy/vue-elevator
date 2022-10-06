<template>
  <div class="controls">
    <div class="control" v-for="floor in floors" :key="floor">
      <p>{{ floor }}</p>
      <div @click="handleButtonPress(floor)" class="elevator-button" :class="{active: isButtonActive(floor)}"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Controls",
  props: [
    "floors", 
    "handleButtonPress", 
    "callQueue", 
    "targetFloors"
  ],
  methods: {
    isButtonActive(floor) {
      return this.callQueue.includes(floor) || this.targetFloors.includes(floor)
    }
  },
};
</script>

<style scoped>
.controls {
  display: flex;
  flex-direction: column-reverse;
}

.control {
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 1px solid gray;
  min-width: 100px;
  height: 100px;
  box-sizing: border-box;
}

.elevator-button {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: 1px solid black;
  cursor: pointer;
}

.active {
  background-color: rgb(235, 184, 15);
}
</style>