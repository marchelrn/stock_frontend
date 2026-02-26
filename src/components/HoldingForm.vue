<template>
  <form class="stack-form" @submit.prevent="handleSubmit">
    <h3>Add Holding</h3>
    <label for="holding-ticker">Ticker</label>
    <input id="holding-ticker" v-model="ticker" type="text" required />
    <label for="holding-lot">Lot</label>
    <input id="holding-lot" v-model.number="lot" type="number" min="1" required />
    <label for="holding-avg">Avg Price</label>
    <input id="holding-avg" v-model.number="avgPrice" type="number" min="1" step="0.01" required />
    <label for="holding-broker">Broker</label>
    <select id="holding-broker" v-model.number="brokerId" required>
      <option v-if="!brokers || brokers.length === 0" value="">Tambah broker dulu</option>
      <option v-else v-for="broker in brokers" :key="broker.id" :value="broker.id">
        {{ broker.name }} (ID: {{ broker.id }})
      </option>
    </select>
    <button type="submit">Add Holding</button>
  </form>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  brokers: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['submit'])

const ticker = ref('')
const lot = ref(1)
const avgPrice = ref(0)
const brokerId = ref(null)

const selectedBroker = computed(() => {
  return props.brokers.find(b => Number(b.id) === Number(brokerId.value))
})

const handleSubmit = () => {
  if (!selectedBroker.value) return

  emit('submit', {
    ticker: ticker.value.trim().toUpperCase(),
    lot: lot.value,
    avg_price: avgPrice.value,
    broker_name: selectedBroker.value.name,
    broker_id: selectedBroker.value.id
  })

  ticker.value = ''
  lot.value = 1
  avgPrice.value = 0
}
</script>
