<script setup>
const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();
const page = ref(0);
const limit = ref(10);
const range = ref(10); // 最後に1匹だけ表示するために、表示範囲は別に設定
const offset = computed(() => page.value * limit.value);
const { data: pokemons, refresh } = await useFetch(
  () =>
    `https://pokeapi.co/api/v2/pokemon?offset=${offset.value}&limit=${range.value}`,
  {
    default: () => [],
    server: false,
  },
);
const hasPrev = computed(() => page.value > 0);
// const maxPage = computed(() => Math.floor(pokemons.value.count / limit.value));
// 初代のみ
const maxPage = computed(() => Math.floor(151 / limit.value));
const hasNext = computed(() => page.value < maxPage.value);
const onPrev = async () => {
  page.value--;
  // 表示範囲切替
  if(page.value < 15){
    range.value = 10;
  }
  await refresh();
};

const onNext = async () => {
  page.value++;
  // 表示範囲切替
  if(page.value > 14){
    range.value = 1;
  }
  await refresh();
};

const onCatch = async (pokemon) => {
  const response = await $fetch(`/api/trainer/${route.params.name}/pokemon`, {
    baseURL: config.public.backendOrigin,
    method: "POST",
    body: {
      name: pokemon.name,
    },
  }).catch((e) => e);
  if (response instanceof Error) return;
  router.push(`/trainer/${route.params.name}`);
};
const { dialog, onOpen, onClose } = useDialog();
</script>

<template>
  <div>
    <h1>ポケモンをつかまえる</h1>
    <NuxtLink :to="`/trainer/${route.params.name}`" class="mainlink">つかまえるのをやめる</NuxtLink>
    <GamifyList>
      <GamifyItem v-for="(pokemon, index) in pokemons.results" :key="pokemon.url">
        <p>No.{{ ('000' + (offset + index + 1)).slice(-3) }}　</p>
        <span class="pokemon-name">{{ pokemon.name }}</span>
        <GamifyButton @click="onOpen(pokemon)">つかまえる</GamifyButton>
      </GamifyItem>
    </GamifyList>
    <GamifyDialog
      v-if="dialog"
      id="confirm-catch"
      :description="`${dialog.name}　をつかまえますか？`"
      @close="onClose"
    >
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="onClose">いいえ</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onCatch(dialog)">はい</GamifyButton>
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>
    <p v-if="range==1">ポケモンNo.：{{ ('000' + (offset + 1)).slice(-3)}}　(全151種類)</p>
    <p v-else>ポケモンNo.：{{ ('000' + (offset + 1)).slice(-3) }} ～ {{ ('000' + (offset + range)).slice(-3) }}　(全151種類)</p>
    <GamifyList direction="horizon">
      <GamifyItem>
        <GamifyButton :disabled="!hasPrev" @click="onPrev">まえへ</GamifyButton>
      </GamifyItem>
      <GamifyItem>
        <GamifyButton :disabled="!hasNext" @click="onNext">つぎへ</GamifyButton>
      </GamifyItem>
    </GamifyList>
  </div>
</template>

<style scoped>
.mainlink {
  display: flex;
  justify-content: flex-end;
}
</style>