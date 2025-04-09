<template>
  <a :href="messageUrl" target="_blank" class="text-[#D68927]">
    {{ displayContent }}
    <ReactionIcon
      v-if="props.commentData.type"
      :reaction="props.commentData.type"
    />
  </a>
</template>

<script lang="ts" setup>
import { computed } from "vue";
import ReactionIcon from "./ReactionIcon.vue";

interface CommentData {
  id: string;
  message?: string;
  hasFromDetail?: boolean;
  link?: string;
  [key: string]: any;
}

const props = defineProps({
  commentData: {
    type: Object as () => CommentData,
    required: true,
  },
});

const displayContent = computed(() => {
  if (props.commentData.message) {
    //社團抓留言
    return props.commentData.message;
  } else if (props.commentData.type) {
    return "";
  } else {
    return props.commentData.id;
  }
});

const messageUrl = computed(() => {
  if (props.commentData.hasFromDetail) {
    //社團抓留言
    return props.commentData.link || "";
  } else {
    return "https://www.facebook.com/" + props.commentData.id;
  }
});
</script>
