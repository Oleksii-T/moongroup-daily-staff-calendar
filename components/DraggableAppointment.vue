<template>
  <div
    draggable="true"
    @dragstart="handleDragStart"
    @drag="handleDrag"
    @dragend="handleDragEnd"
    @click.stop="handleClick"
  >
    <CalendarAppointment 
      :appointment="appointment"
      :index="index"
      :total-overlapping="totalOverlapping"
      :current-staff="currentStaff"
      />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import CalendarAppointment from './CalendarAppointment.vue'

const props = defineProps({
  appointment: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  totalOverlapping: {
    type: Number,
    required: true
  },
  currentStaff: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['click', 'dragStart', 'drag', 'dragEnd'])

const isDragging = ref(false)
const dragStartTime = ref(null)
const dragStartStaff = ref(null)

const handleDragStart = (event) => {
  isDragging.value = true
  dragStartTime.value = props.appointment.from
  dragStartStaff.value = props.currentStaff
  
  // Set the drag image to be the appointment element
  event.dataTransfer.setData('text/plain', JSON.stringify({
    appointmentId: props.appointment.id,
    startTime: props.appointment.from,
    staffEmail: props.currentStaff.email
  }))
  
  emit('dragStart', {
    appointment: props.appointment,
    staff: props.currentStaff
  })
}

const handleDrag = (event) => {
  emit('drag', {
    appointment: props.appointment,
    staff: props.currentStaff,
    clientX: event.clientX,
    clientY: event.clientY
  })
}

const handleDragEnd = (event) => {
  isDragging.value = false
  emit('dragEnd', {
    appointment: props.appointment,
    staff: props.currentStaff,
    dragStartTime: dragStartTime.value,
    dragStartStaff: dragStartStaff.value
  })
}

const handleClick = () => {
  emit('click', props.appointment)
}
</script> 