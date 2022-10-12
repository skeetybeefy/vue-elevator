<template>
  <div class="shaft">
    <div class="shaft-floor" v-for="floor in floors" :key="floor"></div>
    <div
      class="elevator"
      :style="{ transform: `translateY(${(floor - 1) * -100}%)` }"
      :class="{ isIdle }"
    >
      <p>{{ targetFloor }}</p>
      <p>{{ displayingDirection }}</p>
    </div>
  </div>
</template>
<script>
export default {
  name: "Elevator",
  props: ["floors", "floor", "direction", "isIdle", "targetFloor"],
  computed: {
    displayingDirection() {
      if (this.direction === "still") {
        return "";
      } else if (this.direction === "up") {
        return "↑";
      } else {
        return "↓";
      }
    },
  },
};
</script>
<style scoped>
.shaft {
  position: relative;
  display: flex;
  flex-direction: column-reverse;
}

.shaft-floor {
  border: 1px solid gray;
  min-width: 100px;
  height: 100px;
  box-sizing: border-box;
}

.elevator {
  position: absolute;
  bottom: 0;
  display: flex;
  justify-content: center;
  gap: 10px;
  min-width: 100%;
  height: 100px;
  background-color: rgba(221, 14, 14, 0.5);
  transition: transform 1s linear;
}

.isIdle {
  background-color: rgba(14, 221, 38, 0.5);
}
</style>