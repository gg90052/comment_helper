<template>
  <div class="w-full max-w-[1280px] my-0 mx-auto">
    <HeaderComponent @pageChange="pageChange" />
    <template v-if="pageStatus === 'home'">
      <MainArea />
      <DonateArea v-show="!hasData" />
      <FetchResults v-if="hasData" />
    </template>
    <template v-if="pageStatus === 'plus'">
      <Upload />
      <CompareResult />
    </template>
    <template v-if="pageStatus === 'import'">
      <ImportUpload />
      <ImportResult v-if="hasData" />
    </template>
    <GlobalModal v-if="showModal" :data="modalData" />
  </div>
</template>
<script lang="ts" setup>
import HeaderComponent from "@/components/Header.vue";
import MainArea from "@/components/mainArea/MainArea.vue";
import DonateArea from "@/components/donateArea/DonateArea.vue";
import FetchResults from "@/components/resultArea/FetchResults.vue";
import Upload from "@/components/plus/Upload.vue";
import CompareResult from "@/components/plus/CompareResult.vue";
import ImportUpload from "@/components/import/ImportUpload.vue";
import ImportResult from "@/components/import/ImportResult.vue";
import GlobalModal from "@/components/GlobalModal.vue";
import { useDataStore } from "@/store/modules/data";
const dataStore = useDataStore();
const pageStatus = ref("home");
const showModal = ref(false);
const modalData = ref({});
const hasData = computed(() => {
  return dataStore.rawData.length > 0;
});
const pageChange = (page) => {
  pageStatus.value = page;
};
const globalModalShow = (show: boolean, data) => {
  modalData.value = data;
  showModal.value = show;
};
provide("globalModal", globalModalShow);
</script>
<style scoped lang="scss"></style>
