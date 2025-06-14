<template>
  <a :href="messageUrl" target="_blank" class="text-[#D68927]">
    <span v-html="displayContent"></span>
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
  let content = "";

  if (props.commentData.attachment?.media?.image?.src) {
    content += `<img src="${props.commentData.attachment.media.image.src}" alt="attachment" style="max-width: 100px; height: auto;">`;
  }

  if (props.commentData.message) {
    //社團抓留言
    content += props.commentData.message;
  } else if (props.commentData.type) {
    content += "";
  } else if (!props.commentData.attachment && !props.commentData.message) {
    content += props.commentData.id;
  }

  return content;
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
