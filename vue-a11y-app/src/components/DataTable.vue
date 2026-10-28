<script setup>
import { ref, computed } from 'vue';

defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const rawData = [
  { id: 101, name: 'Alice Walker', department: 'Engineering', score: 98, status: 'Active' },
  { id: 102, name: 'Carlos Mendez', department: 'Design', score: 85, status: 'Active' },
  { id: 103, name: 'Beatrice Vance', department: 'Product', score: 92, status: 'Pending' },
  { id: 104, name: 'David Kim', department: 'Security', score: 78, status: 'Active' },
];

const sortKey = ref('name');
const sortDirection = ref('ascending');

const sortedData = computed(() => {
  return [...rawData].sort((a, b) => {
    if (a[sortKey.value] < b[sortKey.value]) return sortDirection.value === 'ascending' ? -1 : 1;
    if (a[sortKey.value] > b[sortKey.value]) return sortDirection.value === 'ascending' ? 1 : -1;
    return 0;
  });
});

const handleSort = (key) => {
  if (sortKey.value === key) {
    sortDirection.value = sortDirection.value === 'ascending' ? 'descending' : 'ascending';
  } else {
    sortKey.value = key;
    sortDirection.value = 'ascending';
  }
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests table structure semantics (WCAG 1.3.1), table captions, column/row header scopes, and sortable column state (`aria-sort`).
    </p>

    <!-- ACCESSIBLE TABLE -->
    <div v-if="isAccessible" class="table-responsive">
      <table class="data-table">
        <caption class="table-caption">
          Employee Performance Metrics (Sorted by {{ sortKey }} {{ sortDirection }})
        </caption>
        <thead>
          <tr>
            <th scope="col">ID</th>
            <th scope="col" :aria-sort="sortKey === 'name' ? sortDirection : 'none'">
              <button
                type="button"
                class="th-sort-btn"
                :aria-label="`Sort by Employee Name, currently ${sortKey === 'name' ? sortDirection : 'unsorted'}`"
                @click="handleSort('name')"
              >
                Employee Name
                <span aria-hidden="true" class="sort-icon">
                  {{ sortKey === 'name' ? (sortDirection === 'ascending' ? ' ▲' : ' ▼') : ' ↕' }}
                </span>
              </button>
            </th>
            <th scope="col">Department</th>
            <th scope="col" :aria-sort="sortKey === 'score' ? sortDirection : 'none'">
              <button
                type="button"
                class="th-sort-btn"
                :aria-label="`Sort by Performance Score, currently ${sortKey === 'score' ? sortDirection : 'unsorted'}`"
                @click="handleSort('score')"
              >
                Score
                <span aria-hidden="true" class="sort-icon">
                  {{ sortKey === 'score' ? (sortDirection === 'ascending' ? ' ▲' : ' ▼') : ' ↕' }}
                </span>
              </button>
            </th>
            <th scope="col">Status</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in sortedData" :key="row.id">
            <td>#{{ row.id }}</td>
            <th scope="row" class="row-header">{{ row.name }}</th>
            <td>{{ row.department }}</td>
            <td>{{ row.score }}%</td>
            <td>
              <span class="badge" :class="row.status === 'Active' ? 'badge-success' : 'badge-warning'">
                {{ row.status }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- INACCESSIBLE TABLE (DIVS) -->
    <div v-else class="fake-table">
      <div class="fake-table-title">Employee Performance Metrics</div>
      <div class="fake-table-header">
        <div class="fake-cell" style="width: 60px;">ID</div>
        <div
          class="fake-cell"
          style="flex: 1; cursor: pointer; font-weight: bold;"
          @click="handleSort('name')"
        >
          Employee Name {{ sortKey === 'name' ? (sortDirection === 'ascending' ? '▲' : '▼') : '' }}
        </div>
        <div class="fake-cell" style="flex: 1;">Department</div>
        <div
          class="fake-cell"
          style="width: 80px; cursor: pointer; font-weight: bold;"
          @click="handleSort('score')"
        >
          Score {{ sortKey === 'score' ? (sortDirection === 'ascending' ? '▲' : '▼') : '' }}
        </div>
        <div class="fake-cell" style="width: 80px;">Status</div>
      </div>
      <div v-for="row in sortedData" :key="row.id" class="fake-table-row">
        <div class="fake-cell" style="width: 60px;">#{{ row.id }}</div>
        <div class="fake-cell" style="flex: 1;">{{ row.name }}</div>
        <div class="fake-cell" style="flex: 1;">{{ row.department }}</div>
        <div class="fake-cell" style="width: 80px;">{{ row.score }}%</div>
        <div class="fake-cell" style="width: 80px;">{{ row.status }}</div>
      </div>
    </div>
  </div>
</template>
