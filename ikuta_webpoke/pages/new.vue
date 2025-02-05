<script setup>
const router = useRouter();
const config = useRuntimeConfig();
const trainerName = ref("");
const safeTrainerName = computed(() => trimAvoidCharacters(trainerName.value));
const valid = computed(() => safeTrainerName.value.length > 0);

// 決定ボタン
const onSubmit = async () => {  
  // トレーナーの追加
  const response = await $fetch("/api/trainer", {
    baseURL: config.public.backendOrigin,
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      name: safeTrainerName.value,
    }),
  }).catch((e) => e);
  if (response instanceof Error) return;
  
  // 追加したトレーナー画面に遷移
  router.push(`/trainer/${safeTrainerName.value}`);
};
const { dialog, onOpen, onClose } = useDialog();
</script>

<template>
  <div>
    <h1>あたらしくはじめる</h1>
    <p>あなたの名前を教えてね！</p>
    <form @submit.prevent>
      <div class="item">
        <label for="name">名前</label>
        <input
          id="name"
          v-model="trainerName"
          aria-describedby="name-description"
          @keydown.enter="valid && onOpen(true)"
        />
        <span id="name-description"
          >※特定の文字は取り除かれます。</span
        >
      </div>
      <GamifyButton type="button" :disabled="!valid" @click="onOpen(true)"
        >けってい</GamifyButton
      >
    </form>
    
    <GamifyDialog
      v-if="dialog"
      id="confirm-submit"
      :description="`あなたの名前は　${safeTrainerName}　でよろしいですか？`"
      @close="onClose"
    >
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="onClose">いいえ</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onSubmit">はい</GamifyButton>
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>    

    <NuxtLink :to="`/`" class="mainlink">スタート画面に戻る</NuxtLink>
  </div>
</template>

<style scoped>
form {
  border-radius: 0.5rem;
  border: solid 4px #555;
  padding: 1.5rem 3rem;
}

form > :not(:last-child) {
  display: block;
  margin-bottom: 1rem;
}

.item > label,
.item > span {
  display: block;
  margin-bottom: 0.25rem;
}
.item > span {
  font-size: 0.875rem;
}

.mainlink {
  display: flex;
  justify-content: flex-end;
}

</style>
