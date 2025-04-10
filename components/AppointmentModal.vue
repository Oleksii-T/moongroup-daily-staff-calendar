<template>
  <div v-if="show" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white rounded-lg p-6 w-full max-w-md">
      <h2 class="text-xl font-bold mb-4">New Appointment</h2>
      
      <form @submit.prevent="handleSubmit">
        <div class="mb-4">
          <label class="block text-sm font-medium mb-1">Client Name</label>
          <input 
            v-model="form.client"
            type="text"
            class="w-full border rounded p-2"
            required
          >
        </div>

        <div class="mb-4">
          <label class="block text-sm font-medium mb-1">Service</label>
          <input 
            v-model="form.name"
            type="text"
            class="w-full border rounded p-2"
            required
          >
        </div>

        <div class="grid grid-cols-2 gap-4 mb-4">
          <div>
            <label class="block text-sm font-medium mb-1">Start Time</label>
            <input 
              v-model="form.from"
              type="time"
              class="w-full border rounded p-2"
              required
            >
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">End Time</label>
            <input 
              v-model="form.to"
              type="time"
              class="w-full border rounded p-2"
              required
            >
          </div>
        </div>

        <div class="flex justify-end gap-2">
          <button 
            type="button"
            class="px-4 py-2 border rounded"
            @click="$emit('close')"
          >
            Cancel
          </button>
          <button 
            type="submit"
            class="px-4 py-2 bg-blue-500 text-white rounded"
          >
            Create
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  show: boolean
  initialTime: string
  staffMember: {
    name: string
    email: string
  }
}>()

const emit = defineEmits<{
  (e: 'close'): void
  (e: 'create', appointment: {
    id: string
    name: string
    from: string
    to: string
    client: string
  }): void
}>()

const form = ref({
  client: '',
  name: '',
  from: props.initialTime,
  to: props.initialTime
})

const handleSubmit = () => {
  emit('create', {
    id: crypto.randomUUID(),
    ...form.value
  })
  emit('close')
}
</script> 