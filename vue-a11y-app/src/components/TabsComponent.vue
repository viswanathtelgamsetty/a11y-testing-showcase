<script setup>
import { ref } from 'vue';

const props = defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const TABS = [
  {
    id: 'general',
    label: 'General Profile',
    content: 'Manage your public display name, email preferences, and personal biography.',
  },
  {
    id: 'security',
    label: 'Security & Auth',
    content: 'Two-factor authentication (2FA), active login sessions, and password management.',
  },
  {
    id: 'billing',
    label: 'Billing & Plans',
    content: 'Current subscription tier, payment methods, downloadable invoices, and usage quotas.',
  },
];

const activeTab = ref('general');
const tabButtons = ref([]);

const handleKeyDown = (e, index) => {
  if (!props.isAccessible) return;

  let target = index;
  if (e.key === 'ArrowRight') {
    target = (index + 1) % TABS.length;
  } else if (e.key === 'ArrowLeft') {
    target = (index - 1 + TABS.length) % TABS.length;
  } else if (e.key === 'Home') {
    target = 0;
  } else if (e.key === 'End') {
    target = TABS.length - 1;
  } else {
    return;
  }

  e.preventDefault();
  activeTab.value = TABS[target].id;
  tabButtons.value[target]?.focus();
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests ARIA tabs pattern: roving tabIndex, Arrow Left/Right key navigation, `role="tablist"`, `role="tab"`, and `role="tabpanel"`.
    </p>

    <!-- ACCESSIBLE TABS -->
    <div v-if="isAccessible" class="tabs-container">
      <div role="tablist" aria-label="Account Settings Sections" class="tabs-list">
        <button
          v-for="(tab, idx) in TABS"
          :id="'vue-tab-' + tab.id"
          :key="tab.id"
          :ref="el => { if (el) tabButtons[idx] = el }"
          type="button"
          role="tab"
          :aria-selected="tab.id === activeTab"
          :aria-controls="'vue-panel-' + tab.id"
          :tabindex="tab.id === activeTab ? 0 : -1"
          class="tab-btn"
          :class="{ 'tab-btn-active': tab.id === activeTab }"
          @click="activeTab = tab.id"
          @keydown="e => handleKeyDown(e, idx)"
        >
          {{ tab.label }}
        </button>
      </div>

      <div
        v-for="tab in TABS"
        :id="'vue-panel-' + tab.id"
        :key="tab.id"
        role="tabpanel"
        :aria-labelledby="'vue-tab-' + tab.id"
        tabindex="0"
        :hidden="tab.id !== activeTab"
        class="tab-panel"
      >
        <h4>{{ tab.label }}</h4>
        <p>{{ tab.content }}</p>
        <button type="button" class="btn btn-secondary btn-sm" style="margin-top: 8px;">
          Edit {{ tab.label }}
        </button>
      </div>
    </div>

    <!-- INACCESSIBLE TABS -->
    <div v-else class="tabs-container">
      <ul class="tabs-list-inaccessible">
        <li v-for="tab in TABS" :key="tab.id" style="display: inline-block;">
          <a
            href="#dummy"
            class="tab-btn"
            :class="{ 'tab-btn-active': tab.id === activeTab }"
            @click.prevent="activeTab = tab.id"
          >
            {{ tab.label }}
          </a>
        </li>
      </ul>

      <div class="tab-panel">
        <h4>{{ TABS.find(t => t.id === activeTab)?.label }}</h4>
        <p>{{ TABS.find(t => t.id === activeTab)?.content }}</p>
        <div
          class="btn btn-secondary btn-sm"
          style="display: inline-block; margin-top: 8px; cursor: pointer"
        >
          Edit {{ TABS.find(t => t.id === activeTab)?.label }}
        </div>
      </div>
    </div>
  </div>
</template>
