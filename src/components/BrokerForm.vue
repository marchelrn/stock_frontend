<template>
  <div compact>
    <button @click="showForm = !showForm"> Add New Broker</button>
    <button style="margin-left: 10px;" @click="showCashForm = !showCashForm"> Add Broker Cash</button>
    <form style="margin-top: 10px;" v-if="showForm" class="stack-form" id="addBroker" @submit.prevent="handleSubmit">
      <h3>Add Broker</h3>
      <label for="broker-name">Broker Name</label>
      <input id="broker-name" v-model="name" type="text" required />
      <label for="broker-cash">Cash</label>
      <input id="broker-cash" v-model.number="cash" type="number" min="0" step="0.01" />
      <button type="submit">Add Broker</button>
    </form>
    <form style="margin-top: 10px;" v-if="showCashForm" class="stack-form" id="addBrokerCash" @submit.prevent="handleCashSubmit">
      <h3>Add Broker Cash</h3>
      <label for="broker-cash-name">Broker Name</label>
      <input id="broker-cash-name" v-model="cashName" type="text" required />
      <label for="broker-cash-amount">Cash Amount</label>
      <input id="broker-cash-amount" v-model.number="cashAmount" type="number" min="0" step="0.01" />
      <button type="submit">Add Broker</button>
    </form>
  </div>
  
</template> 

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['submit'])

const name = ref('')
const cash = ref(0)
const cashAmount = ref(0)
const showForm = ref(false)
const showCashForm = ref(false)

const openForm = () => {
  showForm.value = true
}

const closeForm = () => {
  showForm.value = false
}

const handleSubmit = () => {
  emit('submit', { name: name.value.trim(), cash: cash.value })
  name.value = ''
  cash.value = 0
}
</script>
