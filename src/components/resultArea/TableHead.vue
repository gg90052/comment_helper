<template>
  <thead>
    <tr>
      <th class="!rounded-t-none">序號</th>
      <th>名稱</th>
      <th v-if="props.data && props.data.link" class="w-3/5 !rounded-t-none">
        塗鴉牆網址
      </th>
      <th v-if="props.data && props.data.message" class="w-3/5">留言內容</th>
      <th v-if="props.data && props.data.type" class="!rounded-t-none">心情</th>
      <th
        v-if="props.data && props.data.like_count !== undefined"
        @click="sort('like_count')"
        class="cursor-pointer"
      >
        按讚數
      </th>
      <th
        v-if="props.data && props.data.created_time"
        @click="sort('created_time')"
        class="!rounded-t-none cursor-pointer"
      >
        留言時間
      </th>
    </tr>
  </thead>
</template>

<script setup lang="ts">
const props = defineProps({
  data: {
    type: Object,
    default: () => {},
  },
});
const emit = defineEmits(["sort"]);
const sort = (key: string) => {
  emit("sort", key);
};
</script>
