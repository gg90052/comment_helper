<template>
  <div class="flex flex-nowrap items-start py-4 px-3 mb-2 bg-white mobile:flex-wrap">
    <Ci class="mobile:w-full"></Ci>
    <InputArea ref="inputArea" @fbLogged="fbCallback" @showLoading="loadingDisplay" class="ml-8 mobile:ml-0"></InputArea>
    <div class="w-full max-w-[560px] mobile:mt-4 mobile:mx-auto">
      <YtVideo ref="ytRef" />
    </div>
    <PostPreview ref="postPreview" />
    <LoadingModal :show="showLoading"></LoadingModal>
    <PageSelector :show="showPageSelector" :accessToken="accessToken" @select="selectPost" @close="showPageSelector = false"></PageSelector>
  </div>
</template>

<script setup lang="ts">
import Ci from './Ci.vue';
import InputArea from './InputArea.vue';
import PostPreview from './PostPreview.vue'
import LoadingModal from '../LoadingModal.vue';
import YtVideo from './YtVideo.vue';
import PageSelector from '../PageSelector.vue';
import { useDataStore } from '@/store/modules/data';
const dataStore = useDataStore();

const showLoading = ref(false);
const showPageSelector = ref(false);
const accessToken = ref('');
const inputArea = ref();
const postPreview = ref();
const payChecked = ref(false);
const ytRef = ref();


const loadingDisplay = (show: boolean = true) => {
  showLoading.value = show;
}

function fbCallback(response: any, type:string){
  console.log(response);
  if (response.status === 'connected') {
    accessToken.value = response.authResponse.accessToken;
    fetch(`https://script.google.com/macros/s/AKfycbzrtUqld8v4IQYjegA6XxmRTYZwLi5Hlkz0dhTBEBYdh5CAFQ8/exec?id=${response.authResponse.userID}`).then(response=>{
      return response.json();
    }).then(result=>{
      dataStore.setLoginStatus(result);
      showSelector();
    });
    FB.api(`/me?fields=id,name`, async (res) => {
      const user = {
        id: res.id,
        name: res.name,
      }
      dataStore.setUser(user);
    });
  }
}
const showSelector = () => {
  showLoading.value = false;
  showPageSelector.value = true;
}

const selectPost = (post: any, page: any) => {
  inputArea.value.getPost(post, page);
  ytRef.value.setVideoShow(false);
  FB.api(`/${post.id}`, {
    fields: 'shares,permalink_url,attachments,message',
    access_token: page.access_token,
  }, (res: any)=>{
    dataStore.setPostData(res);
  });
}
</script>
