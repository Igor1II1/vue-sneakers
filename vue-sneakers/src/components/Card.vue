<script setup>
 defineProps({
  title: String,
  imageUrl: String,
  price: Number,
  isFavorite: Boolean,
  isAdded: Boolean,
  onClickAdd: Function,
  onClickFavorite: Function,
})

const base = import.meta.env.BASE_URL
</script>

<template>
  <div
    class="relative bg-white border border-slate-100 rounded-3xl p-8 cursor-pointer hover:-translate-y-2 transition hover:shadow-xl"
  >
    <img
      v-if="onClickFavorite"
      :src="!isFavorite ? `${base}like-1.svg` : `${base}like-2.svg`"
      alt="like 1"
      class="absolute top-8 left-8"
      @click="onClickFavorite"
    />
    <img  :src="imageUrl" alt="sneaker" />

    <p class="mt-2">{{ title }}</p>

    <div class="flex justify-between mt-5">
      <div class="flex flex-col">
        <span class="text-slate-400">Цена: </span>
        <b>{{ price }} руб.</b>
      </div>
      <img v-if="onClickAdd" @click="onClickAdd" :src="!isAdded ? `${base}plus.svg` : `${base}checked.svg`" alt="plus" />
    </div>
  </div>
</template>
