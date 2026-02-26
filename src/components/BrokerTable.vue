<template>
  <div class="table-wrap compact">
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Cash</th>
          <th></th>
        </tr>
      </thead>
      <tbody> 
        <tr v-if="!brokers || brokers.length === 0">
          <td colspan="4">Belum ada broker.</td>
        </tr>
        <tr v-else v-for="broker in brokers" :key="broker.id">
          <td><code>{{ broker.id }}</code></td>
          <td>{{ broker.name }}</td>
          <td>{{ formatCurrency(broker.cash) }}</td>
          <td>
            <button
              class="action-btn delete"
              @click="$emit('delete', broker.name)"
            >
              Delete
            </button>
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
