<template>
  <div class="min-h-screen bg-gradient-to-br from-amber-50 via-emerald-50 to-cyan-50 text-slate-800">
    <main class="mx-auto grid w-[94vw] max-w-7xl gap-4 py-7">
      <header class="grid gap-4 rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur md:grid-cols-[1.4fr_1fr] md:items-center">
        <div>
          <p class="text-xs font-bold uppercase tracking-widest text-teal-700">Stock API Frontend</p>
          <h1 class="text-2xl font-bold">Portfolio Command Center</h1>
          <p class="mt-2 text-sm text-slate-500">Marchel's Portfolio</p>
        </div>
        <ConfigForm @apply="loadDashboard" />
      </header>

      <section class="rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur">
        <div class="flex items-center justify-between gap-2">
          <h2 class="text-xl font-semibold">Portfolio Summary</h2>
          <button class="rounded-lg bg-teal-700 px-3 py-2 text-sm font-semibold text-white hover:bg-teal-800" @click="loadDashboard">Refresh Data</button>
        </div>
        <StatusBar :message="status" :is-error="isError" />
        <KpiGrid :summary="state.summary" />
      </section>

      <section class="rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur">
        <div class="flex items-center justify-between gap-2">
          <h2 class="text-xl font-semibold">Broker Allocation</h2>
        </div>
        <BrokerCards :summary="state.summary" />
      </section>

      <section class="rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur">
        <div class="flex items-center justify-between gap-2">
          <h2 class="text-xl font-semibold">Ticker Breakdown</h2>
        </div>
        <TickerTable :summary="state.summary" />
      </section>

      <section class="grid gap-4 rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur md:grid-cols-[1.6fr_1fr]">
        <div>
          <div class="flex items-center justify-between gap-2">
            <h2 class="text-xl font-semibold">Brokers</h2>
          </div>
          <BrokerTable :brokers="state.brokers" @delete="handleDeleteBroker" />
        </div>
        <BrokerForm
          :brokers="state.brokers"
          @submit="handleAddBroker"
          @cashSubmit="handleAddBrokerCash"
        />
      </section>

      <section class="grid gap-4 rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur md:grid-cols-[1.6fr_1fr]">
        <div>
          <div class="flex items-center justify-between gap-2">
            <h2 class="text-xl font-semibold">Holdings</h2>
          </div>
          <HoldingTable :holdings="state.holdings" @trade="handleTradeHolding" />
        </div>
        <TransactionForm :brokers="state.brokers" :holdings="state.holdings" @submit="handleAddTransaction" />
      </section>

      <section class="grid gap-4 rounded-2xl border border-slate-200/70 bg-white/80 p-4 shadow-sm backdrop-blur md:grid-cols-[1.6fr_1fr]">
        <div>
          <div class="flex items-center justify-between gap-2">
            <h2 class="text-xl font-semibold">Live Stock Prices</h2>
          </div>
          <StockPrices :prices="state.stockPrices" @fetch="handleFetchStockPrices" />
        </div>

        <div class="rounded-xl border border-slate-200 bg-white p-3">
          <h3 class="text-base font-semibold">Endpoint Used</h3>
          <ul class="mt-2 list-disc space-y-1 pl-5 text-sm text-slate-600">
            <li><code>GET /brokers</code></li>
            <li><code>POST /broker</code></li>
            <li><code>DELETE /broker/:name</code></li>
            <li><code>GET /stocks</code></li>
            <li><code>GET /price/:ticker</code> (validasi BUY ticker)</li>
            <li><code>POST /transaction</code> (BUY/SELL)</li>
            <li><code>GET /prices?ticker=A,B,C</code> (optional)</li>
          </ul>
        </div>
      </section>
    </main>

    <div v-if="tradeModal.open" class="fixed inset-0 z-50 grid place-items-center bg-slate-900/50 p-4" @click="closeTradeModal">
      <div class="w-full max-w-md rounded-2xl border border-slate-200 bg-white p-4 shadow-2xl" @click.stop>
        <div class="mb-2 flex items-center justify-between">
          <h3 class="text-lg font-semibold">{{ tradeModal.type }} {{ tradeModal.holding?.ticker }}</h3>
          <button class="rounded p-1 text-slate-500 hover:bg-slate-100" @click="closeTradeModal">✕</button>
        </div>

        <p class="mb-3 text-sm text-slate-500">Ticker: <strong>{{ tradeModal.holding?.ticker }}</strong></p>

        <form class="grid gap-3" @submit.prevent="submitTradeModal">
          <label class="text-sm text-slate-600" for="trade-broker">Broker</label>
          <select id="trade-broker" v-model.number="tradeForm.brokerId" :disabled="tradeModal.type === 'SELL'" required class="rounded-lg border border-slate-300 bg-white px-3 py-2 text-sm">
            <option v-for="b in state.brokers" :key="b.id" :value="b.id">{{ b.name }} (ID: {{ b.id }})</option>
          </select>

          <label class="text-sm text-slate-600" for="trade-lot">Lot</label>
          <input id="trade-lot" v-model.number="tradeForm.lot" type="number" min="1" required class="rounded-lg border border-slate-300 bg-white px-3 py-2 text-sm" />

          <label class="text-sm text-slate-600" for="trade-price">Harga per lembar</label>
          <input id="trade-price" v-model.number="tradeForm.price" type="number" min="1" step="0.01" required class="rounded-lg border border-slate-300 bg-white px-3 py-2 text-sm" />

          <div class="rounded-lg border border-slate-200 bg-slate-50 p-2 text-sm">
            Estimasi nilai transaksi: <strong>{{ tradeValueText }}</strong>
          </div>

          <div class="mt-1 flex justify-end gap-2">
            <button type="button" class="rounded-lg bg-slate-200 px-3 py-2 text-sm font-semibold text-slate-700 hover:bg-slate-300" @click="closeTradeModal">Batal</button>
            <button type="submit" :class="tradeModal.type === 'BUY' ? 'rounded-lg bg-teal-700 px-3 py-2 text-sm font-semibold text-white hover:bg-teal-800' : 'rounded-lg bg-rose-700 px-3 py-2 text-sm font-semibold text-white hover:bg-rose-800'">
              Confirm {{ tradeModal.type }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, reactive } from 'vue'
import { usePortfolio } from './composables/usePortfolio'
import { useFormatters } from './composables/useFormatters'

import ConfigForm from './components/ConfigForm.vue'
import StatusBar from './components/StatusBar.vue'
import KpiGrid from './components/KpiGrid.vue'
import BrokerCards from './components/BrokerCards.vue'
import TickerTable from './components/TickerTable.vue'
import BrokerTable from './components/BrokerTable.vue'
import BrokerForm from './components/BrokerForm.vue'
import HoldingTable from './components/HoldingTable.vue'
import TransactionForm from './components/TransactionForm.vue'
import StockPrices from './components/StockPrices.vue'

const {
  state,
  status,
  isError,
  loadDashboard,
  addBroker,
  deleteBroker,
  createTransaction,
  fetchStockPrices,
  setStatus
} = usePortfolio()

const { formatCurrency } = useFormatters()

const tradeModal = reactive({
  open: false,
  type: 'BUY',
  holding: null
})

const tradeForm = reactive({
  brokerId: null,
  lot: 1,
  price: 0
})

const selectedTradeBroker = computed(() => {
  return state.brokers.find((b) => Number(b.id) === Number(tradeForm.brokerId)) || null
})

const tradeValueText = computed(() => {
  const lot = Number(tradeForm.lot || 0)
  const price = Number(tradeForm.price || 0)
  return formatCurrency(lot * 100 * price)
})

const handleAddBroker = async ({ name, cash }) => {
  await addBroker(name, cash)
}

const handleAddBrokerCash = async ({ brokerId, cashAmount }) => {
  const broker = state.brokers.find((b) => Number(b.id) === Number(brokerId))
  if (!broker) {
    setStatus('Broker tidak valid. Pilih broker yang tersedia.', true)
    return
  }
  await addBroker(broker.name, cashAmount)
}

const handleDeleteBroker = async (name) => {
  await deleteBroker(name)
}

const handleAddTransaction = async (payload) => {
  const broker = state.brokers.find((b) => Number(b.id) === Number(payload.broker_id))
  if (!broker) {
    setStatus('Broker tidak valid. Pilih broker yang tersedia.', true)
    return
  }
  await createTransaction(payload)
}

const handleTradeHolding = ({ type, holding }) => {
  tradeModal.open = true
  tradeModal.type = type
  tradeModal.holding = holding
  tradeForm.brokerId = Number(holding.broker_id)
  tradeForm.lot = 1
  tradeForm.price = Number(holding.avg_price || 0)
}

const closeTradeModal = () => {
  tradeModal.open = false
  tradeModal.holding = null
  tradeForm.brokerId = null
}

const submitTradeModal = async () => {
  if (!tradeModal.holding) return

  const lot = Number(tradeForm.lot)
  const price = Number(tradeForm.price)

  if (!Number.isFinite(lot) || lot <= 0) {
    setStatus('Lot harus angka > 0', true)
    return
  }

  if (!Number.isFinite(price) || price <= 0) {
    setStatus('Harga harus angka > 0', true)
    return
  }

  if (!selectedTradeBroker.value) {
    setStatus('Broker wajib dipilih', true)
    return
  }

  await createTransaction({
    ticker: tradeModal.holding.ticker,
    lot,
    price,
    broker_id: selectedTradeBroker.value.id,
    broker_name: selectedTradeBroker.value.name,
    type: tradeModal.type
  })

  closeTradeModal()
}

const handleFetchStockPrices = async (tickers) => {
  if (!tickers) {
    setStatus('Masukkan minimal satu ticker.', true)
    return
  }
  await fetchStockPrices(tickers)
}

onMounted(() => {
  loadDashboard()
})
</script>
