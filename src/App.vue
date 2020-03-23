<template>
  <div class="app">
    <AppHeader></AppHeader>
    <MaskMap></MaskMap>
  </div>
</template>

<script>
import AppHeader from '@/components/AppHeader.vue';
import MaskMap from '@/views/MaskMap.vue';

export default {
  components: {
    AppHeader,
    MaskMap,
  },
};
</script>

<style>
@import url(./css/reset.css);
@import url(./css/map.css);

.app {
  min-width: 310px !important;
}
</style>
