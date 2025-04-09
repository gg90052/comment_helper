<template>
  <div class="bg-white border p-4 mb-2 flex justify-center flex-wrap">
    <!-- <button class="btn btn-sm btn-blue" @click="fbInit">
      已經付費？按此登入
    </button> -->
    <br /><br />
    <p class="mb-2 w-full">
      抓分享、社團抓留言為付費功能，詳情請看：<a
        class="text-blue-500"
        href="https://gg90052.github.io/comment_helper/pay.html"
        target="_blank"
        >付費說明</a
      >
    </p>
    <SnInput :text="isChecking ? '檢查授權中...' : ''"></SnInput>
  </div>
</template>
<script lang="ts" setup>
import SnInput from "@/components/SnInput.vue";
import { useDataStore } from "@/store/modules/data";
const dataStore = useDataStore();
const isChecking = ref(false);

// function fbInit() {
//   FB.login(
//     function (response: any) {
//       if (response.status === "connected") {
//         isChecking.value = true;
//         fetch(
//           `https://script.google.com/macros/s/AKfycbzrtUqld8v4IQYjegA6XxmRTYZwLi5Hlkz0dhTBEBYdh5CAFQ8/exec?id=${response.authResponse.userID}`
//         )
//           .then((response) => {
//             return response.json();
//           })
//           .then((result) => {
//             dataStore.setLoginStatus(result);
//           });
//         FB.api(`/me?fields=id,name`, async (res) => {
//           const user = {
//             id: res.id,
//             name: res.name,
//           };
//           dataStore.setUser(user);
//         });
//       }
//     },
//     {
//       auth_type: "rerequest",
//     }
//   );
// }
</script>
