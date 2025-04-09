<template>
  <div class="loading-modal text-left" v-if="show">
    <div class="loading-modal__content">
      <div
        class="absolute top-2 right-0 w-10 text-2xl text-gray-500 cursor-pointer"
        @click="emit('close')"
      >
        X
      </div>
      <div class="flex h-full w-full flex-row mobile:flex-col">
        <!-- 左側粉絲專頁列表 -->
        <div
          class="flex-shrink-0 border-gray-400 overflow-auto p-2 h-full mobile:h-[250px] w-[300px] mobile:w-full border-r mobile:border-b mobile:border-r-0"
        >
          <p
            @dblclick="pageNameInputTrigger"
            class="text-base font-semibold mb-3 px-2"
          >
            粉絲專頁 Fanpages
          </p>
          <div class="select_page space-y-1">
            <div class="px-2" v-if="showPageNameInput === true">
              <PageNameInput @selectPage="(page) => selectPage(page)" />
            </div>
            <div
              class="cursor-pointer px-2"
              v-for="(page, index) in pages"
              :key="index"
              @click="selectPage(page)"
            >
              <h3
                class="py-2 px-3 rounded text-lg text-[#3b5fb2] hover:bg-sky-500 hover:text-white transition-colors duration-200"
                :class="{ 'bg-sky-100': target.id === page.id }"
              >
                {{ page.name }}
              </h3>
            </div>
          </div>
        </div>

        <!-- 右側貼文列表 -->
        <div class="flex-1 overflow-auto h-full p-3">
          <div v-if="target.name !== ''">
            <div class="flex items-center flex-wrap mb-6">
              <span class="text-lg font-semibold mr-4"
                >{{ target.name }} 貼文列表</span
              >
              <button
                class="btn-warning h-8 px-4 rounded text-sm"
                @click="copyToken"
              >
                複製token
              </button>
            </div>

            <div class="live_post mb-6">
              <div class="flex flex-wrap items-center gap-4">
                <span>貼文截止時間：</span>
                <datepicker
                  @update:modelValue="onChangeDate"
                  inputFormat="yyyy-MM-dd"
                  class="border rounded"
                  v-model="until"
                />
                <div class="form-control flex-1">
                  <div class="input-group">
                    <input
                      v-model="postFBID"
                      type="text"
                      class="flex-1 h-10 px-3 border rounded-l"
                      placeholder="請輸入貼文FBID"
                    />
                    <button
                      @click="selectPostFromFBID"
                      class="btn-blue px-4 h-10 rounded-r"
                    >
                      用FBID選擇貼文
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div v-if="loading" class="sk-folding-cube">
              <div class="sk-cube1 sk-cube"></div>
              <div class="sk-cube2 sk-cube"></div>
              <div class="sk-cube4 sk-cube"></div>
              <div class="sk-cube3 sk-cube"></div>
            </div>

            <div class="grid grid-cols-3 gap-2" v-if="posts.length > 0">
              <div
                class="border rounded-lg bg-white shadow-sm hover:shadow-md transition-shadow duration-200 text-xs flex flex-col"
                v-for="(post, index) in posts"
                :key="index"
              >
                <!-- 貼文頭部和內容區域 -->
                <div
                  :class="{
                    'flex-1 flex flex-col overflow-hidden': true,
                  }"
                >
                  <!-- 時間顯示 -->
                  <div class="p-2 pb-1">
                    <p class="text-xs text-gray-500 text-right">
                      {{ dayjs(post.created_time).format("YYYY/MM/DD HH:mm") }}
                    </p>
                  </div>

                  <!-- 貼文內容 -->
                  <div
                    class="px-2 pb-2 flex-1 max-h-[150px] overflow-auto mb-4"
                  >
                    <p
                      class="leading-normal whitespace-pre-wrap break-words mb-4"
                    >
                      {{ post.story || post.message || "此貼文沒有文字內容" }}
                    </p>
                  </div>
                </div>

                <!-- 貼文圖片 -->
                <div
                  class="flex-shrink-0 h-[120px] border-y overflow-hidden"
                  ref="imageContainer"
                >
                  <img
                    :src="post.full_picture || 'https://placehold.co/1920x1080'"
                    class="w-full h-full object-cover"
                  />
                </div>

                <!-- 貼文操作區 -->
                <div
                  class="flex items-center justify-between h-[60px] px-2 py-1 mt-auto border-t"
                >
                  <a
                    class="text-blue-600 hover:text-blue-800 text-xs flex items-center shrink-0"
                    :href="`https://www.facebook.com/${post.id}`"
                    target="_blank"
                  >
                    <svg
                      class="w-3 h-3 mr-1"
                      fill="currentColor"
                      viewBox="0 0 24 24"
                    >
                      <path
                        d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 15v-4H8l4-7v4h3l-4 7z"
                      />
                    </svg>
                    在 FB 開啟
                  </a>
                  <button
                    class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 ml-10 rounded text-[16px] transition-colors duration-200"
                    @click="selectPost(post)"
                  >
                    選擇貼文
                  </button>
                </div>
              </div>
            </div>
          </div>
          <div
            v-else
            class="flex items-center justify-center h-full text-gray-500"
          >
            請從左側選擇一個粉絲專頁
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import dayjs from "dayjs";
import Datepicker from "vue3-datepicker";
import { useDataStore } from "@/store/modules/data";
import SnInput from "@/components/SnInput.vue";
import PageNameInput from "@/components/PageNameInput.vue";
import { ref, watch, inject } from "vue";

interface Page {
  id: string;
  name: string;
  access_token: string;
}

interface Post {
  id: string;
  message?: string;
  story?: string;
  created_time: string;
  full_picture?: string;
}

const dataStore = useDataStore();
const emit = defineEmits(["close", "select"]);
const props = defineProps(["show", "accessToken"]);
const showError = inject("globalModal") as (show: boolean, error: any) => void;
const loading = ref(true);
const target = ref<Page>({ id: "", name: "", access_token: "" });
const pages = ref<Page[]>([]);
const posts = ref<Post[]>([]);
const until = ref(new Date());
const postFBID = ref("");
const showPageNameInput = ref(
  localStorage.showPageNameInput === "true" ? true : false
);

const onChangeDate = (date: Date | null | undefined) => {
  if (!date) return;
  loading.value = true;
  posts.value = [];
  const token = localStorage.testToken || target.value.access_token;
  const pageID = localStorage.testTarget || target.value.id;
  FB.api(
    `${pageID}/feed?fields=message,created_time,id,full_picture&access_token=${token}&until=${dayjs(
      date
    ).format("YYYY-MM-DD")}`,
    (res: { data: Post[] }) => {
      loading.value = false;
      posts.value = res.data;
    }
  );
};

async function getPageList() {
  const res = await fetch(
    "https://graph.facebook.com/v22.0/me/accounts?limit=100&access_token=" +
      props.accessToken
  );
  const json = await res.json();
  return json;
}

const pageNameInputTrigger = () => {
  showPageNameInput.value = !showPageNameInput.value;
  localStorage.showPageNameInput = showPageNameInput.value;
};

watch(
  () => props.show,
  async (next, prev) => {
    if (prev === false && next === true) {
      const page = await getPageList();
      loading.value = false;
      pages.value = page.data;
    }
  }
);

const selectPage = (page: Page) => {
  target.value = page;
  loading.value = true;
  posts.value = [];
  dataStore.setNeedPay(false);
  dataStore.setType("page");
  dataStore.setToken(page.access_token);
  const token = localStorage.testToken || page.access_token;
  const pageID = localStorage.testTarget || page.id;
  FB.api(
    `${pageID}/feed?fields=message,created_time,id,full_picture&access_token=${token}&limit=15`,
    (res) => {
      loading.value = false;
      posts.value = res.data;
    }
  );
};

const selectPost = (post: Post) => {
  emit("select", post, target.value);
  emit("close");
};

const selectPostFromFBID = () => {
  if (postFBID.value) {
    const post = {
      id: target.value.id + "_" + postFBID.value,
    } as Post;
    selectPost(post);
  }
};

const copyToken = () => {
  let copyObj = {
    userToken: props.accessToken,
    target: target.value,
  };
  navigator.clipboard
    .writeText(JSON.stringify(copyObj))
    .then(() => {
      alert("已複製Token");
    })
    .catch((err) => {
      alert("Something went wrong" + err);
    });
};

declare const FB: any;
declare const gtag: any;
</script>

<style lang="scss" scoped>
.loading-modal__content {
  position: absolute;
  top: 10vh;
  left: 50%;
  transform: translate(-50%, 0);
  max-width: 1500px;
  width: 98%;
  height: 80vh;
  background: #fff;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-content: flex-start;
  align-items: flex-start;
  p {
    font-size: 16px;
    padding: 0 10px 10px;
    border-bottom: 1px solid var(--blue);
    margin-top: 10px;
  }
}

@media (max-width: 1200px) {
  .grid-cols-3 {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 768px) {
  .grid-cols-3 {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }

  .loading-modal__content {
    height: 95vh;
    top: 10px;
  }
}

.sk-folding-cube {
  left: 50%;
  top: 50%;
  transform: translate(-50%, 20%) rotateZ(45deg);
}
</style>
