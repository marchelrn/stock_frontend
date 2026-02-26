<template>
  <div class="status" :class="{ error: isError }" role="status" aria-live="polite">
    {{ message }}
  </div>
</template>

<script setup>
defineProps({
  message: {
    type: String,
    default: ''
  },
  isError: {
    type: Boolean,
    default: false
  }
})
</script>
