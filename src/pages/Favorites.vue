<script setup>
import { inject, onMounted } from 'vue'
import { useFavorites } from '@/composables/useFavorites'

import CardList from '../components/CardList.vue'

const { favorites, fetchFavorites, addToFavorite } = useFavorites()
const { addToCart, removeFromCart } = inject('cart')

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

onMounted(async () => {
  await fetchFavorites()
})
</script>

<template>
  <h1 class="text-3xl fond-bold mb-8">Мои закладки</h1>

  <CardList
    :items="favorites"
    is-favorites
    @add-to-favorite="addToFavorite"
    @add-to-cart="onClickAddPlus"
  />
</template>
