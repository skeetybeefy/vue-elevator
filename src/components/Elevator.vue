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
  props: ["floors", "callQueue", "idleElevators"],
  data() {
    return {
      floor: 1,
      isIdle: true,
      direction: "still",
      targetFloor: null,
    };
  },
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
  methods: {
    async goToFloor(targetFloor) {
      if (targetFloor) {
        this.targetFloor = targetFloor
      }
      if (this.isIdle) {
        this.$emit("started", this.$.vnode.key, [this.floor, this.targetFloor]);
      }
      this.isIdle = false;
      if (this.floor === targetFloor) {
        await new Promise((r) => setTimeout(r, 3000));
        this.isIdle = true;
        this.direction = "still";
        this.$emit("arrived", this.$.vnode.key, this.targetFloor);
        this.targetFloor = null;
        return null;
      } else {
        if (this.floor > targetFloor) {
          this.direction = "down";
          this.floor = this.floor - 1;
          await new Promise((r) => setTimeout(r, 1000));
          this.goToFloor(targetFloor);
        } else {
          this.direction = "up";
          this.floor = this.floor + 1;
          await new Promise((r) => setTimeout(r, 1000));
          this.goToFloor(targetFloor);
        }
      }
    },
  },
  mounted() {
    this.$emit("arrived", this.$.vnode.key, this.floor);
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