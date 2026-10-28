<script setup>
import { ref } from 'vue';

const props = defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const toast = ref(null);

const showToast = (type) => {
  toast.value = {
    id: Date.now(),
    type,
    message:
      type === 'success'
        ? 'Payment processed successfully! Transaction ID: #88219'
        : 'Security Warning: Unusual login detected from new IP address.',
  };

  if (!props.isAccessible) {
    setTimeout(() => {
      toast.value = null;
    }, 2000);
  }
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests dynamic status notifications (WCAG 4.1.3), `aria-live` regions, timing adjustments (WCAG 2.2.1), and dismissible controls.
    </p>

    <div class="btn-group">
      <button
        type="button"
        class="btn btn-primary btn-sm"
        @click="showToast('success')"
      >
        Trigger Success Toast
      </button>
      <button
        type="button"
        class="btn btn-warning btn-sm"
        @click="showToast('warning')"
      >
        Trigger Warning Toast
      </button>
    </div>

    <!-- ACCESSIBLE LIVE REGION -->
    <div
      v-if="isAccessible"
      role="status"
      aria-live="polite"
      aria-atomic="true"
      class="toast-container"
    >
      <div v-if="toast" :class="'toast toast-' + toast.type">
        <div class="toast-body">
          <strong>{{ toast.type === 'success' ? 'Success: ' : 'Warning: ' }}</strong>
          {{ toast.message }}
        </div>
        <button
          type="button"
          class="toast-close-btn"
          :aria-label="'Dismiss ' + toast.type + ' notification'"
          @click="toast = null"
        >
          &times;
        </button>
      </div>
    </div>

    <!-- INACCESSIBLE TOAST CONTAINER -->
    <div v-else class="toast-container">
      <div v-if="toast" :class="'toast toast-' + toast.type">
        <div class="toast-body">{{ toast.message }}</div>
        <span
          class="toast-close-btn"
          style="cursor: pointer;"
          @click="toast = null"
        >
          &times;
        </span>
      </div>
    </div>
  </div>
</template>
