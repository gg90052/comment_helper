<template>
  <a
    v-if="state.post.id"
    target="_blank"
    :href="`https://www.facebook.com/${state.post.id}`"
  >
    <div class="w-[560px] h-[315px] px-4 py-2 bg-gray-200">
      <p class="text-xs mb-1">選擇貼文預覽</p>
      <div class="w-full h-[220px] overflow-hidden flex justify-center">
        <img
          class="h-full"
          :src="state.post.full_picture || 'https://placehold.co/560x315'"
          alt=""
        />
      </div>
      <article class="ellipse text-left text-xs mt-2 text-blue-700">
        {{ state.post.message }}
      </article>
    </div>
  </a>
</template>

<script setup lang="ts">
import { useDataStore } from "@/store/modules/data";
const dataStore = useDataStore();
const state = reactive({
  post: computed(() => dataStore.postData),
});
</script>
<style lang="scss" scoped>
.ellipse {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
</style>
