<template>
  <div class="kpi-grid" v-if="summary">
    <article class="kpi" v-for="item in kpiItems" :key="item.label">
      <label>{{ item.label }}</label>
      <strong :class="item.class">{{ item.value }}</strong>
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

const { formatCurrency, formatPercent, plClass } = useFormatters()
const kpiItems = computed(() => {
  if (!props.summary) return []
  
  return [
    { label: 'Total Portfolio', value: formatCurrency(props.summary.total_portfolio_value), class: '' },
    { label: 'Total Stock Value', value: formatCurrency(props.summary.total_stock_value), class: '' },
    { label: 'Total Cash', value: formatCurrency(props.summary.total_cash), class: '' },
    { label: 'Total Invested', value: formatCurrency(props.summary.total_invested_value), class: '' },
    { label: 'Total P/L', value: formatCurrency(props.summary.total_profit_loss), class: plClass(props.summary.total_profit_loss) },
    { label: 'Total P/L %', value: formatPercent(props.summary.total_profit_pct), class: plClass(props.summary.total_profit_pct) }
  ]
})
</script>
