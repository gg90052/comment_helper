<template>
  <div :class="showVideo ? '':'hidden'">
    <iframe width="560" height="315" :src="videoURL" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
  </div>
</template>

<script setup lang="ts">
const showVideo = ref(true);
const VIDEO_ID = 'dCn8UppZGOo';
const videoURL = ref('https://www.youtube.com/embed/' + VIDEO_ID);


const setVideoShow = (show: boolean) => {
  showVideo.value = show;
}

defineExpose({
  setVideoShow
})
</script>
