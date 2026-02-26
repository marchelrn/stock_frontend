<template>
  <form class="stack-form" @submit.prevent="handleSubmit">
    <h3>Add Broker</h3>
    <label for="broker-name">Broker Name</label>
    <input id="broker-name" v-model="name" type="text" required />
    <label for="broker-cash">Cash</label>
    <input id="broker-cash" v-model.number="cash" type="number" min="0" step="0.01" />
    <button type="submit">Add Broker</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['submit'])

const name = ref('')
const cash = ref(0)

const handleSubmit = () => {
  emit('submit', { name: name.value.trim(), cash: cash.value })
  name.value = ''
  cash.value = 0
}
</script>
