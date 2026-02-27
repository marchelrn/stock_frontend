<template>
  <div class="background-orb orb-1"></div>
  <div class="background-orb orb-2"></div>

  <main class="container">
    <!-- Hero Section -->
    <header class="hero panel reveal">
      <div>
        <p class="eyebrow">Stock API Frontend</p>
        <h1>Portfolio Command Center</h1>
        <p class="muted">
          Marchel's Portfolio
        </p>
      </div>
      <ConfigForm @apply="loadDashboard" />
    </header>

    <!-- Portfolio Summary -->
    <section class="panel reveal">
      <div class="panel-header">
        <h2>Portfolio Summary</h2>
        <button @click="loadDashboard">Refresh Data</button>
      </div>
      <StatusBar :message="status" :is-error="isError" />
      <KpiGrid :summary="state.summary" />
    </section>

    <!-- Broker Allocation -->
    <section class="panel reveal">
      <div class="panel-header">
        <h2>Broker Allocation</h2>
      </div>
      <BrokerCards :summary="state.summary" />
    </section>

    <!-- Ticker Breakdown -->
    <section class="panel reveal">
      <div class="panel-header">
        <h2>Ticker Breakdown</h2>
      </div>
      <TickerTable :summary="state.summary" />
    </section>

    <!-- Brokers Section -->
    <section class="panel reveal split">
      <div>
        <div class="panel-header">
          <h2>Brokers</h2>
        </div>
        <BrokerTable 
          :brokers="state.brokers"
          @delete="handleDeleteBroker"
        />
      </div>
      <BrokerForm @submit="handleAddBroker" />
    </section>

    <!-- Holdings Section -->
    <section class="panel reveal split">
      <div>
        <div class="panel-header">
          <h2>Holdings</h2>
        </div>
        <HoldingTable 
          :holdings="state.holdings"
          @trade="handleTradeHolding"
        />
      </div>
      <TransactionForm
        :brokers="state.brokers"
        :holdings="state.holdings"
        @submit="handleAddTransaction"
      />
    </section>

    <!-- Live Stock Prices -->
    <section class="panel reveal split">
      <div>
        <div class="panel-header">
          <h2>Live Stock Prices</h2>
        </div>
        <StockPrices 
          :prices="state.stockPrices"
          @fetch="handleFetchStockPrices"
        />
      </div>

      <div class="docs-block">
        <h3>Endpoint Used</h3>
        <ul>
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

  <div v-if="tradeModal.open" class="trade-modal-backdrop" @click="closeTradeModal">
    <div class="trade-modal" @click.stop>
      <div class="trade-modal-header">
        <h3>{{ tradeModal.type }} {{ tradeModal.holding?.ticker }}</h3>
        <button class="trade-close" @click="closeTradeModal">✕</button>
      </div>

      <p class="muted trade-subtitle">
        Ticker: <strong>{{ tradeModal.holding?.ticker }}</strong>
      </p>

      <form class="trade-form" @submit.prevent="submitTradeModal">
        <label for="trade-broker">Broker</label>
        <select id="trade-broker" v-model.number="tradeForm.brokerId" :disabled="tradeModal.type === 'SELL'" required>
          <option v-for="b in state.brokers" :key="b.id" :value="b.id">{{ b.name }} (ID: {{ b.id }})</option>
        </select>

        <label for="trade-lot">Lot</label>
        <input id="trade-lot" v-model.number="tradeForm.lot" type="number" min="1" required />

        <label for="trade-price">Harga per lembar</label>
        <input id="trade-price" v-model.number="tradeForm.price" type="number" min="1" step="0.01" required />

        <div class="trade-preview">
          Estimasi nilai transaksi: <strong>{{ tradeValueText }}</strong>
        </div>

        <div class="trade-actions">
          <button type="button" class="secondary" @click="closeTradeModal">Batal</button>
          <button type="submit" :class="tradeModal.type === 'BUY' ? 'buy' : 'sell'">
            Confirm {{ tradeModal.type }}
          </button>
        </div>
      </form>
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

// no local props
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
  const total = lot * 100 * price
  return formatCurrency(total)
})

const handleAddBroker = async ({ name, cash }) => {
  await addBroker(name, cash)
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
