<template>
  <div class="form-control">
    <div class="input-group">
      <input
        v-model="pagename"
        type="text"
        class="rounded-none w-full input-sm pl-2 border input-bordered max-w-xs"
        placeholder="請輸入粉絲專頁網址名稱"
      />
      <button @click="getToken" :disabled="ajaxing" class="btn btn-blue btn-sm">
        {{ ajaxing ? "取得token中..." : "取得粉絲專頁貼文" }}
      </button>
    </div>
    <a
      class="block text-blue-500 hover:text-yellow-500"
      href="https://www.facebook.com/commenthelper/posts/pfbid02csZcDMCpsT66Cn5AP9xBfeRrPSuB9Ljm9brD1txEmuWKc8zUxebUaQojxQeZoiV8l"
      target="_blank"
      >如何使用？</a
    >
  </div>
</template>
<script lang="ts" setup>
const emit = defineEmits(["selectPage"]);
const pagename = ref("");
const ajaxing = ref(false);
const showError = inject("globalModal") as (show, error) => void;
const getToken = async () => {
  if (pagename.value === "") {
    alert("請輸入粉絲團網址名稱");
    return;
  }
  ajaxing.value = true;
  FB.api(`${pagename.value}/?fields=name,access_token`, (res) => {
    if (!res || res.error) {
      if (res.error.code === 100) {
        // not found
        res.error.message = "找不到粉絲專頁";
        showError(true, res.error);
      }
      if (res.error.code === 10) {
        // no power
        res.error.message = "沒有權限存取粉絲專頁";
        showError(true, res.error);
      }
    } else {
      emit("selectPage", res);
      console.log(res);
    }
    ajaxing.value = false;
  });
};
</script>
