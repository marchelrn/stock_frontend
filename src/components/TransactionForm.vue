<template>
  <form class="stack-form" @submit.prevent="handleSubmit">
    <h3>Add Transaction</h3>

    <label for="tx-type">Type</label>
    <select id="tx-type" v-model="type" required>
      <option value="BUY">BUY</option>
      <option value="SELL">SELL</option>
    </select>

    <label for="tx-broker">Broker</label>
    <select id="tx-broker" v-model.number="brokerId" required>
      <option v-if="brokerOptions.length === 0" value="">Tidak ada broker tersedia</option>
      <option v-else v-for="broker in brokerOptions" :key="broker.id" :value="broker.id">
        {{ broker.name }} (ID: {{ broker.id }})
      </option>
    </select>

    <label for="tx-ticker">Ticker</label>
    <input
      id="tx-ticker"
      v-model="ticker"
      type="text"
      placeholder="Contoh: BBRI"
      required
      @blur="validateBuyTicker"
    />

    <small v-if="validationMessage" :class="validationOk ? 'text-ok' : 'text-error'">
      {{ validationMessage }}
    </small>

    <label for="tx-lot">Lot</label>
    <input id="tx-lot" v-model.number="lot" type="number" min="1" required />

    <label for="tx-price">Harga per lembar</label>
    <input id="tx-price" v-model.number="price" type="number" min="1" step="0.01" required />

    <button type="submit" :disabled="submitting || !canSubmit">
      {{ submitting ? 'Submitting...' : 'Submit Transaction' }}
    </button>
  </form>
</template>

<script setup>
import { computed, ref, watch } from 'vue'
import { useApi } from '../composables/useApi'

const props = defineProps({
  brokers: { type: Array, default: () => [] },
  holdings: { type: Array, default: () => [] }
})

const emit = defineEmits(['submit'])
const { api } = useApi()

const type = ref('BUY')
const ticker = ref('')
const brokerId = ref(null)
const lot = ref(1)
const price = ref(0)
const submitting = ref(false)

const validationMessage = ref('')
const validationOk = ref(false)

const normalizedTicker = computed(() => String(ticker.value || '').trim().toUpperCase())

const sellRows = computed(() => {
  return (props.holdings || []).map((h) => ({
    ticker: String(h.ticker || '').toUpperCase(),
    broker_id: Number(h.broker_id),
    broker_name: h.broker_name,
    avg_price: Number(h.avg_price || 0)
  }))
})

const brokerOptions = computed(() => {
  if (type.value === 'BUY') return props.brokers || []

  const ids = new Set(sellRows.value.map((r) => r.broker_id))
  return (props.brokers || []).filter((b) => ids.has(Number(b.id)))
})

const canSubmit = computed(() => {
  return !!(brokerId.value && normalizedTicker.value && Number(lot.value) > 0 && Number(price.value) > 0)
})

watch(type, (nextType) => {
  validationMessage.value = ''
  validationOk.value = false
  if (nextType === 'SELL') {
    const first = brokerOptions.value[0]
    brokerId.value = first ? Number(first.id) : null
  }
})

watch(brokerOptions, (list) => {
  if (!list.some((b) => Number(b.id) === Number(brokerId.value))) {
    brokerId.value = list[0] ? Number(list[0].id) : null
  }
})

watch([type, brokerId, normalizedTicker], () => {
  if (!normalizedTicker.value) {
    validationMessage.value = ''
    validationOk.value = false
    return
  }

  if (type.value === 'SELL') {
    const row = sellRows.value.find(
      (r) => r.ticker === normalizedTicker.value && Number(r.broker_id) === Number(brokerId.value)
    )

    if (row) {
      validationOk.value = true
      validationMessage.value = 'Ticker tersedia untuk SELL di broker ini'
      if (price.value <= 0) price.value = Number(row.avg_price || 0)
    } else {
      validationOk.value = false
      validationMessage.value = 'Ticker ini tidak ada di holding broker terpilih'
    }
  }
})

const validateBuyTicker = async () => {
  if (type.value !== 'BUY') return true
  if (!normalizedTicker.value) return false

  try {
    const res = await api(`/price/${encodeURIComponent(normalizedTicker.value)}`)
    const marketPrice = Number(res?.data?.price || 0)

    validationOk.value = true
    validationMessage.value = 'Ticker valid (terdaftar / bisa diambil dari IHSG via Yahoo)'
    if (marketPrice > 0) price.value = marketPrice
    return true
  } catch {
    validationOk.value = false
    validationMessage.value = 'Ticker tidak valid / tidak ditemukan di IHSG'
    return false
  }
}

const handleSubmit = async () => {
  if (!canSubmit.value || submitting.value) return

  const broker = (props.brokers || []).find((b) => Number(b.id) === Number(brokerId.value))
  if (!broker) return

  if (type.value === 'BUY') {
    const ok = await validateBuyTicker()
    if (!ok) return
  }

  if (type.value === 'SELL') {
    const exists = sellRows.value.some(
      (r) => r.ticker === normalizedTicker.value && Number(r.broker_id) === Number(brokerId.value)
    )
    if (!exists) {
      validationOk.value = false
      validationMessage.value = 'SELL ditolak: ticker tidak ada di holding broker ini'
      return
    }
  }

  submitting.value = true
  try {
    await emit('submit', {
      type: type.value,
      ticker: normalizedTicker.value,
      broker_id: Number(broker.id),
      broker_name: broker.name,
      lot: Number(lot.value),
      price: Number(price.value)
    })

    ticker.value = ''
    lot.value = 1
  } finally {
    submitting.value = false
  }
}
</script>
