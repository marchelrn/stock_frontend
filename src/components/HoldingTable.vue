<template>
  <div class="mt-3 overflow-auto rounded-xl border border-slate-200 bg-white">
    <table class="w-full min-w-[620px] border-collapse">
      <thead>
        <tr class="bg-slate-50 text-slate-500">
          <th class="border-b border-slate-200 p-2 text-left text-xs">Ticker</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Lot</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Avg Price</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Broker</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">ID</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="!holdings || holdings.length === 0">
          <td colspan="6" class="p-3 text-sm text-slate-500">Belum ada holding.</td>
        </tr>
        <tr v-else v-for="holding in holdings" :key="holding.id || `${holding.ticker}-${holding.broker_id}`">
          <td class="border-b border-slate-100 p-2 text-sm font-semibold">{{ holding.ticker }}</td>
          <td class="border-b border-slate-100 p-2 text-sm">{{ formatNumber(holding.lot, 0) }}</td>
          <td class="border-b border-slate-100 p-2 text-sm">{{ formatCurrency(holding.avg_price) }}</td>
          <td class="border-b border-slate-100 text-sm">{{ holding.broker_name }}</td>
          <td class="border-b border-slate-100 p-2  text-sm"><code>{{ holding.id }}</code></td>
          <td class="border-b border-slate-100 p-2 text-sm">
            <div class="flex gap-2">
              <button class="rounded-md bg-teal-700 px-2 py-1 text-xs font-semibold text-white hover:bg-teal-800" @click="$emit('trade', { type: 'BUY', holding })">Buy</button>
              <button class="rounded-md bg-rose-700 px-2 py-1 text-xs font-semibold text-white hover:bg-rose-800" @click="$emit('trade', { type: 'SELL', holding })">Sell</button>
            </div>
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

defineEmits(['trade'])

const { formatCurrency, formatNumber } = useFormatters()
</script>
