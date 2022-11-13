<template>
  <div id="app">
    <HelloWorld />
    <CardGlare />
  </div>
</template>

<script>
import HelloWorldVue from "./components/HelloWorld.vue";
import CardGlareVue from "./components/CardGlare.vue";

export default {
  components: {
    // Path: src/components/HelloWorld.vue
    HelloWorld: HelloWorldVue,
    // Path: src/components/CardGlare.vue
    CardGlare: CardGlareVue,
  },
};
</script>

<style></style>
