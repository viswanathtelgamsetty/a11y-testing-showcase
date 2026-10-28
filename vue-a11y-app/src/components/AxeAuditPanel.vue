<script setup>
import { ref, computed } from 'vue';
import axe from 'axe-core';

const isRunning = ref(false);
const results = ref(null);
const isOpen = ref(false);

const runAudit = async () => {
  isRunning.value = true;
  try {
    const res = await axe.run(document.body, {
      runOnly: {
        type: 'tag',
        values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'best-practice'],
      },
    });
    results.value = res;
    isOpen.value = true;
  } catch (err) {
    console.error('Axe audit failed:', err);
  } finally {
    isRunning.value = false;
  }
};

const violations = computed(() => results.value?.violations || []);
const criticalCount = computed(() => violations.value.filter(v => v.impact === 'critical').length);
const seriousCount = computed(() => violations.value.filter(v => v.impact === 'serious').length);
const moderateCount = computed(() => violations.value.filter(v => v.impact === 'moderate').length);
const minorCount = computed(() => violations.value.filter(v => v.impact === 'minor').length);

const getImpactBadgeClass = (impact) => {
  switch (impact) {
    case 'critical': return 'badge-critical';
    case 'serious': return 'badge-serious';
    case 'moderate': return 'badge-warning';
    default: return 'badge-minor';
  }
};
</script>

<template>
  <div class="axe-panel-wrapper">
    <div class="axe-toolbar">
      <div class="axe-toolbar-left">
        <span class="axe-logo-icon">🔍</span>
        <div>
          <strong>Automated Axe-Core A11y Scanner (Vue)</strong>
          <p class="axe-subtitle">Audit the current page in real time with rules from WCAG 2.1 &amp; 2.2</p>
        </div>
      </div>

      <div class="axe-toolbar-right">
        <button
          type="button"
          class="btn btn-audit"
          :disabled="isRunning"
          @click="runAudit"
        >
          {{ isRunning ? 'Auditing DOM...' : '⚡ Run Live Axe Audit' }}
        </button>
        <button
          v-if="results"
          type="button"
          class="btn btn-secondary btn-sm"
          @click="isOpen = !isOpen"
        >
          {{ isOpen ? 'Collapse Report' : `View Report (${violations.length})` }}
        </button>
      </div>
    </div>

    <!-- Summary Stats Bar -->
    <div v-if="results" class="axe-summary-bar">
      <div class="summary-stat stat-total">
        <span class="stat-num">{{ violations.length }}</span>
        <span class="stat-label">Total Violations</span>
      </div>
      <div class="summary-stat stat-critical">
        <span class="stat-num">{{ criticalCount }}</span>
        <span class="stat-label">Critical</span>
      </div>
      <div class="summary-stat stat-serious">
        <span class="stat-num">{{ seriousCount }}</span>
        <span class="stat-label">Serious</span>
      </div>
      <div class="summary-stat stat-moderate">
        <span class="stat-num">{{ moderateCount }}</span>
        <span class="stat-label">Moderate</span>
      </div>
      <div class="summary-stat stat-minor">
        <span class="stat-num">{{ minorCount }}</span>
        <span class="stat-label">Minor</span>
      </div>
    </div>

    <!-- Results List -->
    <div v-if="isOpen && results" class="axe-results-container">
      <div v-if="violations.length === 0" class="axe-empty-state">
        🎉 No accessibility violations detected! All active components comply with WCAG standards.
      </div>

      <div v-else class="axe-violation-list">
        <div v-for="(violation, i) in violations" :key="i" class="axe-violation-card">
          <div class="violation-header">
            <span class="impact-badge" :class="getImpactBadgeClass(violation.impact)">
              {{ violation.impact?.toUpperCase() || 'INFO' }}
            </span>
            <span class="violation-id">{{ violation.id }}</span>
            <a
              :href="violation.helpUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="violation-link"
            >
              Axe Rule Docs &rarr;
            </a>
          </div>
          <h4 class="violation-help">{{ violation.help }}</h4>
          <p class="violation-desc">{{ violation.description }}</p>

          <div class="violation-nodes">
            <div class="nodes-title">Failing Elements ({{ violation.nodes.length }}):</div>
            <div v-for="(node, nIdx) in violation.nodes" :key="nIdx" class="node-item">
              <code class="node-selector">{{ node.target.join(' ') }}</code>
              <div class="node-html-snippet">
                <code>{{ node.html }}</code>
              </div>
              <div v-if="node.failureSummary" class="node-failure-summary">
                {{ node.failureSummary }}
              </div>
            </div>
          </div>

          <div class="violation-tags">
            <span v-for="(tag, tIdx) in violation.tags" :key="tIdx" class="wcag-tag">
              #{{ tag }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
