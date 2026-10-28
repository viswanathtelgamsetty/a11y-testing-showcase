<script setup>
import { ref } from 'vue';

defineProps({
  title: String,
  index: Number,
  wcagTags: Array,
  issuesSummary: Array,
  fixSummary: Array,
  keyboardTips: String,
  isAccessible: Boolean,
});

defineEmits(['toggleAccessible']);

const showDetails = ref(false);
</script>

<template>
  <div class="comp-card" :class="isAccessible ? 'comp-card-accessible' : 'comp-card-inaccessible'">
    <div class="comp-card-header">
      <div class="comp-header-main">
        <div class="comp-index-badge">#{{ index }}</div>
        <div>
          <h2 class="comp-title">{{ title }}</h2>
          <div class="comp-tags-row">
            <span v-for="(tag, i) in wcagTags" :key="i" class="wcag-badge">
              {{ tag }}
            </span>
          </div>
        </div>
      </div>

      <div class="comp-controls">
        <div class="toggle-mode-group">
          <span class="mode-label" :class="{ 'mode-active-bad': !isAccessible }">
            Inaccessible
          </span>
          <button
            type="button"
            class="mode-switch-btn"
            :class="isAccessible ? 'switch-accessible' : 'switch-inaccessible'"
            :aria-label="`Toggle accessibility mode for ${title}. Currently ${isAccessible ? 'Accessible' : 'Inaccessible'}`"
            @click="$emit('toggleAccessible')"
          >
            <span class="mode-switch-handle" />
          </button>
          <span class="mode-label" :class="{ 'mode-active-good': isAccessible }">
            Accessible
          </span>
        </div>

        <button
          type="button"
          class="btn btn-outline-info btn-xs"
          :aria-expanded="showDetails"
          @click="showDetails = !showDetails"
        >
          {{ showDetails ? 'Hide Guide' : 'A11y Breakdown' }}
        </button>
      </div>
    </div>

    <!-- Guide Drawer -->
    <div v-if="showDetails" class="comp-guide-drawer">
      <div class="guide-grid">
        <div class="guide-column guide-bad">
          <h4>❌ Inaccessible Flaws (Violations)</h4>
          <ul>
            <li v-for="(item, i) in issuesSummary" :key="i">{{ item }}</li>
          </ul>
        </div>
        <div class="guide-column guide-good">
          <h4>✅ Accessible Implementation (Fixes)</h4>
          <ul>
            <li v-for="(item, i) in fixSummary" :key="i">{{ item }}</li>
          </ul>
        </div>
      </div>
      <div v-if="keyboardTips" class="guide-keyboard-tips">
        <strong>⌨️ Testing Tips:</strong> {{ keyboardTips }}
      </div>
    </div>

    <div class="comp-card-body">
      <slot />
    </div>
  </div>
</template>
