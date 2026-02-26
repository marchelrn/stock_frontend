<template>
  <div class="broker-grid">
    <p v-if="!brokers || brokers.length === 0" class="muted">Belum ada data broker.</p>
    <article v-else v-for="broker in sortedBrokers" :key="broker.id" class="broker-card">
      <h3>{{ broker.name }}</h3>
      <div class="meta">
        <span>Total Value: {{ formatCurrency(broker.total_value) }}</span>
        <span>Stock Value: {{ formatCurrency(broker.stock_value) }}</span>
        <span>Cash: {{ formatCurrency(broker.cash) }}</span>
        <span>Weight: {{ formatPercent(broker.weight) }}</span>
      </div>
      <div class="bar">
        <span :style="{ width: `${Math.max(0, Math.min(100, broker.weight))}%` }"></span>
      </div>
    </article>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useFormatters } from '../composables/useFormatters'

const props = defineProps({
  summary: {
    type: Object,
    default: null
  }
})

const { formatCurrency, formatPercent } = useFormatters()

const brokers = computed(() => props.summary?.brokers || [])

const sortedBrokers = computed(() => {
  return [...brokers.value].sort((a, b) => b.total_value - a.total_value)
})
</script>
