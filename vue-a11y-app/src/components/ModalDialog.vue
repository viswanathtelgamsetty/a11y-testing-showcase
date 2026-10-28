<script setup>
import { ref, watch, nextTick, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const isOpen = ref(false);
const openBtnRef = ref(null);
const closeBtnRef = ref(null);
const modalRef = ref(null);

const openModal = () => {
  isOpen.value = true;
  if (props.isAccessible) {
    nextTick(() => {
      closeBtnRef.value?.focus();
    });
  }
};

const closeModal = () => {
  isOpen.value = false;
  if (props.isAccessible) {
    openBtnRef.value?.focus();
  }
};

const handleKeyDown = (e) => {
  if (!props.isAccessible || !isOpen.value) return;

  if (e.key === 'Escape') {
    closeModal();
  }

  if (e.key === 'Tab') {
    if (!modalRef.value) return;
    const focusable = modalRef.value.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const first = focusable[0];
    const last = focusable[focusable.length - 1];

    if (e.shiftKey && document.activeElement === first) {
      e.preventDefault();
      last?.focus();
    } else if (!e.shiftKey && document.activeElement === last) {
      e.preventDefault();
      first?.focus();
    }
  }
};

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
});
</script>

<template>
  <div className="component-demo">
    <p class="demo-description">
      Tests keyboard focus containment, ARIA dialog roles, modal labeling, and Escape key handling.
    </p>

    <!-- Trigger Button -->
    <button
      v-if="isAccessible"
      ref="openBtnRef"
      type="button"
      class="btn btn-primary"
      aria-haspopup="dialog"
      @click="openModal"
    >
      Open Account Deletion Modal
    </button>
    <div
      v-else
      ref="openBtnRef"
      class="btn btn-primary"
      style="display: inline-block; cursor: pointer"
      @click="openModal"
    >
      Open Account Deletion Modal
    </div>

    <!-- Modal Dialog -->
    <teleport to="body">
      <div v-if="isOpen">
        <!-- ACCESSIBLE MODAL -->
        <div
          v-if="isAccessible"
          class="modal-backdrop"
          @click="closeModal"
        >
          <div
            ref="modalRef"
            class="modal-container"
            role="dialog"
            aria-modal="true"
            aria-labelledby="vue-modal-title"
            aria-describedby="vue-modal-desc"
            @click.stop
          >
            <div class="modal-header">
              <h3 id="vue-modal-title" class="modal-title">Confirm Account Deletion</h3>
              <button
                ref="closeBtnRef"
                type="button"
                class="modal-close-btn"
                aria-label="Close confirmation dialog"
                @click="closeModal"
              >
                &times;
              </button>
            </div>
            <div class="modal-body">
              <p id="vue-modal-desc">
                Are you sure you want to permanently delete your account? This action cannot be undone.
              </p>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" @click="closeModal">Cancel</button>
              <button
                type="button"
                class="btn btn-danger"
                @click="() => { alert('Account deleted.'); closeModal(); }"
              >
                Yes, Delete My Account
              </button>
            </div>
          </div>
        </div>

        <!-- INACCESSIBLE MODAL -->
        <div v-else class="modal-backdrop">
          <div class="modal-container">
            <div class="modal-header">
              <div class="modal-title" style="font-size: 1.25rem; font-weight: 600">
                Confirm Account Deletion
              </div>
              <span class="modal-close-btn" style="cursor: pointer" @click="closeModal">
                &times;
              </span>
            </div>
            <div class="modal-body">
              <div>Are you sure you want to permanently delete your account? This action cannot be undone.</div>
            </div>
            <div class="modal-footer">
              <div class="btn btn-secondary" style="cursor: pointer" @click="closeModal">Cancel</div>
              <div
                class="btn btn-danger"
                style="cursor: pointer"
                @click="() => { alert('Account deleted.'); closeModal(); }"
              >
                Yes, Delete My Account
              </div>
            </div>
          </div>
        </div>
      </div>
    </teleport>
  </div>
</template>
