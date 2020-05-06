<template>
  <div class="searchLocation">
    <div>
      <input
        class="searchBox"
        type="text"
        placeholder="목적지를 입력해주세요."
        v-model="searchValue"
        @keydown="checkEnter"
      /><img src="../imgs/search.png" alt="" @click="locationSend" />
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      searchValue: '',
    };
  },
  methods: {
    locationSend() {
      if (this.searchValue != '') this.$emit('send_location', this.searchValue);
    },
    checkEnter(keyValue) {
      if (keyValue.key === 'Enter') this.locationSend();
    },
  },
};
</script>

<style></style>
