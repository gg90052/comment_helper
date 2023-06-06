<template>
  <div :class="showVideo ? '':'hidden'">
    <iframe width="560" height="315" :src="videoURL" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
  </div>
</template>

<script setup lang="ts">
const showVideo = ref(false);
const videoURL = ref('https://www.youtube.com/embed/dCn8UppZGOo');
const CHANNEL_ID = 'UCNLxbtdTe-fFl8uDUS6tMcw';
const YOUTUBE_API_KEY = process.env.VUE_APP_YOUTUBE_API_KEY;

const getVideo = () => {
  fetch(`https://www.googleapis.com/youtube/v3/search?part=snippet&channelId=${CHANNEL_ID}&maxResults=1&order=date&key=${YOUTUBE_API_KEY}`)
  .then(response => response.json())
  .then(data => {
    // 處理 API 回傳的影片結果
    const videos = data.items;
    videos.forEach(video => {
      const videoId = video.id.videoId;
      videoURL.value = `https://www.youtube.com/embed/${videoId}`;
      showVideo.value = true;
    });
  })
  .catch(error => {
    console.error('發生錯誤：', error);
  });
}
getVideo();

const setVideoShow = (show: boolean) => {
  showVideo.value = show;
}

defineExpose({
  setVideoShow
})
</script>
