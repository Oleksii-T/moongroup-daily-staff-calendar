<template>
  <div
    :style="appointmentStyle"
    class="absolute mx-0.5 rounded z-10 overflow-hidden cursor-pointer"
    @click.stop="handleClick"
  >
    <div class="font-medium text-sm truncate bg-green-300 px-1">{{ appointment.from }} - {{ appointment.to }}</div>
    <div class="h-full bg-orange-300 px-1">
      <div class="text-xs text-gray-600 truncate font-bold">{{ appointment.name }}</div>
      <div class="text-xs text-gray-600 truncate">{{ appointment.client }}</div>
    </div>
  </div>
</template>

<script setup>
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

const emit = defineEmits(['click'])

const timeToMinutes = (time) => {
  const [hours, minutes] = time.split(':').map(Number)
  return hours * 60 + minutes
}

const getOverlappingAppointments = (staff, appointment) => {
  const appointmentStart = timeToMinutes(appointment.from)
  const appointmentEnd = timeToMinutes(appointment.to)
  
  return staff.appointments.filter(apt => {
    const start = timeToMinutes(apt.from)
    const end = timeToMinutes(apt.to)
    return (start < appointmentEnd && end > appointmentStart)
  })
}

const getOrderedIndex = (appointment, overlappingAppts) => {
  const sortedAppts = [...overlappingAppts].sort((a, b) => {
    const aStart = timeToMinutes(a.from)
    const bStart = timeToMinutes(b.from)
    return aStart - bStart
  })
  
  return sortedAppts.findIndex(apt => apt.id === appointment.id)
}

const appointmentStyle = computed(() => {
  const startTime = timeToMinutes(props.appointment.from)
  const endTime = timeToMinutes(props.appointment.to)
  const duration = endTime - startTime
  const height = (duration / 15) * 28 // 32px is the height of one 15-min time slot
  
  const overlappingAppts = getOverlappingAppointments(props.currentStaff, props.appointment)
  const orderedIndex = getOrderedIndex(props.appointment, overlappingAppts)
  
  const width = `${100 / props.totalOverlapping}%`
  const left = `${(orderedIndex * 100) / props.totalOverlapping}%`
  
  return {
    height: `${height}px`,
    width,
    left,
  }
})

const handleClick = () => {
  emit('click', props.appointment)
}
</script> 