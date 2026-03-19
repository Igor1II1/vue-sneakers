<script setup>
import { reactive, watch, ref, onMounted } from 'vue'
import CardList from '../components/CardList.vue'
import debounce from 'lodash.debounce'
import { inject } from 'vue'

const { cart, addToCart, removeFromCart} = inject('cart')

const items = ref([])


const filters = reactive({
  sortBy: 'title',
  searchQuery: '',
})


// добавление и удаление товара из массива
const onClickAddPlus = (item)=>{
 if(!item.isAdded){
addToCart(item)
 }else{
  removeFromCart(item)
 }
}

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

// отвечает за поиск
const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
},300)


const addToFavorite = (item)=>{
  const favorites = JSON.parse(localStorage.getItem('favorites') || '[]')
  if(!item.isFavorite){
    item.isFavorite = true
    item.favoriteId = Date.now()
    favorites.push({ id: item.favoriteId, item_id: item.id })
  } else {
    item.isFavorite = false
    const idx = favorites.findIndex(f => f.item_id === item.id)
    if (idx !== -1) favorites.splice(idx, 1)
    item.favoriteId = null
  }
  localStorage.setItem('favorites', JSON.stringify(favorites))
}


  onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

 

  // запрос всех кросовок
  await fetchItems()
  // получение всех закладок
  fetchFavorites()

   // при перезагрузке страницы сохранять зеленые галочки
  items.value = items.value.map((item)=>({
    ...item,
    isAdded:cart.value.some((CartItem) => CartItem.id ===item.id)
  }))
})


// watch — это функция во Vue, 
// которая позволяет следить за изменениями какой-то 
// переменной (reactive или ref) и выполнять код, когда она меняется.
// Проще говоря:
// watch = “слежка за данными: если они изменились — сделай что-то”
watch(cart, ()=>{
  items.value = items.value.map((item)=>({
    ...item,
    isAdded: false
  }))
})



const fetchFavorites = () => {
  const favorites = JSON.parse(localStorage.getItem('favorites') || '[]')
  items.value = items.value.map((item) => {
    const favorite = favorites.find((f) => f.item_id === item.id)
    if (!favorite) {
      return item
    }
    return {
      ...item,
      isFavorite: true,
      favoriteId: favorite.id,
    }
  })
}


let allItems = []

const fetchItems = async () => {
  try {
    if (allItems.length === 0) {
      const res = await fetch('/items.json')
      allItems = await res.json()
    }

    let filtered = [...allItems]

    if (filters.searchQuery) {
      const q = filters.searchQuery.toLowerCase()
      filtered = filtered.filter(item => item.title.toLowerCase().includes(q))
    }

    if (filters.sortBy === 'price') {
      filtered.sort((a, b) => a.price - b.price)
    } else if (filters.sortBy === '-price') {
      filtered.sort((a, b) => b.price - a.price)
    } else {
      filtered.sort((a, b) => a.title.localeCompare(b.title))
    }

    items.value = filtered.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false,
    }))
  } catch (err) {
    console.log(err)
  }
}


watch(filters, fetchItems)

</script>



<template>
    <div>
    <div class="flex justify-between items-center">
        <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>
        <div class="flex gap-4">
          <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
            <option value="name">По названию</option>
            <option value="price">По цене (дешевые)</option>
            <option value="-price">По цене (дорогие)</option>
          </select>

          <div class="relative">
            <img class="absolute left-3 top-3" src="/search.svg" />
            <input
              @input="onChangeSearchInput"
              class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
              placeholder="Поиск..."
            />
          </div>
        </div>
      </div>

      <div class="mt-10">
        <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
      </div>
      </div>
</template>