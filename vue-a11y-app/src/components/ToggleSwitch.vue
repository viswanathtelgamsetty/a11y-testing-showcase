<script setup>
import { ref } from 'vue';

defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const isOn = ref(false);

const toggle = () => {
  isOn.value = !isOn.value;
};

const handleKeyDown = (e) => {
  if (e.key === ' ' || e.key === 'Enter') {
    e.preventDefault();
    toggle();
  }
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests ARIA switch pattern (WCAG 4.1.2), keyboard interaction (Space/Enter), and state indication (`aria-checked`).
    </p>

    <div class="toggle-wrapper">
      <!-- ACCESSIBLE SWITCH -->
      <div v-if="isAccessible" class="toggle-row">
        <span id="vue-airplane-label" class="toggle-label-text">
          Enable Offline Airplane Mode
        </span>
        <button
          id="vue-airplane-switch"
          type="button"
          role="switch"
          :aria-checked="isOn"
          aria-labelledby="vue-airplane-label"
          class="toggle-track"
          :class="isOn ? 'track-on' : 'track-off'"
          @click="toggle"
          @keydown="handleKeyDown"
        >
          <span class="toggle-thumb" :class="isOn ? 'thumb-on' : 'thumb-off'" />
        </button>
        <span class="toggle-state-text" aria-hidden="true">{{ isOn ? 'ON' : 'OFF' }}</span>
      </div>

      <!-- INACCESSIBLE SWITCH -->
      <div v-else class="toggle-row">
        <div class="toggle-label-text">Enable Offline Airplane Mode</div>
        <div
          class="toggle-track"
          :class="isOn ? 'track-on' : 'track-off'"
          style="cursor: pointer;"
          @click="toggle"
        >
          <div class="toggle-thumb" :class="isOn ? 'thumb-on' : 'thumb-off'" />
        </div>
        <div style="font-size: 13px; color: #666;">{{ isOn ? 'ON' : 'OFF' }}</div>
      </div>
    </div>
  </div>
</template>
