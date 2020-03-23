<template>
  <div class="header">
    <img src="../imgs/mask.png" alt="mask" />공적 마스크 검색 사이트
  </div>
</template>

<script>
export default {};
</script>

<style>
.header {
  position: absolute;
  top: 0;
  width: 100%;
  height: 40px;
  line-height: 48px;
  background-color: rgba(98, 118, 228, 0.849);
  z-index: 10000;
  text-align: center;

  font-size: 28px;
  /* font-weight: 100; */
  color: rgb(255, 255, 255);
  font-family: 'Jua', sans-serif;
}

.header img {
  width: 25px;
  padding-right: 5px;
}
</style>
