<template>
  <div class="mt-3 overflow-auto rounded-xl border border-slate-200 bg-white">
    <table class="w-full min-w-[560px] border-collapse">
      <thead>
        <tr class="bg-slate-50 text-slate-500">
          <th class="border-b border-slate-200 p-2 text-left text-xs">No</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">ID</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Name</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Cash</th>
          <th class="border-b border-slate-200 p-2 text-left text-xs">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="!brokers || brokers.length === 0">
          <td colspan="5" class="p-3 text-sm text-slate-500">Belum ada broker.</td>
        </tr>
        <tr v-else v-for="(broker, index) in brokers" :key="broker.id">
          <td class="border-b border-slate-100 p-2 text-sm">{{ index + 1 }}</td>
          <td class="border-b border-slate-100 p-2 text-sm"><code>{{ broker.id }}</code></td>
          <td class="border-b border-slate-100 p-2 text-sm">{{ broker.name }}</td>
          <td class="border-b border-slate-100 p-2 text-sm">{{ formatCurrency(broker.cash) }}</td>
          <td class="border-b border-slate-100 p-2 text-sm">
            <button class="rounded-md bg-rose-700 px-2 py-1 text-xs font-semibold text-white hover:bg-rose-800" @click="$emit('delete', broker.name)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { useFormatters } from '../composables/useFormatters'

defineProps({
  brokers: {
    type: Array,
    default: () => []
  }
})

defineEmits(['delete'])



const { formatCurrency } = useFormatters()
</script>
