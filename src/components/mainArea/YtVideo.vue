<template>
  <div :class="showVideo ? '':'hidden'" class="w-full h-0 pb-[56.25%] relative">
    <iframe class="w-full h-full absolute left-0 top-0" ref="iframe" :src="videoURL" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
  </div>
</template>

<script setup lang="ts">
const showVideo = ref(true);
const VIDEO_ID = 'kPVZ-6JMpNc';
const videoURL = ref('https://www.youtube.com/embed/' + VIDEO_ID);


const setVideoShow = (show: boolean) => {
  showVideo.value = show;
}

defineExpose({
  setVideoShow
})
</script>
