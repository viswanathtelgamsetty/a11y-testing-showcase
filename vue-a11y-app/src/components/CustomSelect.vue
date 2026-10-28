<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const OPTIONS = [
  { value: 'us', label: 'United States (+1)' },
  { value: 'uk', label: 'United Kingdom (+44)' },
  { value: 'ca', label: 'Canada (+1)' },
  { value: 'de', label: 'Germany (+49)' },
  { value: 'in', label: 'India (+91)' },
  { value: 'jp', label: 'Japan (+81)' },
];

const isOpen = ref(false);
const selectedIdx = ref(0);
const highlightedIdx = ref(0);
const triggerRef = ref(null);

const selectedOption = computed(() => OPTIONS[selectedIdx.value]);

const handleKeyDown = (e) => {
  if (!props.isAccessible) return;

  switch (e.key) {
    case 'Enter':
    case ' ':
      e.preventDefault();
      if (isOpen.value) {
        selectedIdx.value = highlightedIdx.value;
        isOpen.value = false;
      } else {
        isOpen.value = true;
        highlightedIdx.value = selectedIdx.value;
      }
      break;
    case 'ArrowDown':
      e.preventDefault();
      if (!isOpen.value) {
        isOpen.value = true;
      } else if (highlightedIdx.value < OPTIONS.length - 1) {
        highlightedIdx.value++;
      }
      break;
    case 'ArrowUp':
      e.preventDefault();
      if (isOpen.value && highlightedIdx.value > 0) {
        highlightedIdx.value--;
      }
      break;
    case 'Escape':
      if (isOpen.value) {
        e.preventDefault();
        isOpen.value = false;
      }
      break;
    case 'Tab':
      isOpen.value = false;
      break;
  }
};

const selectOption = (idx) => {
  selectedIdx.value = idx;
  isOpen.value = false;
  if (props.isAccessible) {
    triggerRef.value?.focus();
  }
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests custom interactive controls: keyboard navigation (Arrow Up/Down, Enter, Esc), ARIA combobox pattern, focus management, and screen reader announcements.
    </p>

    <div class="custom-select-container">
      <!-- ACCESSIBLE COMBOBOX -->
      <template v-if="isAccessible">
        <label id="vue-select-label" class="form-label">Select Your Country Region</label>
        <div class="select-wrapper">
          <button
            id="vue-select-btn"
            ref="triggerRef"
            type="button"
            role="combobox"
            :aria-expanded="isOpen"
            aria-haspopup="listbox"
            aria-labelledby="vue-select-label vue-select-btn"
            aria-controls="vue-select-listbox"
            class="select-trigger"
            @click="isOpen = !isOpen"
            @keydown="handleKeyDown"
          >
            <span>{{ selectedOption.label }}</span>
            <span aria-hidden="true" class="select-arrow">{{ isOpen ? '▲' : '▼' }}</span>
          </button>

          <ul
            v-if="isOpen"
            id="vue-select-listbox"
            role="listbox"
            aria-labelledby="vue-select-label"
            tabindex="-1"
            class="select-options-list"
          >
            <li
              v-for="(opt, idx) in OPTIONS"
              :id="'vue-opt-' + idx"
              :key="opt.value"
              role="option"
              :aria-selected="idx === selectedIdx"
              class="select-option"
              :class="{
                'option-highlighted': idx === highlightedIdx,
                'option-selected': idx === selectedIdx,
              }"
              @click="selectOption(idx)"
              @mouseenter="highlightedIdx = idx"
            >
              {{ opt.label }}
              <span v-if="idx === selectedIdx" aria-hidden="true"> ✓</span>
            </li>
          </ul>
        </div>
      </template>

      <!-- INACCESSIBLE DIV DROPDOWN -->
      <template v-else>
        <div class="form-label-fake">Select Your Country Region</div>
        <div class="select-wrapper">
          <div
            class="select-trigger"
            style="cursor: pointer"
            @click="isOpen = !isOpen"
          >
            <span>{{ selectedOption.label }}</span>
            <span class="select-arrow">{{ isOpen ? '▲' : '▼' }}</span>
          </div>

          <div v-if="isOpen" class="select-options-list">
            <div
              v-for="(opt, idx) in OPTIONS"
              :key="opt.value"
              class="select-option"
              style="cursor: pointer"
              @click="selectOption(idx)"
            >
              {{ opt.label }}
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>
