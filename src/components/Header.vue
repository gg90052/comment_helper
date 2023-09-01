<template>
  <header
    class="w-full text-white text-3xl text-left grid grid-cols-2 relative mobile:grid-cols-1 mobile:text-2xl mobile:text-center"
  >
    <div
      @click="change('home')"
      class="block cursor-pointer bg-chBlue hover:bg-chLightblue pl-12 py-3 mobile:pl-0"
    >
      Facebook粉絲團留言抽籤小助手
    </div>
    <div
      @click="change('plus')"
      class="block cursor-pointer bg-green-600 hover:bg-green-400 pl-12 py-3 mobile:pl-0"
    >
      比對工具
    </div>
    <a
      href="#"
      @click.prevent="change('import')"
      class="absolute right-2 bottom-2 text-xs"
      >v09010919</a
    >
  </header>
</template>
<script lang="ts" setup>
import { useDataStore } from "@/store/modules/data";
const emit = defineEmits(["pageChange"]);
const change = (page) => {
  const dataStore = useDataStore();
  dataStore.$reset;
  emit("pageChange", page);
};
</script>
