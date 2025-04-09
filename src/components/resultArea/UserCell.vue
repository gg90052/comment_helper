<template>
  <template v-if="hasFromDetail">
    <a :href="profileUrl" class="text-[#4094c5]" target="_blank">{{
      displayName
    }}</a>
  </template>
  <template v-else>{{ displayName }}</template>
</template>

<script lang="ts" setup>
import { computed } from "vue";
import { useDataStore } from "@/store/modules/data";

interface CommentFrom {
  id: string;
  name: string;
}

interface CommentData {
  id: string;
  hasFromDetail?: boolean;
  from?: CommentFrom;
  [key: string]: any;
}

const props = defineProps({
  commentData: {
    type: Object as () => CommentData,
    required: true,
  },
});

const dataStore = useDataStore();

const hasFromDetail = computed(() => {
  return props.commentData.hasFromDetail;
});

const displayName = computed(() => {
  if (dataStore.needPaid === true && dataStore.payedUser === false) {
    return "undefined";
  } else {
    return props.commentData.from
      ? props.commentData.from.name
      : props.commentData.name
      ? props.commentData.name
      : props.commentData.id;
  }
});

const profileUrl = computed(() => {
  return `https://www.facebook.com/${props.commentData.from?.id}`;
});
</script>
