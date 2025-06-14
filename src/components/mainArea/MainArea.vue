<template>
  <div
    class="flex flex-nowrap items-start py-4 px-3 mb-2 bg-white mobile:flex-wrap"
  >
    <Ci class="mobile:w-full"></Ci>
    <InputArea
      ref="inputArea"
      @fbLogged="fbCallback"
      @showLoading="loadingDisplay"
      class="ml-8 mobile:ml-0"
    >
      <div>
        <p v-if="checkingVIP">確認用戶身分中...</p>
        <p v-if="!checkingVIP && dataStore.payedUser === false">
          免費用戶 -
          <a
            class="text-blue-500"
            target="_blank"
            href="https://gg90052.github.io/comment_helper/pay.html"
            >付費說明</a
          >
        </p>
        <p v-if="!checkingVIP && dataStore.payedUser === true">已付費用戶</p>
      </div>
    </InputArea>
    <div
      class="w-full max-w-[560px] mobile:mt-4 mobile:mx-auto"
      v-if="!dataStore.postData.hasOwnProperty('id')"
    >
      <YtVideo ref="ytRef" />
    </div>
    <PostPreview ref="postPreview" />
    <LoadingModal :show="showLoading"></LoadingModal>
    <PageSelector
      :show="showPageSelector"
      :accessToken="accessToken"
      @select="selectPost"
      @close="showPageSelector = false"
    ></PageSelector>
  </div>
</template>

<script setup lang="ts">
import Ci from "./Ci.vue";
import InputArea from "./InputArea.vue";
import PostPreview from "./PostPreview.vue";
import LoadingModal from "../LoadingModal.vue";
import YtVideo from "./YtVideo.vue";
import PageSelector from "../PageSelector.vue";
import { useDataStore } from "@/store/modules/data";
const dataStore = useDataStore();

const showLoading = ref(false);
const showPageSelector = ref(false);
const accessToken = ref("");
const inputArea = ref();
const postPreview = ref();
const ytRef = ref();
const checkingVIP = ref(false);

const loadingDisplay = (show: boolean = true) => {
  showLoading.value = show;
};

onMounted(() => {
  const lastLoggedUser = localStorage.getItem("lastLoggedUser");
  if (lastLoggedUser) {
    checkVIP(lastLoggedUser);
  }
});
const checkVIP = (lastLoggedUser: string) => {
  try {
    checkingVIP.value = true;
    fetch(
      `https://script.google.com/macros/s/AKfycbzrtUqld8v4IQYjegA6XxmRTYZwLi5Hlkz0dhTBEBYdh5CAFQ8/exec?id=${lastLoggedUser}`
    )
      .then((response) => {
        return response.json();
      })
      .then((result) => {
        dataStore.setIsPayedUser(result);
        checkingVIP.value = false;
      });
  } catch (e) {
    console.log(e);
  }
};

function fbCallback(response: any, passVIPLogin: boolean = false) {
  console.log(response);
  if (passVIPLogin === true) {
    showSelector();
    return;
  }
  if (response.status === "connected") {
    accessToken.value = response.authResponse.accessToken;
    try {
      fetch(
        `https://script.google.com/macros/s/AKfycbzrtUqld8v4IQYjegA6XxmRTYZwLi5Hlkz0dhTBEBYdh5CAFQ8/exec?id=${response.authResponse.userID}`
      )
        .then((response) => {
          return response.json();
        })
        .then((result) => {
          checkVIP(response.authResponse.userID);
          showSelector();
        });
    } catch (e) {
      console.log(e);
    }

    FB.api(`/me?fields=id,name,permissions`, async (res) => {
      const user = {
        id: res.id,
        name: res.name,
      };
      dataStore.setUser(user);
    });
  }
}
const showSelector = () => {
  showLoading.value = false;
  showPageSelector.value = true;
};

const selectPost = (post: any, page: any) => {
  inputArea.value.getPost(post, page);
  if (ytRef.value) {
    ytRef.value.setVideoShow(false);
  }
  FB.api(
    `/${post.id}`,
    {
      fields: "shares,permalink_url,full_picture,message",
      access_token: page.access_token,
    },
    (res: any) => {
      dataStore.setPostData(res);
    }
  );
};
declare const FB: any;
</script>
