<script setup>
const { data: trainers } = await useTrainers();
</script>

<template>
  <div>
    <h1>つづきからはじめる</h1>
    <p>トレーナーを選択してね！</p>
    <GamifyList>
      <GamifyItem v-for="trainer in trainers" :key="trainer">
        <NuxtLink :to="`/trainer/${trainer}`">{{ trainer }}</NuxtLink>
      </GamifyItem>
    </GamifyList>
    <NuxtLink :to="`/`" class="mainlink">スタート画面に戻る</NuxtLink>
  </div>
</template>

<style scoped>
.mainlink {
  display: flex;
  justify-content: flex-end;
}
</style>