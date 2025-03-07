<script setup>
const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();
const { data: trainer, refresh } = await useFetch(
  () => `/api/trainer/${route.params.name}`,
  {
    default: () => [],
    server: false,
    baseUrl: config.public.backendOrigin,
  },
);


// トレーナー削除
const onDelete = async () => {
  const response = await $fetch(`/api/trainer/${route.params.name}`, {
    baseURL: config.public.backendOrigin,
    method: "DELETE",
  }).catch((e) => e);
  if (response instanceof Error) return;
  router.push("/");
};

// ポケモンのニックネーム変更
const nickname = ref("");
const onNickname = async (pokemon) => {
  const newTrainer = trainer.value;
  const index = newTrainer.pokemons.findIndex(({ id }) => id === pokemon.id);
  newTrainer.pokemons[index].nickname = trimAvoidCharacters(nickname.value);
  nickname.value = "";
  const response = await $fetch(`/api/trainer/${route.params.name}`, {
    baseURL: config.public.backendOrigin,
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(newTrainer),
  }).catch((e) => e);
  if (response instanceof Error) return;
  await refresh();
  onCloseNickname();
};

// ポケモン削除
const onRelease = async (pokemonId) => {
  const response = await fetch(
    `/api/trainer/${route.params.name}/pokemon/${pokemonId}`,
    {
      baseURL: config.public.backendOrigin,
      method: "DELETE",
    },
  ).catch((e) => e);
  if (response instanceof Error) return;
  await refresh();
  onCloseRelease();
};
const {
  dialog: deleteDialog,
  onOpen: onOpenDelete,
  onClose: onCloseDelete,
} = useDialog();
const {
  dialog: nicknameDialog,
  onOpen: onOpenNickname,
  onClose: onCloseNickname,
} = useDialog();
const {
  dialog: releaseDialog,
  onOpen: onOpenRelease,
  onClose: onCloseRelease,
} = useDialog();
const {
  dialog: failureDialog,
  onOpen: onOpenFailure,
  onClose: onCloseFailure,
} = useDialog();
</script>

<template>
  <div>
    <h2>トレーナー名 『{{ trainer.name }}』</h2>
    <CatchButton :to="`/trainer/${route.params.name}/catch`"
      >ポケモンをつかまえる</CatchButton
    >
    <h2>てもちポケモン</h2>
    <GamifyList>
      <GamifyItem v-for="pokemon in trainer.pokemons" :key="pokemon.id">
        <img :src="pokemon.sprites.front_default" />
        <span class="pokemon-name">{{ pokemon.nickname || pokemon.name }}</span>
        <GamifyButton @click="onOpenNickname(pokemon)"
          >ニックネームをつける</GamifyButton
        >
        <GamifyButton @click="onOpenRelease(pokemon)"
          >ポケモンを逃がす</GamifyButton
        >
      </GamifyItem>
    </GamifyList>
    <GamifyButton @click="onOpenDelete(true)"
      >トレーナーをやめる</GamifyButton
    >
    <GamifyDialog
      v-if="deleteDialog"
      id="confirm-delete"
      title="【警告】"
      description="　　本当にトレーナをやめますか？この操作は取り消せません。"
      @close="onCloseDelete"
    >
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="onCloseDelete">いいえ</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onDelete">はい</GamifyButton>
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>
    
    <GamifyDialog
      v-if="nicknameDialog"
      id="confirm-nickname"
      :description="`${nicknameDialog.name}　のニックネームをつけてください。`"
      @close="onCloseNickname"
    >
      <div class="item">
        <label for="name">ニックネーム</label>
        <input
          id="name"
          v-model="nickname"
          @keydown.enter="onNickname(nicknameDialog)"
        />
      </div>
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="onCloseNickname">キャンセル</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onNickname(nicknameDialog)"
            >けってい</GamifyButton
          >
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>
    
    <GamifyDialog
      v-if="releaseDialog"
      id="confirm-release"
      title="【警告】"
      :description="`　　本当に　${
        releaseDialog.nickname || releaseDialog.name
      }　を逃がしますか？この操作は取り消せません。`"
      @close="onCloseRelease"
    >
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="onCloseRelease">いいえ</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onRelease(releaseDialog.id)">はい</GamifyButton>
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>  
    
    
    <NuxtLink :to="`/`" class="mainlink">スタート画面に戻る</NuxtLink>
  </div>
</template>

<style scoped>
.item > label {
  display: block;
  margin-bottom: 0.25rem;
}

.gamify-item:hover img {
  animation: bounce;
  animation-duration: 0.8s;
  animation-iteration-count: infinite;
}

.trainer-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 3rem;
}

.trainer-info > img {
  width: 2rem;
  height: 2rem;
}

.mainlink {
  display: flex;
  justify-content: flex-end;
}
</style>
