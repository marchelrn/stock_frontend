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
          @delete="handleDeleteHolding"
        />
      </div>
      <HoldingForm 
        :brokers="state.brokers"
        @submit="handleAddHolding"
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
          <li><code>POST /stock</code></li>
          <li><code>DELETE /stock/:tickers</code></li>
          <li><code>GET /prices?ticker=A,B,C</code> (optional)</li>
        </ul>
      </div>
    </section>
  </main>

</template>

<script setup>
import { onMounted } from 'vue'
import { usePortfolio } from './composables/usePortfolio'

import ConfigForm from './components/ConfigForm.vue'
import StatusBar from './components/StatusBar.vue'
import KpiGrid from './components/KpiGrid.vue'
import BrokerCards from './components/BrokerCards.vue'
import TickerTable from './components/TickerTable.vue'
import BrokerTable from './components/BrokerTable.vue'
import BrokerForm from './components/BrokerForm.vue'
import HoldingTable from './components/HoldingTable.vue'
import HoldingForm from './components/HoldingForm.vue'
import StockPrices from './components/StockPrices.vue'

const {
  state,
  status,
  isError,
  loadDashboard,
  addBroker,
  deleteBroker,
  addHolding,
  deleteHolding,
  fetchStockPrices,
  setStatus
} = usePortfolio()

const handleAddBroker = async ({ name, cash }) => {
  await addBroker(name, cash)
}

const handleDeleteBroker = async (name) => {
  await deleteBroker(name)
}

const handleAddHolding = async (payload) => {
  const broker = state.brokers.find(b => b.id === payload.broker_id)
  if (!broker) {
    setStatus('Broker tidak valid. Pilih broker yang tersedia.', true)
    return
  }
  await addHolding(payload)
}

const handleDeleteHolding = async (ticker) => {
  await deleteHolding(ticker)
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
