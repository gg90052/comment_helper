<template>
  <div class="flex justify-center content-center items-center w-full h-[100dvh] fixed bg-black/[.8] left-0 top-0 z-[99999]">
    <div class="card w-96 bg-neutral text-neutral-content">
      <div class="card-body items-center text-center">
        <h2 class="card-title">{{ props.data.error_user_title }}</h2>
        <p>Code: {{ props.data.code }}</p>
        <p>subode: {{ props.data.error_subcode }}</p>
        <p>message: {{ props.data.message }}</p>
        <p>{{ props.data.error_user_msg }}</p>
        <div class="card-actions justify-end">
          <button class="btn btn-primary" @click="showError(false, {})">確認</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
const showError = inject('globalModal') as (show, error) => void;
const props = defineProps({
  data: {
    type: Object,
    required: true
  }
});
</script>