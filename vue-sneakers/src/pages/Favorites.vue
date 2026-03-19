<script setup>
import { ref, onMounted } from 'vue'
import CardList from '../components/CardList.vue'

const favorites = ref([])

onMounted(async () => {
  try {
    const savedFavorites = JSON.parse(localStorage.getItem('favorites') || '[]')
    const res = await fetch(`${import.meta.env.BASE_URL}items.json`)
    const allItems = await res.json()
    favorites.value = allItems.filter(item =>
      savedFavorites.some(f => f.item_id === item.id)
    )
  } catch (err) {
    console.log(err)
  }
})
</script>



<template>
    <div>
    <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>
    <CardList :items="favorites" is-favorites />
    </div>
</template>
