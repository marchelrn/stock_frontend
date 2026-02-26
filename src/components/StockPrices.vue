<template>
  <div>
    <form class="inline-form" @submit.prevent="handleSubmit">
      <input 
        v-model="tickers" 
        type="text" 
        placeholder="Contoh: BBRI,ASII,PGAS" 
        required 
      />
      <button type="submit">Get Prices</button>
    </form>
    <div class="table-wrap compact">
      <table>
        <thead>
          <tr>
            <th>Ticker</th>
            <th>Price</th>
            <th>Prev Close</th>
            <th>Change</th>
            <th>Change %</th>
            <th>Currency</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="entries.length === 0">
            <td colspan="6">Tidak ada data harga.</td>
          </tr>
          <tr v-else v-for="price in sortedEntries" :key="price.ticker">
            <td><strong>{{ price.ticker }}</strong></td>
            <td>{{ formatNumber(price.price, 2) }}</td>
            <td>{{ formatNumber(price.previous_close, 2) }}</td>
            <td :class="plClass(price.change)">{{ formatNumber(price.change, 2) }}</td>
            <td :class="plClass(price.change_percent)">{{ formatPercent(price.change_percent) }}</td>
            <td>{{ price.currency }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useFormatters } from '../composables/useFormatters'

const props = defineProps({
  prices: {
    type: Object,
    default: () => ({})
  }
})

const emit = defineEmits(['fetch'])

const { formatNumber, formatPercent, plClass } = useFormatters()

const tickers = ref('')

const entries = computed(() => Object.values(props.prices || {}))

const sortedEntries = computed(() => {
  return [...entries.value].sort((a, b) => a.ticker.localeCompare(b.ticker))
})

const handleSubmit = () => {
  const tickerList = tickers.value
    .split(',')
    .map(t => t.trim().toUpperCase())
    .filter(Boolean)
    .join(',')
  
  if (tickerList) {
    emit('fetch', tickerList)
  }
}
</script>
