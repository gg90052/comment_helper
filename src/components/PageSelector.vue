<template>
  <div class="loading-modal text-left" v-if="show">
    <div class="loading-modal__content">
      <div
        class="absolute top-2 right-0 w-10 text-2xl text-gray-500 cursor-pointer"
        @click="emit('close')"
      >
        X
      </div>
      <div
        class="w-full flex-shrink-0 border-r border-black h-full overflow-auto"
        v-if="phase === 1"
      >
        <template v-if="!dataStore.logged">
          <div class="mt-10 px-4">
            <SnInput></SnInput>
          </div>
        </template>
        <p>粉絲專頁 Fanpages</p>
        <div class="select_page">
          <div class="pl-4 py-3">
            <PageNameInput @selectPage="(page) => selectPage(page)" />
          </div>
          <div
            class="cursor-pointer"
            v-for="(page, index) in pages"
            :key="index"
            @click="selectPage(page)"
          >
            <h3 class="pl-4 py-3 hover:bg-sky-500 hover:text-white">
              {{ page.name }}
            </h3>
          </div>
        </div>
        <p class="page_group">社團 Groups</p>
        <div class="select_group">
          <div
            class="cursor-pointer"
            v-for="(group, index) in groups"
            :key="index"
            @click="selectGroup(group)"
          >
            <h3 class="pl-4 py-3 hover:bg-sky-500 hover:text-white">
              {{ group.name }}
            </h3>
          </div>
        </div>
      </div>
      <div class="flex-grow overflow-auto h-full pb-3 px-4" v-if="phase === 2">
        <p class="flex flex-start flex-wrap">
          <button
            @click="phase = 1"
            class="btn bg-green-600 hover:bg-green-400 border-none btn-sm -mt-[3px] align-middle mr-4"
          >
            回到選擇粉絲專頁/社團
          </button>
          <span class="mr-4 my-2">{{ target.name }}貼文列表 </span>
          <button
            class="btn-free-height btn-warning h-8 align-middle text-xs"
            @click="copyToken"
          >
            複製token
          </button>
        </p>
        <div v-if="loading" class="sk-folding-cube">
          <div class="sk-cube1 sk-cube"></div>
          <div class="sk-cube2 sk-cube"></div>
          <div class="sk-cube4 sk-cube"></div>
          <div class="sk-cube3 sk-cube"></div>
        </div>
        <div class="live_post mt-4">
          <div class="flex flex-start flex-wrap" v-if="target.name !== ''">
            <span>貼文截止時間：</span>
            <datepicker
              @update:modelValue="onChangeDate"
              inputFormat="yyyy-MM-dd"
              class="pl-2 border inline-block"
              v-model="until"
            />
            <div class="form-control ml-8 mobile:ml-0 mobile:mt-4">
              <div class="input-group rounded-none">
                <input
                  v-model="postFBID"
                  type="text"
                  class="rounded-none w-full input-sm pl-2 border input-bordered max-w-xs"
                  placeholder="請輸入貼文FBID"
                />
                <button @click="selectPostFromFBID" class="btn btn-blue btn-sm">
                  用FBID選擇貼文
                </button>
              </div>
            </div>
          </div>
        </div>
        <div class="flex flex-start flex-wrap" v-if="posts.length > 0">
          <div
            class="w-full max-w-[340px] border-2 px-4 py-2 m-4 max-h-60 rounded-lg mobile:mx-0 mobile:my-2"
            v-for="(post, index) in posts"
            :key="index"
          >
            <a
              class="text-blue-700 hover:underline hover:text-blue-500 block max-h-32 overflow-auto break-words"
              :href="`https://www.facebook.com/${post.id}`"
              target="_blank"
              >{{ post.story || post.message }}</a
            >
            <p class="!border-none !text-sm text-right !mt-2">
              {{ dayjs(post.created_time).format("YYYY-MM-DD HH:mm:ss") }}
            </p>
            <button
              class="btn-blue h-8 align-middle block w-full"
              @click="selectPost(post)"
            >
              選擇貼文 Select Post
            </button>
          </div>
        </div>
        <!-- <table v-if="posts.length > 0" class="table-fixed border-collapse border border-gray-400 mt-4">
					<thead>
            <tr>
              <th class="border py-2 border-gray-300"></th>
              <th class="border py-2 border-gray-300"></th>
              <th class="border py-2 border-gray-300">貼文內容</th>
              <th class="border py-2 border-gray-300">貼文時間</th>
            </tr>
          </thead>
					<tbody>
            <tr v-for="(post, index) in posts" :key="index">
              <td class="text-center w-[120px] border border-gray-300">
                <button class="btn-free-height btn-blue h-8 align-middle text-xs" @click="selectPost(post)">選擇貼文</button>
              </td>
              <td class="border border-gray-300 w-8 text-center">{{ index + 1 }}</td>
              <td class="border break-all border-gray-300 max-w-lg px-2 py-4"><a class="text-blue-700 hover:underline hover:text-blue-500" :href="`https://www.facebook.com/${post.id}`" target="_blank">{{ post.story || post.message }}</a></td>
              <td class="border border-gray-300 w-[200px] text-center">{{ dayjs(post.created_time).format('YYYY-MM-DD HH:mm:ss') }}</td>
            </tr>
          </tbody>
				</table> -->
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import * as dayjs from "dayjs";
import Datepicker from "vue3-datepicker";
import { useDataStore } from "@/store/modules/data";
import SnInput from "@/components/SnInput.vue";
import PageNameInput from "@/components/PageNameInput.vue";
const dataStore = useDataStore();
const emit = defineEmits(["close", "select"]);
const props = defineProps(["show", "accessToken"]);
const showError = inject("globalModal") as (show, error) => void;
const phase = ref(1);
const loading = ref(true);
const target = ref({ name: "" });
const pages = ref([]);
const groups = ref([]);
const posts = ref([]);
const until = ref(new Date());
const postFBID = ref("");
const onChangeDate = (date) => {
  loading.value = true;
  posts.value = [];
  if (dataStore.postType === "page") {
    const token = localStorage.testToken || target.value.access_token;
    const pageID = localStorage.testTarget || target.value.id;
    FB.api(
      `${pageID}/published_posts?access_token=${token}&until=${dayjs(
        date
      ).format("YYYY-MM-DD")}`,
      (res) => {
        loading.value = false;
        posts.value = res.data;
      }
    );
  } else {
    const token = localStorage.testToken || props.accessToken;
    const groupID = localStorage.testTarget || target.value.id;
    FB.api(
      `${groupID}/feed?access_token=${token}&until=${dayjs(date).format(
        "YYYY-MM-DD"
      )}&limit=15`,
      (res) => {
        loading.value = false;
        posts.value = res.data;
      }
    );
  }
};

const getGroupList = async () => {
  const res = await fetch(
    `https://graph.facebook.com/v14.0/me/groups?admin_only=true&fields=name,id&limit=100&access_token=${props.accessToken}`
  );
  const json = await res.json();
  return json;
};

async function getPageList() {
  const res = await fetch(
    "https://graph.facebook.com/v14.0/me/accounts?limit=100&access_token=" +
      props.accessToken
  );
  const json = await res.json();
  return json;
}
watch(
  () => props.show,
  async (next, prev) => {
    if (prev === false && next === true) {
      phase.value = 1;
      const page = await getPageList();
      // console.log(page);
      const group = await getGroupList();
      // console.log(group)
      loading.value = false;
      pages.value = page.data;
      groups.value = group.data;
    }
  }
);

const selectPage = (page) => {
  phase.value = 2;
  target.value = page;
  gtag("event", "pageSelect", { pageName: target.value.name });
  loading.value = true;
  posts.value = [];
  dataStore.setNeedPay(false);
  dataStore.setType("page");
  dataStore.setToken(page.access_token);
  const token = localStorage.testToken || page.access_token;
  const pageID = localStorage.testTarget || page.id;
  FB.api(`${pageID}/published_posts?access_token=${token}&limit=15`, (res) => {
    loading.value = false;
    posts.value = res.data;
  });
};

const selectGroup = (group) => {
  phase.value = 2;
  target.value = group;
  gtag("event", "groupSelect", { groupName: target.value.name });
  loading.value = true;
  posts.value = [];
  dataStore.setNeedPay(true);
  dataStore.setType("group");
  dataStore.setToken(props.accessToken);
  const token = localStorage.testToken || props.accessToken;
  const groupID = localStorage.testTarget || group.id;
  FB.api(`${groupID}/feed?access_token=${token}`, (res) => {
    if (res.error) {
      showError(true, res.error);
      phase.value = 1;
      return;
    } else {
      loading.value = false;
      posts.value = res.data;
    }
  });
};

const selectPost = (post) => {
  emit("select", post, target.value);
  emit("close");
  target.value = { name: "" };
  posts.value = [];
  postFBID.value = "";
};
const selectPostFromFBID = () => {
  if (postFBID.value) {
    const post = {
      id: target.value.id + "_" + postFBID.value,
    };
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
</script>

<style lang="scss" scoped>
.loading-modal__content {
  position: absolute;
  top: 10vh;
  left: 50%;
  transform: translate(-50%, 0);
  max-width: 1200px;
  width: 100%;
  height: 80vh;
  background: #fff;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-content: flex-start;
  align-items: flex-start;
  p {
    color: var(--blue);
    font-size: 20px;
    padding: 0 10px 10px;
    border-bottom: 1px solid var(--blue);
    margin-top: 30px;
  }
}

.sk-folding-cube {
  left: 50%;
  top: 50%;
  transform: translate(-50%, 20%) rotateZ(45deg);
}
</style>
