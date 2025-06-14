<template>
  <div class="form-control">
    <div class="input-group">
      <input
        v-model="sn"
        type="text"
        class="rounded-none w-full input-sm pl-2 border input-bordered max-w-xs"
        :placeholder="props.text !== '' ? props.text : '請輸入序號'"
        :disabled="props.text !== '' ? true : false"
      />
      <button @click="signUp" :disabled="ajaxing" class="btn btn-blue btn-sm">
        {{ ajaxing ? "檢查序號中..." : "授權" }}
      </button>
    </div>
  </div>
</template>
<script lang="ts" setup>
import { useDataStore } from "@/store/modules/data";
const dataStore = useDataStore();
const props = defineProps({
  text: {
    type: String,
    default: "",
  },
});
const sn = ref("");
const ajaxing = ref(false);
const FBLogin = () => {
  FB.login(
    function (response: any) {
      dataStore.setUser({
        id: response.authResponse.userID,
        name: response.authResponse.userID,
      });
      signUp();
    },
    {
      auth_type: "rerequest",
      scope: "",
      return_scopes: true,
    }
  );
};
const signUp = async () => {
  ajaxing.value = true;
  // console.log(dataStore.userFbName, dataStore.userFbId);
  if (dataStore.userFbId === "") {
    alert("無法確認身分，請先完成授權");
    FBLogin();
    return;
  } else {
    FB.api(`/me?fields=id,name`, async (res) => {
      const user = {
        id: res.id,
        name: res.name,
      };
      dataStore.setUser(user);
      const body = {
        token: sn.value === "" ? "-1" : sn.value,
        username: dataStore.userFbName,
        app_scope_id: dataStore.userFbId,
      };
      const signRequest = await fetch(
        `https://script.google.com/macros/s/AKfycbzrtUqld8v4IQYjegA6XxmRTYZwLi5Hlkz0dhTBEBYdh5CAFQ8/exec`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8",
            accept: "*/*",
          },
          mode: "cors",
          body: new URLSearchParams(body).toString(),
        }
      );
      const signResult = await signRequest.json();
      if (signResult.code == 1) {
        alert(signResult.message);
        dataStore.setIsPayedUser(true);
      } else {
        alert(signResult.message + "\n" + JSON.stringify(body));
        ajaxing.value = false;
      }
    });
  }
};
</script>
