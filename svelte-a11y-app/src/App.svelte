<script>
  import Header from './components/Header.svelte';
  import ProductDetail from './components/ProductDetail.svelte';
  import Footer from './components/Footer.svelte';

  let cartCount = $state(1);

  function handleAddToCart() {
    cartCount++;
  }
</script>

<div class="store-app-root">
  <Header {cartCount} />
  <ProductDetail onAddToCart={handleAddToCart} />
  <Footer />
</div>
