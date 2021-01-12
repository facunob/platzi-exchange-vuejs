<template>
  <button
    v-on:click="buttonClick"
    class="bg-transparent hover:bg-green-500 text-green-700 font-semibold hover:text-white p-2"
  >
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: "PxButton",

  methods: {
    buttonClick() {
      this.$emit("custom-click");
    },
  },
};
</script>
