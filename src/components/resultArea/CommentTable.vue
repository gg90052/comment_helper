<template>
  <table
    class="resultTable table table-auto whitespace-normal table-zebra w-full"
  >
    <!-- 表頭部分 -->
    <TableHead :data="datas[0]" @sort="sort" />

    <!-- 表格主體 -->
    <tbody>
      <template v-for="(tr, index) in sortTableData" :key="tr.id">
        <tr>
          <!-- 序號 -->
          <th>{{ index + 1 }}</th>

          <!-- 名稱 - 使用自定義組件 -->
          <td>
            <UserCell :commentData="tr" />
          </td>

          <!-- 留言內容 - 使用自定義組件 -->
          <td class="w-3/5 whitespace-normal hover:underline">
            <MessageCell :commentData="tr" />
          </td>

          <!-- 按讚數 -->
          <td class="text-center">{{ tr.like_count }}</td>

          <!-- 留言時間 -->
          <td v-if="tr.created_time">{{ generateDate(tr.created_time) }}</td>
        </tr>
      </template>
    </tbody>
  </table>
</template>
<script lang="ts" setup>
import dayjs from "dayjs";
import { useDataStore } from "@/store/modules/data";
import TableHead from "./TableHead.vue";
import UserCell from "./UserCell.vue";
import MessageCell from "./MessageCell.vue";
import { ref, computed, watchEffect } from "vue";

// 定義評論數據的介面
interface CommentFrom {
  id: string;
  name: string;
}

interface CommentData {
  id: string;
  message?: string;
  created_time: string;
  like_count: number;
  hasFromDetail?: boolean;
  from?: CommentFrom;
  link?: string;
  [key: string]: any; // 允許其他可能的屬性
}

interface FileData {
  id: string | number;
  datas: CommentData[];
  [key: string]: any;
}

const dataStore = useDataStore();
const sortKey = ref<string>("created_time");
const sortDir = ref<boolean>(false);
const sortTableData = ref<CommentData[]>([]);

const datas = computed<CommentData[][]>(() => {
  if (props.useCompare === true) {
    const files = dataStore.files as FileData[];
    const file = files.find((item) => item.id === dataStore.showFileTable);
    return file?.datas ? [file.datas] : [];
  } else {
    return props.datas.length > 0
      ? props.datas
      : (dataStore.filteredData as CommentData[][]);
  }
});

const generateDate = (time: string): string => {
  return dayjs(time).isValid()
    ? dayjs(time).format("YYYY-MM-DD HH:mm:ss")
    : time;
};

const props = defineProps({
  useCompare: {
    type: Boolean,
    default: false,
  },
  datas: {
    type: Array as () => CommentData[][],
    default: () => [],
  },
  sort: {
    type: Boolean,
    default: true,
  },
});

const sort = (key: string) => {
  if (sortKey.value === key) {
    sortDir.value = !sortDir.value;
  } else {
    sortKey.value = key;
    sortDir.value = false;
  }
  sortTable();
};

const sortTable = () => {
  if (props.sort === true) {
    // 確保數據是一個一維數組
    const currentData = Array.isArray(datas.value[0])
      ? datas.value[0]
      : datas.value;
    const pureDatas: CommentData[] = JSON.parse(JSON.stringify(currentData));
    callWorker(pureDatas, sortDir.value, sortKey.value);
    worker.onmessage = function (event) {
      sortTableData.value = event.data;
    };
  } else {
    // 確保數據是一個一維數組
    sortTableData.value = Array.isArray(datas.value[0])
      ? (datas.value[0] as CommentData[])
      : (datas.value as unknown as CommentData[]);
  }
};

const callWorker = (datas: CommentData[], dir: boolean, key: string) => {
  worker.postMessage({ datas, dir, key });
};

const code = `(function () {
  self.onmessage = function (event){
    const {datas, dir, key} = event.data;
    const sortResult = datas.sort((a, b) => {
      if (dir){
        return a[key] > b[key] ? 1 : -1;
      }else{
        return a[key] < b[key] ? 1 : -1;
      }
    });
    self.postMessage(sortResult);
  }
})();`;

const createBlobObjectURL = (code: string) => {
  const blob = new Blob([`${code}`], { type: "text/javascript" });
  const url = URL.createObjectURL(blob);
  return url;
};

const worker = new Worker(createBlobObjectURL(code));

// 確保初始化時調用 sortTable
watchEffect(() => {
  if (datas.value && datas.value.length > 0) {
    sortTable();
  }
});

//watch dataStore.filterChange
watchEffect(() => {
  if (dataStore.filterChange === true) {
    sortTable();
    dataStore.setFilterChange(false);
  }
});

defineExpose({
  sort,
});
</script>
