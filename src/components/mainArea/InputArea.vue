<template>
  <div
    class="flex-grow flex-shrink-0 max-w-lg text-left mobile:text-center mobile:max-w-full mobile:w-full"
  >
    <div class="source">
      <div class="mt-4 flex flex-col mr-8">
        <slot></slot>
        <!-- <button
          class="fbloginbtn btn btn-sm mr-8 mobile:block mobile:mx-auto"
          :class="post.id === '' ? 'btn-blue' : 'btn-outline'"
          @click="fbInit"
        >
        </button> -->
        <button
          class="py-2 font-bold my-2 rounded-lg mobile:block mobile:mx-auto btn-blue"
          @click="fbInit"
        >
          <span class="text-xl">從粉絲專頁選擇貼文</span> <br />Select post from
          fanpage
        </button>
        <input
          type="hidden"
          v-model="overwrite_token"
          class="w-2/3 input-sm border"
          placeholder="請輸入TOKEN"
        />
        <div>
          <a
            target="_blank"
            class="text-pink-700 text-sm"
            href="https://app-sorteos.com/en/apps/facebook-comment-picker
"
            >Rafflys by AppSorteos 網站</a
          >
          <p class="text-sm" v-if="tokenUpdateTime">
            Token更新時間：{{ formatTime(tokenUpdateTime) }}
          </p>
        </div>
      </div>
    </div>
    <div
      class="mt-4 flex flex-nowrap items-end mobile:justify-center"
      v-if="post.id !== ''"
    >
      <button
        class="bg-blue-500 hover:bg-blue-400 transition-colors px-3 py-1 text-white font-bold rounded-md btn-free-height mr-2 flex flex-col items-center"
        @click="getData('comments')"
      >
        <svg class="svg-icon fill-current w-10 mx-2" viewBox="0 0 20 20">
          <path
            d="M14.999,8.543c0,0.229-0.188,0.417-0.416,0.417H5.417C5.187,8.959,5,8.772,5,8.543s0.188-0.417,0.417-0.417h9.167C14.812,8.126,14.999,8.314,14.999,8.543 M12.037,10.213H5.417C5.187,10.213,5,10.4,5,10.63c0,0.229,0.188,0.416,0.417,0.416h6.621c0.229,0,0.416-0.188,0.416-0.416C12.453,10.4,12.266,10.213,12.037,10.213 M14.583,6.046H5.417C5.187,6.046,5,6.233,5,6.463c0,0.229,0.188,0.417,0.417,0.417h9.167c0.229,0,0.416-0.188,0.416-0.417C14.999,6.233,14.812,6.046,14.583,6.046 M17.916,3.542v10c0,0.229-0.188,0.417-0.417,0.417H9.373l-2.829,2.796c-0.117,0.116-0.71,0.297-0.71-0.296v-2.5H2.5c-0.229,0-0.417-0.188-0.417-0.417v-10c0-0.229,0.188-0.417,0.417-0.417h15C17.729,3.126,17.916,3.313,17.916,3.542 M17.083,3.959H2.917v9.167H6.25c0.229,0,0.417,0.187,0.417,0.416v1.919l2.242-2.215c0.079-0.077,0.184-0.12,0.294-0.12h7.881V3.959z"
          ></path>
        </svg>
        抓留言<br />Get comments
      </button>
      <button
        class="bg-blue-500 hover:bg-blue-400 transition-colors px-3 py-1 text-white font-bold rounded-md btn-free-height mr-2 flex flex-col items-center"
        @click="getData('reactions')"
      >
        <svg class="svg-icon fill-current w-10 mx-2" viewBox="0 0 20 20">
          <path
            d="M9.719,17.073l-6.562-6.51c-0.27-0.268-0.504-0.567-0.696-0.888C1.385,7.89,1.67,5.613,3.155,4.14c0.864-0.856,2.012-1.329,3.233-1.329c1.924,0,3.115,1.12,3.612,1.752c0.499-0.634,1.689-1.752,3.612-1.752c1.221,0,2.369,0.472,3.233,1.329c1.484,1.473,1.771,3.75,0.693,5.537c-0.19,0.32-0.425,0.618-0.695,0.887l-6.562,6.51C10.125,17.229,9.875,17.229,9.719,17.073 M6.388,3.61C5.379,3.61,4.431,4,3.717,4.707C2.495,5.92,2.259,7.794,3.145,9.265c0.158,0.265,0.351,0.51,0.574,0.731L10,16.228l6.281-6.232c0.224-0.221,0.416-0.466,0.573-0.729c0.887-1.472,0.651-3.346-0.571-4.56C15.57,4,14.621,3.61,13.612,3.61c-1.43,0-2.639,0.786-3.268,1.863c-0.154,0.264-0.536,0.264-0.69,0C9.029,4.397,7.82,3.61,6.388,3.61"
          ></path>
        </svg>
        抓按讚<br />Get reactions
      </button>
      <div class="text-center">
        <a
          href="https://www.facebook.com/commenthelper/posts/pfbid02oSyRNGea2PA7rLPThHRE8BHjXDXBFoeu7D2a92j68buLEC2u63Xex3GypMkqtoXql"
          target="_blank"
          class="text-blue-400 text-sm"
          >如何抓分享</a
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { getNextAPI } from "@/api/api";
import { useDataStore } from "@/store/modules/data";
import dayjs from "dayjs";
const dataStore = useDataStore();
const emit = defineEmits(["fbLogged", "showLoading"]);

onMounted(() => {
  window.addEventListener("importShare", function (e: Event) {
    const customEvent = e as any;
    importShare(customEvent.detail.data);
  });
  window.addEventListener("importComment", function (e: Event) {
    // console.log(e);
    const customEvent = e as any;
    importComment(customEvent.detail.data);
  });
  window.addEventListener("facebookTokensUpdated", (e: Event) => {
    // console.log(e);
    const customEvent = e as any;
    tokenUpdateTime.value = customEvent.detail.timestamp;
    localStorage.setItem("facebookTokens", JSON.stringify(customEvent.detail));
  });
  const facebookTokens = localStorage.getItem("facebookTokens");
  if (facebookTokens) {
    tokenUpdateTime.value = JSON.parse(facebookTokens).timestamp;
  }
});

const fbResponse = ref(undefined);
const tokenUpdateTime = ref(undefined);
const post = ref({ id: "" });
const page = ref<{ access_token: string; name?: string }>({ access_token: "" });
const overwrite_token = ref("");
const fields = {
  comments: [
    "like_count",
    "message_tags",
    "attachment",
    "message",
    "from{id,name}",
    "created_time",
  ],
  reactions: ["type", "name"],
};
let rawData = [];

function fbInit() {
  if (fbResponse.value !== undefined) {
    emit("fbLogged", fbResponse.value, true);
    return;
  }
  emit("showLoading");
  // const scope = "pages_show_list, pages_read_engagement, pages_read_user_content, groups_access_member_info, business_management";
  const scope =
    "pages_show_list, pages_read_engagement, pages_read_user_content, business_management";
  FB.login(
    function (response: any) {
      fbResponse.value = response;
      localStorage.setItem("lastLoggedUser", response.authResponse.userID);
      emit("fbLogged", response);
    },
    {
      auth_type: "rerequest",
      scope: scope,
      return_scopes: true,
    }
  );
}

const importShare = (shareData) => {
  // shareData = shareTestData;
  localStorage.sharedposts = JSON.stringify(shareData);
  alert("匯入完成");
  dataStore.setCommand("shares");
  dataStore.setRawData(shareData);
  dataStore.setNeedPay(true);
  // dataStore.setNeedPay(false);
};
const importComment = (commentData, needPay = true) => {
  localStorage.commentPosts = JSON.stringify(commentData);
  console.log(commentData);
  alert("匯入完成");
  dataStore.setCommand("import_comments");
  dataStore.setRawData(commentData);
  // console.log(commentData);
  dataStore.setNeedPay(needPay);
};

const getData = async (command: string) => {
  emit("showLoading");
  rawData = [];
  dataStore.setCommand(command);
  const storedToken = localStorage.getItem("facebookTokens")
    ? JSON.parse(localStorage.getItem("facebookTokens") || "")
    : undefined;
  if (storedToken) {
    const pageName = page.value.name;
    const overwriteToken = storedToken?.tokens.find(
      (token) => token.name === pageName
    )?.access_token;
    overwrite_token.value = overwriteToken;
  }

  FB.api(
    `/${post.value.id}/${command}`,
    {
      fields: fields[command].join(","),
      limit: 100,
      order: "chronological",
      access_token:
        overwrite_token.value !== ""
          ? overwrite_token.value
          : page.value.access_token,
    },
    (res: any) => {
      if (res.data) {
        rawData = rawData.concat(res.data);
        if (res.paging && res.paging.next) {
          getNext(res.paging.next);
        } else {
          finishFetch();
        }
      } else {
        alert(res.error.message);
        emit("showLoading", false);
      }
    }
  );
};

const getNext = async (url) => {
  const res = await getNextAPI(url);
  rawData = rawData.concat(res.data);
  if (res.paging.next) {
    getNext(res.paging.next);
  } else {
    finishFetch();
  }
};

const finishFetch = () => {
  if (rawData.length === 0) {
    alert("沒有資料");
  } else {
    console.log(rawData);
    dataStore.setRawData(rawData);
  }
  emit("showLoading", false);
};

const getPost = (target: any, target_page: any) => {
  post.value = target;
  page.value = target_page;
};

const isMobile = () => {
  return /Mobi|Android|iPhone/i.test(navigator.userAgent);
};

watch(overwrite_token, (newVal) => {
  if (newVal !== "") {
    sessionStorage.overwriteToken = newVal;
  }
});

// 將時間戳轉換為易讀格式
function formatTime(timestamp: number | string) {
  if (!timestamp) return "";
  return dayjs(timestamp).format("YYYY年MM月DD日 HH:mm:ss");
}

defineExpose({
  getPost,
});

declare const FB: any;
</script>
<style scoped lang="scss">
.fbloginbtn {
  width: 350px;
  height: 76px;
  background: url("@/assets/login.jpg") center center no-repeat;
  background-size: contain;
  border: none;
}
</style>
