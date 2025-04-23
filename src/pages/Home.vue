<script setup>
import axios from 'axios'
import { useFavorites } from '@/composables/useFavorites'
import { inject, reactive, watch, ref, onMounted } from 'vue'
import debounce from 'lodash.debounce'

import CardList from '../components/CardList.vue'

const { cart, addToCart, removeFromCart } = inject('cart')
const { addToFavorite } = useFavorites()

const items = ref([])

const filters = reactive({
  sortBy: 'title',
  searchQuery: '',
})

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://eee984114c1a0c3e.mokky.dev/favorites`)

    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id)

      if (!favorite) {
        return item
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      }
    })
  } catch (err) {
    console.log(err)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy,
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    const { data } = await axios.get(`https://34a43f60562e814c.mokky.dev/items`, {
      params,
    })

    items.value = data.map((obj) => {
      const existingItem = items.value.find((item) => item.id === obj.id)

      return {
        ...obj,
        isFavorite: existingItem?.isFavorite || false,
        favoriteId: existingItem?.favoriteId || null,
        isAdded: existingItem?.isAdded || false,
      }
    })
  } catch (err) {
    console.log(err)
  }
}

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 300)

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((CartItem) => CartItem.id === item.id),
  }))
})

watch(filters, fetchItems)

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false,
  }))
})
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold">Все кроссовки</h2>

    <div class="flex gap-4">
      <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
        <option value="name">По названию</option>
        <option value="price">По цене (дешевые)</option>
        <option value="-price">По цене (дорогие)</option>
      </select>

      <div class="relative">
        <img class="absolute left-4 top-3" src="/search.svg" alt="" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          type="text"
          placeholder="Поиск..."
        />
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </div>
</template>
