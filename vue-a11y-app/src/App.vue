<script setup>
import { ref } from 'vue';
import Header from './components/Header.vue';
import ProductDetail from './components/ProductDetail.vue';
import Footer from './components/Footer.vue';

const cartCount = ref(1);

const handleAddToCart = () => {
  cartCount.value++;
};
</script>

<template>
  <div class="store-app-root">
    <Header :cart-count="cartCount" />
    <ProductDetail @add-to-cart="handleAddToCart" />
    <Footer />
  </div>
</template>
