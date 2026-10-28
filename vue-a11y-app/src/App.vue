<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import axe from 'axe-core';
import { PRODUCTS, CATEGORIES, COMPARISON_PRODUCTS, STORE_FAQS } from './data/products.js';
import { WCAG_BUGS } from './data/wcagBugs.js';

// Accessibility State (default false = Inaccessible for testing)
const isAccessible = ref(false);
const highlightIssues = ref(false);

// Active View Routing: 'home' | 'products' | 'product-detail'
const currentPage = ref('home');
const selectedProduct = ref(PRODUCTS[0]);

// Category & Catalog Filtering
const selectedCategory = ref('All');
const searchQuery = ref('');
const sortBy = ref('featured');
const inStockOnly = ref(false);

// Selected color & quantity on Product Detail Page
const detailColor = ref(PRODUCTS[0].colors[0]?.name || 'Standard');
const detailQuantity = ref(1);
const detailTab = ref('overview');

// FAQ Accordion State
const openFaqId = ref(null);
const toggleFaq = (id) => {
  openFaqId.value = openFaqId.value === id ? null : id;
};

// Cart State
const cartItems = ref([
  {
    id: PRODUCTS[0].id,
    name: PRODUCTS[0].name,
    price: PRODUCTS[0].price,
    image: PRODUCTS[0].image,
    selectedColor: 'Matte Black',
    quantity: 1,
  },
]);
const discountPercent = ref(0);
const couponInput = ref('');
const couponFeedback = ref('');

// Wishlist
const wishlistIds = ref(['prod-1', 'prod-4']);

// Modals
const isCartOpen = ref(false);
const isCheckoutOpen = ref(false);
const isBugExplorerOpen = ref(false);
const isLiveAxeOpen = ref(false);
const isScanningAxe = ref(false);
const axeResults = ref(null);
const categoryDropdownOpen = ref(false);

// Focus Tracker (Keyboard testing assistant)
const focusedInfo = ref(null);

// Toast alerts
const toasts = ref([]);
const addToast = (msg, type = 'info') => {
  const id = Date.now() + Math.random();
  toasts.value.push({ id, message: msg, type });
  setTimeout(() => {
    toasts.value = toasts.value.filter((t) => t.id !== id);
  }, 4000);
};
const removeToast = (id) => {
  toasts.value = toasts.value.filter((t) => t.id !== id);
};

// Navigation
const navigate = (page, cat = null) => {
  if (cat) selectedCategory.value = cat;
  currentPage.value = page;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const openProductDetail = (prod) => {
  selectedProduct.value = prod;
  detailColor.value = prod.colors?.[0]?.name || 'Standard';
  detailQuantity.value = 1;
  currentPage.value = 'product-detail';
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

// Filtered Products
const filteredProducts = computed(() => {
  let list = [...PRODUCTS];
  if (selectedCategory.value !== 'All') {
    list = list.filter((p) => p.category === selectedCategory.value);
  }
  if (inStockOnly.value) {
    list = list.filter((p) => p.stockStatus === 'in-stock');
  }
  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase();
    list = list.filter(
      (p) =>
        p.name.toLowerCase().includes(q) ||
        p.category.toLowerCase().includes(q) ||
        p.description.toLowerCase().includes(q)
    );
  }
  if (sortBy.value === 'price-low') list.sort((a, b) => a.price - b.price);
  else if (sortBy.value === 'price-high') list.sort((a, b) => b.price - a.price);
  else if (sortBy.value === 'rating') list.sort((a, b) => b.rating - a.rating);
  return list;
});

// Cart Totals
const cartTotalCount = computed(() => cartItems.value.reduce((sum, i) => sum + i.quantity, 0));
const cartSubtotal = computed(() => cartItems.value.reduce((sum, i) => sum + i.price * i.quantity, 0));
const cartDiscount = computed(() => (cartSubtotal.value * discountPercent.value) / 100);
const cartShipping = computed(() => (cartSubtotal.value > 50 || cartSubtotal.value === 0 ? 0 : 9.99));
const cartGrandTotal = computed(() => Math.max(0, cartSubtotal.value - cartDiscount.value + cartShipping.value));

const addToCart = (product, color = 'Standard', qty = 1) => {
  const existing = cartItems.value.find((i) => i.id === product.id && i.selectedColor === color);
  if (existing) {
    existing.quantity += qty;
  } else {
    cartItems.value.push({
      id: product.id,
      name: product.name,
      price: product.price,
      image: product.image,
      selectedColor: color,
      quantity: qty,
    });
  }
  addToast(`Added ${qty}x "${product.name}" (${color}) to cart!`, 'success');
};

const updateCartQty = (id, color, delta) => {
  const item = cartItems.value.find((i) => i.id === id && i.selectedColor === color);
  if (item) {
    item.quantity += delta;
    if (item.quantity <= 0) {
      cartItems.value = cartItems.value.filter((i) => !(i.id === id && i.selectedColor === color));
      addToast('Item removed from cart.', 'info');
    }
  }
};

const removeFromCart = (id, color) => {
  cartItems.value = cartItems.value.filter((i) => !(i.id === id && i.selectedColor === color));
  addToast('Item removed from cart.', 'info');
};

const toggleWishlist = (id) => {
  if (wishlistIds.value.includes(id)) {
    wishlistIds.value = wishlistIds.value.filter((x) => x !== id);
    addToast('Removed item from wishlist.', 'info');
  } else {
    wishlistIds.value.push(id);
    addToast('Saved item to wishlist!', 'info');
  }
};

const applyCoupon = () => {
  if (couponInput.value.trim().toUpperCase() === 'A11Y20') {
    discountPercent.value = 20;
    couponFeedback.value = '✅ 20% discount applied successfully!';
  } else {
    couponFeedback.value = '❌ Invalid code. Use "A11Y20"';
  }
};

// Checkout Form State
const checkoutForm = ref({
  fullName: '',
  email: '',
  address: '',
  city: '',
  zip: '',
  cardNumber: '',
});
const checkoutErrors = ref({});
const checkoutSuccess = ref(false);

const handleCheckoutSubmit = () => {
  const errs = {};
  if (!checkoutForm.value.fullName.trim()) errs.fullName = 'Full Name is required';
  if (!checkoutForm.value.email.trim()) errs.email = 'Email Address is required';
  if (!checkoutForm.value.address.trim()) errs.address = 'Street Address is required';
  if (!checkoutForm.value.city.trim()) errs.city = 'City is required';
  if (!checkoutForm.value.zip.trim()) errs.zip = 'Postal / Zip code is required';
  if (!checkoutForm.value.cardNumber.trim()) errs.cardNumber = 'Card number is required';

  if (Object.keys(errs).length > 0) {
    checkoutErrors.value = errs;
    return;
  }
  checkoutSuccess.value = true;
  cartItems.value = [];
  discountPercent.value = 0;
  addToast('Order successfully confirmed! Thank you.', 'success');
};

// Live Axe Audit
const runAxe = async () => {
  isScanningAxe.value = true;
  try {
    const res = await axe.run(document.body, {
      runOnly: {
        type: 'tag',
        values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'best-practice'],
      },
    });
    axeResults.value = res;
    isLiveAxeOpen.value = true;
  } catch (e) {
    console.error('Axe error:', e);
  } finally {
    isScanningAxe.value = false;
  }
};

// Focus Tracker listener
const handleFocusIn = (e) => {
  const el = e.target;
  if (!el || el === document.body) {
    focusedInfo.value = null;
    return;
  }
  const tag = el.tagName.toLowerCase();
  const role = el.getAttribute('role') || 'implicit';
  const id = el.id ? `#${el.id}` : '';
  const label = el.getAttribute('aria-label') || el.innerText?.slice(0, 30) || (el.placeholder ? `Placeholder: "${el.placeholder}"` : 'None');
  focusedInfo.value = {
    tag,
    id,
    role,
    label,
    hasNoName: !el.getAttribute('aria-label') && !el.innerText?.trim() && !el.title,
  };
};

onMounted(() => {
  window.addEventListener('focusin', handleFocusIn);
});
onUnmounted(() => {
  window.removeEventListener('focusin', handleFocusIn);
});
</script>

<template>
  <div :class="['store-app', isAccessible ? 'mode-accessible' : 'mode-inaccessible']">
    <!-- Skip link in accessible mode -->
    <a v-if="isAccessible" href="#main-store-page-content" class="skip-link">
      Skip to main page content
    </a>


    <!-- Announcement Bar -->
    <div
      :class="['announcement-bar', isAccessible ? 'announcement-accessible' : 'announcement-inaccessible', highlightIssues && !isAccessible ? 'a11y-flagged-border' : '']"
      :role="isAccessible ? 'region' : undefined"
      :aria-label="isAccessible ? 'Promotions' : undefined"
    >
      <div class="announcement-content">
        <span>📢 ⚡ Flash Sale: Use code <strong>A11Y20</strong> for 20% off all acoustic hardware! Free 2-Day Shipping on $50+</span>
        <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro" title="WCAG 1.4.3: Low contrast text (2.1:1 ratio, failing AA 4.5:1)">
          [Low Contrast]
        </span>
      </div>
    </div>

    <!-- Header / Navbar -->
    <nav
      :class="['store-navbar', isAccessible ? 'nav-accessible' : 'nav-inaccessible']"
      :role="isAccessible ? 'navigation' : undefined"
      :aria-label="isAccessible ? 'Main Store Navigation' : undefined"
    >
      <div class="nav-container">
        <!-- Logo -->
        <div class="nav-brand">
          <button type="button" class="brand-link-btn" @click="navigate('home')">
            <span class="brand-logo-icon">⚡</span>
            <span class="brand-name">AuraStore</span>
          </button>
        </div>

        <!-- Pages Nav -->
        <div class="nav-page-links">
          <button
            type="button"
            :class="['nav-page-link-btn', currentPage === 'home' ? 'active' : '']"
            @click="navigate('home')"
          >
            Home
          </button>
          <button
            type="button"
            :class="['nav-page-link-btn', currentPage === 'products' ? 'active' : '']"
            @click="navigate('products')"
          >
            Products
          </button>
        </div>

        <!-- Categories Dropdown (Hover vs Button) -->
        <div class="nav-categories">
          <div v-if="isAccessible" class="accessible-dropdown">
            <button
              type="button"
              class="category-trigger-btn"
              :aria-expanded="categoryDropdownOpen"
              aria-haspopup="true"
              @click="categoryDropdownOpen = !categoryDropdownOpen"
            >
              <span>Category: <strong>{{ selectedCategory }}</strong></span>
              <span class="dropdown-caret">▾</span>
            </button>
            <ul v-if="categoryDropdownOpen" class="category-dropdown-menu" role="menu">
              <li v-for="cat in CATEGORIES" :key="cat" role="none">
                <button
                  type="button"
                  :class="['category-item-btn', selectedCategory === cat ? 'active' : '']"
                  @click="selectedCategory = cat; navigate('products'); categoryDropdownOpen = false"
                >
                  {{ cat }}
                </button>
              </li>
            </ul>
          </div>
          <div v-else class="hover-only-dropdown">
            <div class="hover-trigger-fake">
              <span>Categories ▾</span>
              <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Hover Only]</span>
            </div>
            <div class="hover-menu-pane">
              <div
                v-for="cat in CATEGORIES"
                :key="cat"
                :class="['hover-item', selectedCategory === cat ? 'active' : '']"
                @click="selectedCategory = cat; navigate('products')"
              >
                {{ cat }}
              </div>
            </div>
          </div>
        </div>

        <!-- Search Bar -->
        <div class="nav-search-wrapper">
          <form v-if="isAccessible" class="search-form" role="search" @submit.prevent="navigate('products')">
            <label for="vue-search-input" class="visually-hidden">Search products in catalog</label>
            <input
              id="vue-search-input"
              v-model="searchQuery"
              type="search"
              class="search-input"
              placeholder="Search headphones, gear..."
              @input="currentPage !== 'products' && navigate('products')"
            />
            <button type="submit" class="search-btn" aria-label="Submit search">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
            </button>
          </form>
          <div v-else class="search-form-inaccessible">
            <input
              v-model="searchQuery"
              type="text"
              class="search-input"
              placeholder="Search products..."
              @input="currentPage !== 'products' && navigate('products')"
            />
            <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[No Label]</span>
            <button type="button" class="search-btn" @click="navigate('products')">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
              <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[No Name]</span>
            </button>
          </div>
        </div>

        <!-- Actions -->
        <div class="nav-actions">
          <button
            type="button"
            class="action-icon-btn"
            :aria-label="isAccessible ? `Wishlist, ${wishlistIds.length} items` : undefined"
            @click="navigate('products'); addToast(`You have ${wishlistIds.length} saved items!`, 'info')"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/></svg>
            <span class="action-pill-count">{{ wishlistIds.length }}</span>
            <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro">[No Label]</span>
          </button>

          <button
            v-if="isAccessible"
            type="button"
            class="action-cart-btn"
            :aria-label="`Shopping cart, ${cartTotalCount} items`"
            @click="isCartOpen = true"
          >
            <span>🛒 Cart</span>
            <span class="cart-count-badge">{{ cartTotalCount }}</span>
          </button>
          <div
            v-else
            class="action-cart-btn fake-div-btn"
            @click="isCartOpen = true"
          >
            <span>🛒 Cart</span>
            <span class="cart-count-badge">{{ cartTotalCount }}</span>
            <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[&lt;div&gt; Button]</span>
          </div>
        </div>
      </div>
    </nav>

    <!-- Main Content Area -->
    <main id="main-store-page-content">
      <!-- 1. HOME PAGE VIEW -->
      <div v-if="currentPage === 'home'" class="home-page">
        <!-- Hero Banner -->
        <section class="hero-banner">
          <div class="hero-grid">
            <div class="hero-content">
              <div class="hero-badge-wrapper">
                <span :class="['hero-deal-pill', isAccessible ? 'deal-pill-accessible' : 'deal-pill-inaccessible']">
                  🔥 2026 AUDIO INNOVATION WINNER
                </span>
                <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro">[Low Contrast]</span>
              </div>
              <h1 class="hero-title">
                Pure Acoustic Precision.<br />
                <span class="hero-gradient-text">Zero Compromise.</span>
              </h1>
              <p class="hero-subtitle">
                Lossless spatial audio with 40mm beryllium drivers and adaptive 45dB noise cancellation.
              </p>
              <div class="hero-actions">
                <button
                  type="button"
                  class="btn-primary hero-cta-btn"
                  @click="navigate('products')"
                >
                  {{ isAccessible ? 'Explore All Hardware →' : 'Click Here' }}
                </button>
                <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro">[Ambiguous Text]</span>
                <div class="hero-coupon-chip">
                  <span>Coupon:</span>
                  <strong class="coupon-code">A11Y20</strong>
                  <span class="coupon-discount">(Save 20%)</span>
                </div>
              </div>
            </div>
            <div class="hero-visual">
              <div class="hero-image-frame">
                <img
                  src="https://images.unsplash.com/photo-1546435770-a3e426bf472b?w=900&auto=format&fit=crop&q=80"
                  :alt="isAccessible ? 'Aura Studio headphones displayed on minimalist wooden pedestal' : ''"
                  class="hero-img"
                />
                <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-top-left">[Missing Alt]</span>
              </div>
            </div>
          </div>
        </section>

        <div class="home-content-container">
          <!-- Featured Categories -->
          <section class="home-section">
            <div class="section-header-row">
              <div>
                <span class="section-eyebrow">EXPLORE HARDWARE</span>
                <h2 class="section-main-heading">Featured Categories</h2>
              </div>
              <button type="button" class="btn-secondary" @click="navigate('products')">
                View Full Catalog →
              </button>
            </div>
            <div class="categories-grid">
              <div
                v-for="cat in [
                  { name: 'Audio', icon: '🎧', count: '18 Items', desc: 'Over-ear headphones and studio monitors' },
                  { name: 'Wearables', icon: '⌚', count: '12 Items', desc: 'Heart rate and sleep tracking smartwatches' },
                  { name: 'Computing', icon: '⌨️', count: '14 Items', desc: 'Mechanical split keyboards and 4K webcams' },
                  { name: 'Accessories', icon: '⚡', count: '22 Items', desc: 'Foldable MagSafe chargers and laptop risers' },
                ]"
                :key="cat.name"
                class="category-card"
                @click="selectedCategory = cat.name; navigate('products')"
              >
                <div class="category-card-icon">{{ cat.icon }}</div>
                <h3 class="category-card-title">{{ cat.name }}</h3>
                <span class="category-card-count">{{ cat.count }}</span>
                <p class="category-card-desc">{{ cat.desc }}</p>
                <span class="category-card-link">Shop {{ cat.name }} →</span>
              </div>
            </div>
          </section>

          <!-- Top Products Grid Preview -->
          <section class="home-section">
            <div class="section-header-row">
              <div>
                <span class="section-eyebrow">BEST SELLERS</span>
                <h2 class="section-main-heading">Featured Audiophile Gear</h2>
              </div>
              <button type="button" class="btn-secondary" @click="navigate('products')">
                Browse All ({{ PRODUCTS.length }})
              </button>
            </div>
            <div class="products-grid">
              <article
                v-for="prod in PRODUCTS.slice(0, 4)"
                :key="prod.id"
                class="product-card"
              >
                <div class="product-card-media">
                  <img
                    :src="prod.image"
                    :alt="isAccessible ? prod.altAccessible : prod.altInaccessible"
                    class="product-card-img"
                  />
                  <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-top-left">[Bad Alt]</span>
                  <span v-if="prod.badge" class="product-badge badge-accessible">{{ prod.badge }}</span>
                </div>
                <div class="product-card-body">
                  <div class="product-meta-row">
                    <span class="product-card-category">{{ prod.category }}</span>
                    <!-- Stock Color Only Issue in Flawed Mode -->
                    <span v-if="isAccessible" :class="['stock-status', prod.stockStatus === 'low-stock' ? 'stock-low' : 'stock-ok']">
                      ● {{ prod.stockStatus === 'low-stock' ? `Only ${prod.stockCount} left` : 'In Stock' }}
                    </span>
                    <div v-else class="stock-indicator-inaccessible">
                      <span :class="['stock-dot-only', prod.stockStatus === 'low-stock' ? 'dot-red' : 'dot-green']"></span>
                      <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Color Only]</span>
                    </div>
                  </div>
                  <h3 class="product-card-title">{{ prod.name }}</h3>
                  <div class="product-price-row">
                    <span class="product-price-current">${{ prod.price.toFixed(2) }}</span>
                    <span v-if="prod.originalPrice" class="product-price-original">${{ prod.originalPrice.toFixed(2) }}</span>
                  </div>
                  <div class="product-card-actions">
                    <button type="button" class="btn-secondary card-quick-btn" @click="openProductDetail(prod)">
                      {{ isAccessible ? 'Product Details' : 'Details' }}
                    </button>
                    <button type="button" class="btn-primary card-cart-btn" @click="addToCart(prod)">
                      Add to Cart
                    </button>
                  </div>
                </div>
              </article>
            </div>
          </section>

          <!-- Comparison Table (div vs table) -->
          <section class="comparison-section">
            <div class="section-intro">
              <span class="section-eyebrow">BENCHMARK MATRIX</span>
              <h2 class="section-heading">Aura vs Leading Competitors</h2>
            </div>
            <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-banner">
              [WCAG 1.3.1: Table built with &lt;div&gt; flexbox; missing table semantics for screen readers]
            </span>
            <div v-if="isAccessible" class="table-responsive-wrapper">
              <table class="comparison-table">
                <caption class="visually-hidden">Headphone acoustic hardware comparison</caption>
                <thead>
                  <tr>
                    <th scope="col">Hardware Feature</th>
                    <th scope="col" class="col-aura highlight-header">Aura SoundMax Pro</th>
                    <th scope="col">Competitor A</th>
                    <th scope="col">Competitor B</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(row, i) in COMPARISON_PRODUCTS" :key="i">
                    <th scope="row" class="row-feature-header">{{ row.feature }}</th>
                    <td class="highlight-cell"><strong>{{ row.aura }}</strong></td>
                    <td>{{ row.competitorA }}</td>
                    <td>{{ row.competitorB }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div v-else class="fake-div-table">
              <div class="fake-table-row fake-header-row">
                <div class="fake-cell fake-col-feature">Hardware Feature</div>
                <div class="fake-cell fake-col-aura">Aura SoundMax Pro</div>
                <div class="fake-cell">Competitor A</div>
                <div class="fake-cell">Competitor B</div>
              </div>
              <div v-for="(row, i) in COMPARISON_PRODUCTS" :key="i" class="fake-table-row">
                <div class="fake-cell fake-col-feature">{{ row.feature }}</div>
                <div class="fake-cell fake-col-aura">{{ row.aura }}</div>
                <div class="fake-cell">{{ row.competitorA }}</div>
                <div class="fake-cell">{{ row.competitorB }}</div>
              </div>
            </div>
          </section>

          <!-- FAQ Accordion -->
          <section class="faq-section">
            <div class="section-intro">
              <span class="section-eyebrow">HELP & SUPPORT</span>
              <h2 class="section-heading">Frequently Asked Questions</h2>
            </div>
            <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-banner">
              [WCAG 2.4.3 & 4.1.2: &lt;div&gt; headers lacking aria-expanded • Ghost links in collapsed panels]
            </span>
            <div class="faq-list">
              <div v-for="faq in STORE_FAQS" :key="faq.id" class="faq-card">
                <h3 v-if="isAccessible">
                  <button
                    type="button"
                    class="faq-trigger-btn"
                    :aria-expanded="openFaqId === faq.id"
                    @click="toggleFaq(faq.id)"
                  >
                    <span>{{ faq.question }}</span>
                    <span>{{ openFaqId === faq.id ? '▲' : '▼' }}</span>
                  </button>
                </h3>
                <div v-else class="faq-trigger-fake-div" @click="toggleFaq(faq.id)">
                  <span>{{ faq.question }}</span>
                  <span>{{ openFaqId === faq.id ? '▲' : '▼' }}</span>
                  <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Div Header]</span>
                </div>

                <div v-if="isAccessible" :hidden="openFaqId !== faq.id" class="faq-panel-accessible">
                  <p class="faq-answer-text">{{ faq.answer }}</p>
                  <a v-if="faq.hasLink" href="#faq" class="faq-link">{{ faq.linkText }} →</a>
                </div>
                <div v-else :class="['faq-panel-inaccessible', openFaqId === faq.id ? 'open' : 'collapsed']">
                  <p class="faq-answer-text">{{ faq.answer }}</p>
                  <a v-if="faq.hasLink" href="#faq" class="faq-link">{{ faq.linkText }} →</a>
                  <span v-if="highlightIssues && openFaqId !== faq.id" class="a11y-pill-tag tag-micro">
                    [Ghost Link in Tab Order]
                  </span>
                </div>
              </div>
            </div>
          </section>
        </div>
      </div>

      <!-- 2. PRODUCTS LIST PAGE VIEW -->
      <div v-else-if="currentPage === 'products'" class="products-page">
        <!-- Breadcrumbs -->
        <nav class="page-breadcrumbs" :aria-label="isAccessible ? 'Breadcrumb' : undefined">
          <div class="breadcrumbs-container">
            <button type="button" class="breadcrumb-link" @click="navigate('home')">Home</button>
            <span class="breadcrumb-separator">/</span>
            <span class="breadcrumb-current">Products</span>
            <span v-if="selectedCategory !== 'All'" class="breadcrumb-separator">/</span>
            <span v-if="selectedCategory !== 'All'" class="breadcrumb-current">{{ selectedCategory }}</span>
          </div>
        </nav>

        <div class="products-page-container">
          <div class="page-header-strip">
            <h1 class="page-title">
              {{ selectedCategory === 'All' ? 'All Hardware & Gear' : `${selectedCategory} Collection` }}
            </h1>
            <p class="page-description">Browse our full catalog of certified headphones, wearables, keyboards, and gear.</p>
          </div>

          <!-- Filters Toolbar -->
          <div class="filter-toolbar">
            <div class="filter-left">
              <div class="category-pills">
                <button
                  v-for="cat in CATEGORIES"
                  :key="cat"
                  type="button"
                  :class="['pill-btn', selectedCategory === cat ? 'active' : '']"
                  @click="selectedCategory = cat"
                >
                  {{ cat }}
                </button>
              </div>

              <!-- In-Stock Checkbox -->
              <div class="stock-filter-wrapper">
                <label v-if="isAccessible" class="checkbox-label" for="vue-stock-check">
                  <input id="vue-stock-check" v-model="inStockOnly" type="checkbox" class="semantic-checkbox" />
                  <span>In-stock only</span>
                </label>
                <div v-else class="fake-checkbox-wrapper" @click="inStockOnly = !inStockOnly">
                  <span :class="['fake-checkbox-box', inStockOnly ? 'checked' : '']">{{ inStockOnly ? '✓' : '' }}</span>
                  <span>In-stock only</span>
                  <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Span Checkbox]</span>
                </div>
              </div>
            </div>

            <div class="filter-right">
              <span class="product-results-count">Showing <strong>{{ filteredProducts.length }}</strong> products</span>
              <div class="sort-wrapper">
                <select v-if="isAccessible" v-model="sortBy" class="semantic-select" aria-label="Sort products by">
                  <option value="featured">Featured Picks</option>
                  <option value="price-low">Price: Low to High</option>
                  <option value="price-high">Price: High to Low</option>
                  <option value="rating">Top Customer Rated</option>
                </select>
                <div v-else class="fake-sort-dropdown">
                  <div class="fake-sort-trigger">
                    <span>{{ sortBy === 'featured' ? 'Featured Picks' : sortBy }}</span>
                    <span>▾</span>
                  </div>
                  <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Div Select]</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Product Grid -->
          <div v-if="filteredProducts.length === 0" class="empty-catalog-state">
            <h3>No products found</h3>
            <button type="button" class="btn-primary" @click="selectedCategory = 'All'; searchQuery = ''; inStockOnly = false">
              Reset Filters
            </button>
          </div>
          <div v-else class="products-grid">
            <article v-for="prod in filteredProducts" :key="prod.id" class="product-card">
              <div class="product-card-media">
                <img :src="prod.image" :alt="isAccessible ? prod.altAccessible : prod.altInaccessible" class="product-card-img" />
                <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-top-left">[Bad Alt]</span>
                <span v-if="prod.badge" class="product-badge badge-accessible">{{ prod.badge }}</span>
              </div>
              <div class="product-card-body">
                <div class="product-meta-row">
                  <span class="product-card-category">{{ prod.category }}</span>
                  <span v-if="isAccessible" :class="['stock-status', prod.stockStatus === 'low-stock' ? 'stock-low' : 'stock-ok']">
                    ● {{ prod.stockStatus === 'low-stock' ? `Only ${prod.stockCount} left` : 'In Stock' }}
                  </span>
                  <div v-else class="stock-indicator-inaccessible">
                    <span :class="['stock-dot-only', prod.stockStatus === 'low-stock' ? 'dot-red' : 'dot-green']"></span>
                    <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Color Only]</span>
                  </div>
                </div>
                <h3 class="product-card-title">{{ prod.name }}</h3>
                <div class="product-price-row">
                  <span class="product-price-current">${{ prod.price.toFixed(2) }}</span>
                  <span v-if="prod.originalPrice" class="product-price-original">${{ prod.originalPrice.toFixed(2) }}</span>
                </div>
                <div class="product-card-actions">
                  <button type="button" class="btn-secondary card-quick-btn" @click="openProductDetail(prod)">
                    {{ isAccessible ? 'Product Details' : 'Details' }}
                  </button>
                  <button type="button" class="btn-primary card-cart-btn" @click="addToCart(prod)">
                    Add to Cart
                  </button>
                </div>
              </div>
            </article>
          </div>
        </div>
      </div>

      <!-- 3. PRODUCT DETAILS PAGE VIEW -->
      <div v-else-if="currentPage === 'product-detail'" class="product-detail-page">
        <!-- Breadcrumbs -->
        <nav class="page-breadcrumbs" :aria-label="isAccessible ? 'Breadcrumb' : undefined">
          <div class="breadcrumbs-container">
            <button type="button" class="breadcrumb-link" @click="navigate('home')">Home</button>
            <span class="breadcrumb-separator">/</span>
            <button type="button" class="breadcrumb-link" @click="navigate('products')">Products</button>
            <span class="breadcrumb-separator">/</span>
            <span class="breadcrumb-current">{{ selectedProduct.name }}</span>
          </div>
        </nav>

        <div class="product-detail-container">
          <div class="detail-top-bar">
            <button type="button" class="btn-secondary back-btn" @click="navigate('products')">
              ← Back to Products Catalog
            </button>
          </div>

          <div class="detail-main-grid">
            <!-- Left Media -->
            <div class="detail-media-column">
              <div class="detail-image-box">
                <img
                  :src="selectedProduct.image"
                  :alt="isAccessible ? selectedProduct.altAccessible : selectedProduct.altInaccessible"
                  class="detail-large-img"
                />
                <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-top-left">[Bad Alt]</span>
              </div>
              <div class="detail-badges-row">
                <span class="detail-category-badge">{{ selectedProduct.category }}</span>
                <span v-if="isAccessible" :class="['stock-status', selectedProduct.stockStatus === 'low-stock' ? 'stock-low' : 'stock-ok']">
                  ● {{ selectedProduct.stockStatus === 'low-stock' ? `Urgent: ${selectedProduct.stockCount} left` : 'In Stock' }}
                </span>
                <div v-else class="stock-indicator-inaccessible">
                  <span :class="['stock-dot-only', selectedProduct.stockStatus === 'low-stock' ? 'dot-red' : 'dot-green']"></span>
                  <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Color Only]</span>
                </div>
              </div>
            </div>

            <!-- Right Info & Purchase -->
            <div class="detail-info-column">
              <h1 class="detail-product-title">{{ selectedProduct.name }}</h1>
              <div class="detail-price-strip">
                <span class="detail-current-price">${{ selectedProduct.price.toFixed(2) }}</span>
                <span v-if="selectedProduct.originalPrice" class="detail-original-price">${{ selectedProduct.originalPrice.toFixed(2) }}</span>
                <span v-if="selectedProduct.badge" class="detail-sale-badge">{{ selectedProduct.badge }}</span>
              </div>
              <p class="detail-lead-desc">{{ selectedProduct.description }}</p>

              <!-- Color Variants -->
              <div v-if="selectedProduct.colors" class="detail-options-block">
                <span class="option-heading">Color: <strong>{{ detailColor }}</strong></span>
                <div class="detail-swatches-row">
                  <button
                    v-for="c in selectedProduct.colors"
                    :key="c.name"
                    type="button"
                    :class="['detail-swatch-circle', detailColor === c.name ? 'active' : '']"
                    :style="{ backgroundColor: c.hex }"
                    :aria-label="isAccessible ? `Select ${c.name} color` : undefined"
                    @click="detailColor = c.name"
                  >
                    <span v-if="detailColor === c.name">✓</span>
                  </button>
                  <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro">[No Swatch Names]</span>
                </div>
              </div>

              <!-- Quantity -->
              <div class="detail-options-block">
                <span class="option-heading">Quantity:</span>
                <div class="quantity-stepper">
                  <button
                    v-if="isAccessible"
                    type="button"
                    class="stepper-btn"
                    aria-label="Decrease quantity"
                    @click="detailQuantity = Math.max(1, detailQuantity - 1)"
                  >
                    −
                  </button>
                  <div v-else class="stepper-btn fake-stepper-btn" @click="detailQuantity = Math.max(1, detailQuantity - 1)">−</div>
                  <span class="stepper-value">{{ detailQuantity }}</span>
                  <button
                    v-if="isAccessible"
                    type="button"
                    class="stepper-btn"
                    aria-label="Increase quantity"
                    @click="detailQuantity++"
                  >
                    +
                  </button>
                  <div v-else class="stepper-btn fake-stepper-btn" @click="detailQuantity++">+</div>
                </div>
              </div>

              <!-- Add to Cart CTA -->
              <div class="detail-cta-row">
                <button
                  type="button"
                  class="btn-primary detail-add-btn"
                  @click="addToCart(selectedProduct, detailColor, detailQuantity)"
                >
                  Add to Cart • ${{ (selectedProduct.price * detailQuantity).toFixed(2) }}
                </button>
                <button
                  type="button"
                  class="btn-secondary detail-wish-btn"
                  @click="toggleWishlist(selectedProduct.id)"
                >
                  {{ wishlistIds.includes(selectedProduct.id) ? '❤️ Saved' : '🤍 Wishlist' }}
                </button>
              </div>

              <!-- Detail Tabs -->
              <div class="detail-tabs-wrapper">
                <div v-if="isAccessible" role="tablist" class="accessible-tablist">
                  <button
                    type="button"
                    role="tab"
                    :class="['tab-btn', detailTab === 'overview' ? 'active' : '']"
                    :aria-selected="detailTab === 'overview'"
                    @click="detailTab = 'overview'"
                  >
                    Features
                  </button>
                  <button
                    type="button"
                    role="tab"
                    :class="['tab-btn', detailTab === 'specs' ? 'active' : '']"
                    :aria-selected="detailTab === 'specs'"
                    @click="detailTab = 'specs'"
                  >
                    Specifications
                  </button>
                </div>
                <div v-else class="fake-tablist">
                  <a href="#features" :class="['fake-tab', detailTab === 'overview' ? 'active' : '']" @click.prevent="detailTab = 'overview'">Features</a>
                  <a href="#specs" :class="['fake-tab', detailTab === 'specs' ? 'active' : '']" @click.prevent="detailTab = 'specs'">Specifications</a>
                  <span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[Fake Tabs]</span>
                </div>

                <div class="detail-tab-content">
                  <div v-if="detailTab === 'overview'">
                    <ul class="detail-features-list">
                      <li v-for="(f, i) in selectedProduct.features" :key="i">{{ f }}</li>
                    </ul>
                  </div>
                  <div v-if="detailTab === 'specs'">
                    <div class="detail-specs-table">
                      <div v-for="(v, k) in selectedProduct.specs" :key="k" class="spec-row">
                        <span class="spec-key">{{ k.toUpperCase() }}:</span>
                        <span class="spec-val">{{ v }}</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer :class="['store-footer', isAccessible ? 'footer-accessible' : 'footer-inaccessible']">
      <div class="footer-top-grid">
        <div class="footer-col">
          <div class="footer-logo">⚡ AuraStore Vue</div>
          <p class="footer-bio">Universal accessibility testing application across Angular, React, and Vue 3.</p>
          <div class="newsletter-box">
            <h4 class="newsletter-title">Subscribe to Insider Updates</h4>
            <div class="newsletter-input-wrap">
              <input type="text" class="newsletter-input" placeholder="Enter your email..." />
              <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro">[No Label]</span>
              <button type="button" class="btn-primary" @click="addToast('Subscribed to newsletter!', 'success')">Join</button>
            </div>
          </div>
        </div>
        <div class="footer-col">
          <h4 class="footer-col-title">Shop Hardware</h4>
          <ul class="footer-links-list">
            <li><a href="#audio" @click.prevent="selectedCategory = 'Audio'; navigate('products')">Over-Ear ANC Audio</a></li>
            <li><a href="#wearables" @click.prevent="selectedCategory = 'Wearables'; navigate('products')">Smartwatches & Sensors</a></li>
            <li><a href="#keyboards" @click.prevent="selectedCategory = 'Computing'; navigate('products')">Mechanical Keyboards</a></li>
          </ul>
        </div>
        <div class="footer-col">
          <h4 class="footer-col-title">Social Links</h4>
          <div class="footer-social-row">
            <a href="#x" class="social-icon-btn" :aria-label="isAccessible ? 'Twitter' : undefined">𝕏</a>
            <a href="#yt" class="social-icon-btn" :aria-label="isAccessible ? 'YouTube' : undefined">▶</a>
            <a href="#gh" class="social-icon-btn" :aria-label="isAccessible ? 'GitHub' : undefined">🐙</a>
            <span v-if="highlightIssues && !isAccessible" class="a11y-pill-tag tag-micro">[No Social Labels]</span>
          </div>
        </div>
      </div>
      <div class="footer-bottom-bar">
        <div :class="[isAccessible ? 'text-accessible-contrast' : 'text-low-contrast']">
          © 2026 AuraStore Vue. WCAG 2.1 / 2.2 accessibility testing environment.
        </div>
      </div>
    </footer>

    <!-- Cart Drawer -->
    <div v-if="isCartOpen" class="cart-drawer-overlay" @click.self="isCartOpen = false">
      <div class="cart-drawer-panel" :role="isAccessible ? 'dialog' : undefined" aria-label="Shopping Cart">
        <div class="cart-drawer-header">
          <h2 class="cart-header-title">🛒 Your Cart ({{ cartTotalCount }})</h2>
          <button v-if="isAccessible" type="button" class="drawer-close-btn" aria-label="Close cart drawer" @click="isCartOpen = false">✕</button>
          <span v-else class="drawer-close-fake-span" @click="isCartOpen = false">✕</span>
        </div>
        <div class="cart-items-container">
          <div v-if="cartItems.length === 0" class="cart-empty-state">Your cart is empty.</div>
          <ul v-else class="cart-items-list">
            <li v-for="item in cartItems" :key="`${item.id}-${item.selectedColor}`" class="cart-item-card">
              <img :src="item.image" alt="" class="cart-item-thumb" />
              <div class="cart-item-details">
                <h3 class="cart-item-name">{{ item.name }}</h3>
                <div class="cart-item-variant">Variant: {{ item.selectedColor }}</div>
                <div class="cart-item-price">${{ item.price.toFixed(2) }}</div>
                <div class="cart-item-actions">
                  <div class="cart-stepper">
                    <button type="button" class="stepper-sub-btn" @click="updateCartQty(item.id, item.selectedColor, -1)">−</button>
                    <span class="stepper-num">{{ item.quantity }}</span>
                    <button type="button" class="stepper-add-btn" @click="updateCartQty(item.id, item.selectedColor, 1)">+</button>
                  </div>
                  <button type="button" class="cart-item-remove-btn" @click="removeFromCart(item.id, item.selectedColor)">🗑️</button>
                </div>
              </div>
            </li>
          </ul>
        </div>
        <div v-if="cartItems.length > 0" class="cart-drawer-footer">
          <div class="coupon-input-group">
            <input v-model="couponInput" type="text" class="coupon-input" placeholder="Coupon Code (A11Y20)" />
            <button type="button" class="btn-secondary" @click="applyCoupon">Apply</button>
          </div>
          <div v-if="couponFeedback" class="coupon-feedback">{{ couponFeedback }}</div>
          <div class="cart-price-summary">
            <div class="summary-row"><span>Subtotal:</span><span>${{ cartSubtotal.toFixed(2) }}</span></div>
            <div v-if="discountPercent > 0" class="summary-row discount-row"><span>Discount ({{ discountPercent }}%):</span><span>-${{ cartDiscount.toFixed(2) }}</span></div>
            <div class="summary-row"><span>Shipping:</span><span>{{ cartShipping === 0 ? 'FREE' : `$${cartShipping.toFixed(2)}` }}</span></div>
            <div class="summary-row total-row"><strong>Total:</strong><strong class="total-figure">${{ cartGrandTotal.toFixed(2) }}</strong></div>
          </div>
          <button type="button" class="btn-primary checkout-trigger-btn" @click="isCartOpen = false; isCheckoutOpen = true">
            Proceed to Checkout →
          </button>
        </div>
      </div>
    </div>

    <!-- Checkout Modal -->
    <div v-if="isCheckoutOpen" class="checkout-modal-backdrop" @click.self="isCheckoutOpen = false">
      <div class="checkout-dialog" :role="isAccessible ? 'dialog' : undefined" aria-label="Checkout">
        <div class="checkout-dialog-header">
          <h2 class="checkout-title">{{ checkoutSuccess ? '🎉 Order Confirmed!' : '💳 Checkout' }}</h2>
          <button type="button" class="checkout-close-btn" @click="isCheckoutOpen = false; checkoutSuccess = false">✕</button>
        </div>

        <div v-if="checkoutSuccess" class="order-confirmation-body">
          <div class="success-badge-circle">✓</div>
          <h3>Thank you, {{ checkoutForm.fullName || 'Valued Customer' }}!</h3>
          <p>Order #AURA-{{ Math.floor(100000 + Math.random() * 900000) }} has been scheduled for dispatch.</p>
          <button type="button" class="btn-primary" @click="isCheckoutOpen = false; checkoutSuccess = false">Done</button>
        </div>

        <form v-else @submit.prevent="handleCheckoutSubmit">
          <fieldset class="checkout-fieldset">
            <legend class="checkout-legend">Customer Details</legend>
            <div class="form-field-group">
              <label v-if="isAccessible" for="vue-name" class="field-label">Full Name *</label>
              <div v-else class="fake-label-row"><span>Full Name *</span><span v-if="highlightIssues" class="a11y-pill-tag tag-micro">[No Label]</span></div>
              <input id="vue-name" v-model="checkoutForm.fullName" type="text" class="form-input" placeholder="Alex Morgan" />
              <span v-if="checkoutErrors.fullName" class="error-text">{{ checkoutErrors.fullName }}</span>
            </div>
            <div class="form-field-group">
              <label v-if="isAccessible" for="vue-email" class="field-label">Email Address *</label>
              <div v-else class="fake-label-row"><span>Email Address *</span></div>
              <input id="vue-email" v-model="checkoutForm.email" type="email" class="form-input" placeholder="alex@example.com" />
              <span v-if="checkoutErrors.email" class="error-text">{{ checkoutErrors.email }}</span>
            </div>
            <div class="form-field-group">
              <label v-if="isAccessible" for="vue-address" class="field-label">Street Address *</label>
              <div v-else class="fake-label-row"><span>Street Address *</span></div>
              <input id="vue-address" v-model="checkoutForm.address" type="text" class="form-input" placeholder="123 Audio Boulevard" />
              <span v-if="checkoutErrors.address" class="error-text">{{ checkoutErrors.address }}</span>
            </div>
            <div class="form-row-two-col">
              <div class="form-field-group">
                <input v-model="checkoutForm.city" type="text" class="form-input" placeholder="City" />
              </div>
              <div class="form-field-group">
                <input v-model="checkoutForm.zip" type="text" class="form-input" placeholder="Zip / Postal" />
              </div>
            </div>
            <div class="form-field-group">
              <input v-model="checkoutForm.cardNumber" type="text" class="form-input" placeholder="Card Number 4000 1234 5678 9010" />
            </div>
          </fieldset>
          <div class="checkout-submit-row">
            <div class="checkout-final-total">
              <span>Total Due:</span>
              <strong>${{ cartGrandTotal.toFixed(2) }}</strong>
            </div>
            <button type="submit" class="btn-primary">Authorize & Pay</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Bug Explorer Modal -->
    <div v-if="isBugExplorerOpen" class="bug-modal-backdrop" @click.self="isBugExplorerOpen = false">
      <div class="bug-modal-card">
        <div class="bug-modal-header">
          <div>
            <div class="bug-modal-eyebrow">AUDIT SPECIFICATION</div>
            <h2 class="bug-modal-title">🐞 15 Planted Accessibility Issues</h2>
          </div>
          <button type="button" class="bug-modal-close" @click="isBugExplorerOpen = false">✕</button>
        </div>
        <div class="bug-cards-scroll">
          <div v-for="(bug, i) in WCAG_BUGS" :key="bug.id" class="bug-detail-card">
            <div class="bug-badge-row">
              <span class="bug-num">#{{ i + 1 }}</span>
              <span class="bug-area-tag">{{ bug.area }}</span>
              <span class="bug-wcag-tag">{{ bug.wcag }}</span>
              <span class="bug-impact-tag impact-critical">{{ bug.impact }}</span>
            </div>
            <h3 class="bug-title-heading">{{ bug.title }}</h3>
            <p class="bug-text"><strong>Issue:</strong> {{ bug.flawedBehavior }}</p>
            <p class="bug-text-fix"><strong>Remediation:</strong> {{ bug.remediation }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Live Axe Audit Modal -->
    <div v-if="isLiveAxeOpen" class="axe-modal-backdrop" @click.self="isLiveAxeOpen = false">
      <div class="axe-modal-card">
        <div class="axe-modal-header">
          <div>
            <div class="axe-modal-eyebrow">AUTOMATED WCAG AUDIT</div>
            <h2 class="axe-modal-title">⚡ Live Axe-Core Audit Report</h2>
            <p class="axe-modal-subtitle">Mode: {{ isAccessible ? '✅ Remediated (Accessible)' : '⚠️ Flawed (Inaccessible)' }}</p>
          </div>
          <div class="axe-modal-actions">
            <button type="button" class="btn-primary" :disabled="isScanningAxe" @click="runAxe">
              {{ isScanningAxe ? 'Scanning...' : '🔄 Re-Run' }}
            </button>
            <button type="button" class="axe-modal-close" @click="isLiveAxeOpen = false">✕</button>
          </div>
        </div>
        <div class="axe-violations-scroll">
          <div v-if="!axeResults || axeResults.violations.length === 0" class="axe-zero-pass">
            <span class="zero-icon">🎉</span>
            <h3>Zero Accessibility Violations!</h3>
          </div>
          <div v-else>
            <div v-for="v in axeResults.violations" :key="v.id" class="violation-item-card">
              <div class="violation-tags">
                <span class="impact-badge impact-critical">{{ v.impact }}</span>
                <code class="rule-code">{{ v.id }}</code>
              </div>
              <h4 class="violation-help-title">{{ v.help }}</h4>
              <p class="violation-desc">{{ v.description }}</p>
              <div class="violation-nodes-box">
                <span class="nodes-title">Matching Elements ({{ v.nodes.length }}):</span>
                <div v-for="(n, idx) in v.nodes.slice(0, 3)" :key="idx" class="node-snippet-item">
                  <div class="node-target"><code>{{ n.target.join(' > ') }}</code></div>
                  <pre class="node-html"><code>{{ n.html }}</code></pre>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>


    <!-- Toast Notifications -->
    <div class="toast-container" :role="isAccessible ? 'status' : undefined" :aria-live="isAccessible ? 'polite' : undefined">
      <div v-for="t in toasts" :key="t.id" class="toast-item">
        <span>{{ t.type === 'success' ? '✅' : 'ℹ️' }}</span>
        <span>{{ t.message }}</span>
        <button type="button" class="toast-dismiss-btn" @click="removeToast(t.id)">✕</button>
      </div>
    </div>
  </div>
</template>
