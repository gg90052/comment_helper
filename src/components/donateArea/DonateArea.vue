<template>
  <div
    class="flex flex-nowrap justify-between items-start pt-6 px-7 mb-5 bg-white"
  >
    <Donate></Donate>
    <!-- 小助手 -->
    <div class="adsItem grow ml-4">
      <ins
        class="adsbygoogle"
        style="display: block"
        data-ad-client="ca-pub-2578812574511816"
        data-ad-slot="6865902461"
        data-ad-format="auto"
        data-full-width-responsive="true"
      ></ins>
    </div>
    <!-- <FB></FB> -->
  </div>
</template>

<script setup lang="ts">
import Donate from "./Donate.vue";
import FB from "./FB.vue";
onMounted(() => {
  window.addAds();
});
</script>
