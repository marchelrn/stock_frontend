<template>
  <form class="config-form" @submit.prevent="handleSubmit">
    <label for="base-url">Base URL API</label>
    <div class="config-row">
      <input 
        id="base-url" 
        v-model="url" 
        type="url" 
        placeholder="{{ import.meta.env.VITE_API_URL }}"
        required 
      />
      <button type="submit">Apply</button>
    </div>
  </form>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useApi } from '../composables/useApi'

const emit = defineEmits(['apply'])

const { getBaseUrl, setBaseUrl } = useApi()

const url = ref('')

onMounted(() => {
  url.value = getBaseUrl()
})

const handleSubmit = () => {
  setBaseUrl(url.value)
  emit('apply')
}
</script>
