<template>
  <div>
    <div class="flex justify-center">
      <bounce-loader :loading="isLoading" :size="100" :color="'#68d391'" />
    </div>
    <px-assets-table v-if="!isLoading" v-bind:assets="assets" />
  </div>
</template>

<script>
import api from "@/api";
import PxAssetsTable from "@/components/PxAssetsTable";

export default {
  name: "Home",
  components: { PxAssetsTable },

  data() {
    return {
      isLoading: false,
      assets: [],
    };
  },

  created() {
    this.isLoading = true;
    api
      .getAssets()
      .then((assets) => (this.assets = assets))
      .finally(() => (this.isLoading = false));
  },
};
</script>
