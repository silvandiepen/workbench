<template>
  <WorkBench />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import WorkBench from "./components/WorkBench.vue";

export default defineComponent({
  name: "App",
  components: {
    WorkBench,
  },
});
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100vh;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
    Ubuntu, Cantarell, "Fira Sans", "Droid Sans", "Helvetica Neue", Arial,
    sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  background-color: #f7f7f7;
}
* {
  box-sizing: border-box;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
}
.workbench {
}
</style>
