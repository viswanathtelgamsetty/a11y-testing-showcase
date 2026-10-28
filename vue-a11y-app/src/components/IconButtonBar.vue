<script setup>
import { reactive } from 'vue';

defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const actions = reactive({ liked: false, saved: false });

const toggle = (key) => {
  actions[key] = !actions[key];
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests icon-only buttons (WCAG 4.1.2 accessible name), visible focus rings (WCAG 2.4.7), and decorative icon handling (`aria-hidden`).
    </p>

    <div class="icon-toolbar">
      <!-- ACCESSIBLE ICON BUTTONS -->
      <template v-if="isAccessible">
        <button
          type="button"
          class="icon-btn-accessible"
          :class="{ 'is-active': actions.liked }"
          :aria-label="actions.liked ? 'Remove from favorites' : 'Add to favorites'"
          :aria-pressed="actions.liked"
          @click="toggle('liked')"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" :fill="actions.liked ? '#ef4444' : 'none'" stroke="currentColor" stroke-width="2" aria-hidden="true">
            <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" />
          </svg>
        </button>

        <button
          type="button"
          class="icon-btn-accessible"
          :class="{ 'is-active': actions.saved }"
          :aria-label="actions.saved ? 'Remove bookmark' : 'Bookmark this article'"
          :aria-pressed="actions.saved"
          @click="toggle('saved')"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" :fill="actions.saved ? '#3b82f6' : 'none'" stroke="currentColor" stroke-width="2" aria-hidden="true">
            <path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z" />
          </svg>
        </button>

        <button
          type="button"
          class="icon-btn-accessible"
          aria-label="Share article via link"
          @click="() => alert('Link copied to clipboard!')"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
            <circle cx="18" cy="5" r="3" />
            <circle cx="6" cy="12" r="3" />
            <circle cx="18" cy="19" r="3" />
            <line x1="8.59" y1="13.51" x2="15.42" y2="17.49" />
            <line x1="15.41" y1="6.51" x2="8.59" y2="10.49" />
          </svg>
        </button>
      </template>

      <!-- INACCESSIBLE ICON BUTTONS -->
      <template v-else>
        <button
          type="button"
          class="icon-btn-inaccessible"
          @click="toggle('liked')"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" />
          </svg>
        </button>

        <button
          type="button"
          class="icon-btn-inaccessible"
          @click="toggle('saved')"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z" />
          </svg>
        </button>

        <button
          type="button"
          class="icon-btn-inaccessible"
          @click="() => alert('Link copied to clipboard!')"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="18" cy="5" r="3" />
            <circle cx="6" cy="12" r="3" />
            <circle cx="18" cy="19" r="3" />
            <line x1="8.59" y1="13.51" x2="15.42" y2="17.49" />
            <line x1="15.41" y1="6.51" x2="8.59" y2="10.49" />
          </svg>
        </button>
      </template>
    </div>
  </div>
</template>
