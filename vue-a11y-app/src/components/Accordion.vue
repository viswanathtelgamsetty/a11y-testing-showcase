<script setup>
import { ref } from 'vue';

defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const items = [
  {
    id: 'vue-sec-1',
    title: 'What accessibility standards are supported?',
    content: 'We test against WCAG 2.1 and 2.2 Level A, AA, and AAA criteria, including Section 508 and EN 301 549 standards.',
    link: '#standards-info',
  },
  {
    id: 'vue-sec-2',
    title: 'How does automated testing differ from manual testing?',
    content: 'Automated audits (like Axe-core and Lighthouse) typically detect 30% to 50% of WCAG issues. Screen readers and keyboard navigation testing are mandatory for complete coverage.',
    link: '#audit-guide',
  },
  {
    id: 'vue-sec-3',
    title: 'Can screen readers read collapsed content?',
    content: 'Properly collapsed panels using hidden attributes or display:none are hidden from screen readers. Bad implementations merely set opacity:0, causing "ghost" tab stops.',
    link: '#screen-reader-faq',
  },
];

const openIdx = ref(0);

const toggle = (idx) => {
  openIdx.value = openIdx.value === idx ? -1 : idx;
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests collapsible disclosure widgets: keyboard navigation, `aria-expanded` state communication, and avoiding invisible keyboard traps in collapsed content.
    </p>

    <div class="accordion-group">
      <div v-for="(item, idx) in items" :key="item.id" class="accordion-item">
        <!-- ACCESSIBLE ACCORDION -->
        <template v-if="isAccessible">
          <h3 class="accordion-heading">
            <button
              :id="'vue-acc-btn-' + item.id"
              type="button"
              :aria-expanded="openIdx === idx"
              :aria-controls="'vue-acc-panel-' + item.id"
              class="accordion-trigger"
              @click="toggle(idx)"
            >
              <span>{{ item.title }}</span>
              <span aria-hidden="true" class="accordion-icon">
                {{ openIdx === idx ? '−' : '+' }}
              </span>
            </button>
          </h3>
          <div
            :id="'vue-acc-panel-' + item.id"
            role="region"
            :aria-labelledby="'vue-acc-btn-' + item.id"
            :hidden="openIdx !== idx"
            class="accordion-panel"
          >
            <p>{{ item.content }}</p>
            <a :href="item.link" class="demo-link">Learn more about {{ item.title }} &rarr;</a>
          </div>
        </template>

        <!-- INACCESSIBLE ACCORDION -->
        <template v-else>
          <div
            class="accordion-trigger"
            style="cursor: pointer; font-weight: 600"
            @click="toggle(idx)"
          >
            <span>{{ item.title }}</span>
            <span class="accordion-icon">{{ openIdx === idx ? '−' : '+' }}</span>
          </div>
          <div
            class="accordion-panel-inaccessible"
            :style="openIdx === idx ? { padding: '14px 16px', display: 'block' } : { maxHeight: 0, opacity: 0, overflow: 'hidden' }"
          >
            <p>{{ item.content }}</p>
            <a :href="item.link" class="demo-link">Learn more about {{ item.title }} &rarr;</a>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
