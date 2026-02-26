<template>
  <div class="table-wrap compact">
    <table>
      <thead>
        <tr>
          <th>Ticker</th>
          <th>Lot</th>
          <th>Avg Price</th>
          <th>Broker</th>
          <th>ID</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="!holdings || holdings.length === 0">
          <td colspan="6">Belum ada holding.</td>
        </tr>
        <tr v-else v-for="holding in holdings" :key="holding.id || `${holding.ticker}-${holding.broker_id}`">
          <td><strong>{{ holding.ticker }}</strong></td>
          <td>{{ formatNumber(holding.lot, 0) }}</td>
          <td>{{ formatCurrency(holding.avg_price) }}</td>
          <td>{{ holding.broker_name }}</td>
          <td><code>{{ holding.id }}</code></td>
          <td>
            <button
              class="action-btn delete"
              @click="$emit('delete', holding.ticker)"
            >
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { useFormatters } from '../composables/useFormatters'

defineProps({
  holdings: {
    type: Array,
    default: () => []
  }
})

defineEmits(['delete'])

const { formatCurrency, formatNumber } = useFormatters()
</script>
