<template>
  <div class="bg-white p-4 grid grid-cols-4 gap-2">
    <p class="col-span-4 text-xl text-cyan-600">本名單由「Facebook 粉絲團留言抽籤小助手」產出</p>
    <template v-for="(card, index) in dataStore.drawResult" :key="card.id">
      <div v-if="showPrizeTitle(index) && dataStore.prizeList.length > 0" class="col-span-4 text-white bg-blue-500 block py-2">{{ showPrizeTitle(index).title }}：<span class="text-yellow-300 font-bold mx-2">{{showPrizeTitle(index).num}}</span>名</div>
      <div class="card w-full bg-base-100 border shadow-xl text-left">
        <div class="card-body relative">
          <h2 class="card-title whitespace-nowrap">
            <div class="w-12 aspect-square overflow-hidden rounded-full">
              <img :src="pictureUrl(card)" alt="" />
            </div>
            <a v-if="card.from?.link" :href="card.from.link" target="_blank" class="text-[#D68927] hover:underline">{{ card.from.name }}</a>
            <template v-else>{{ name(card) }}</template>
          </h2>
          <p>
            <a v-if="card.message || card.attachment" class="text-[#D68927] hover:underline" :href="messageLink(card)" target="_blank">
              <component :is="renderImage(card)" v-if="card.attachment?.media?.image?.src" />{{ card.message}}
            </a>
            <p v-else-if="card.story === ''"></p>
            <ReactionIcon :reaction="card.type" v-else />
          </p>
          <p v-if="card.created_time" class="absolute text-xs right-4 bottom-1">{{ dayjs(card.created_time).format('YYYY-MM-DD HH:mm:ss') }}</p>
        </div>
      </div>
    </template>
  </div>
</template>
<script setup lang="ts">
import { h } from 'vue';
import dayjs from 'dayjs';
import ReactionIcon from './ReactionIcon.vue';
import { useDataStore } from '@/store/modules/data';
const dataStore = useDataStore();

const name = (card) => {
  if (card.from) {
    return card.from.name;
  }else if (card.name) {
    return card.name;
  }else {
    return card.id;
  }
}
const messageLink = (card) => {
  if (card.hasFromDetail){
    //社團抓留言
    return card.link;
  }else{
    return 'https://www.facebook.com/' + card.id;
  }
}
const titleArray = computed(()=>{
  const arr = [];
  let count = 0;
  dataStore.prizeList.forEach(item=>{
    const obj = Object.assign({count}, item);
    arr.push(obj);
    count += item.num;
  });
  return arr;
});

const renderImage = (card) => {
  if (card.attachment?.media?.image?.src) {
    return h('img', {
      src: card.attachment.media.image.src,
      alt: 'attachment',
      style: {
        maxWidth: '100px',
        height: 'auto'
      }
    });
  }
  return null;
}

const showPrizeTitle = (index) => {
  return titleArray.value.find(item=>item.count === index);
}
const pictureUrl = (card) => {
  if (card.picture){
    return card.picture.url;
  }else{
    const id = card.from ? card.from.id : card.id;
    if (id === 'no_id') return '';
    const token = sessionStorage.overwriteToken ? sessionStorage.overwriteToken : dataStore.accessToken;
    return `https://graph.facebook.com/${id}/picture?type=large&access_token=${token}`;
  }
}

</script>