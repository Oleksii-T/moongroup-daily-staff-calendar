<template>
  <div class="w-full overflow-x-auto max-w-full">
    <!-- Header with staff names -->
    <div class="flex gap-[15px]">
      <!-- Time column header -->
      <div class="h-20 w-[45px]"></div>
      <!-- Staff column headers -->
      <div 
        v-for="staff in staffMembers" 
        :key="staff.email" 
        class="h-20 w-[200px]"
      >
        <div class="flex flex-col items-center">
          <img 
            :src="staff.avatar_url" 
            :alt="staff.name"
            class="w-12 h-12 rounded-full mb-1"
          >
          <span class="text-sm font-medium">{{ staff.name }}</span>
        </div>
      </div>
    </div>

    <!-- Calendar row -->
    <div v-for="time in timeSlots" :key="time" class="flex gap-[15px]">
      <!-- First calendar column - Time label -->
      <div class="flex w-[45px] h-[28px] justify-end">
        <div v-if="isPeriodOf(time, '00')">
          <span class="text-xl font-bold">{{ getHourFromTime(time) }}</span>
          <span class="text-sm">00</span>
        </div>
        <div v-if="isPeriodOf(time, 30)">
          <span class="text-sm">30</span>
        </div>
      </div>
      <!-- Other calendar columns - staffs appointments -->
      <div 
        v-for="staff in staffMembers" 
        :key="`${time}-${staff.email}`"
        :class="[
          'w-[200px] relative group border-t',
          isPeriodOf(time, '00') || isPeriodOf(time, 30) ? 'border-gray-200' : '',
          isPeriodOf(time, '15') || isPeriodOf(time, 45) ? 'border-gray-100' : '',
          isTimeAvailable(staff, time) ? 'hover:bg-gray-50' : 'bg-gray-50 cursor-not-allowed'
        ]"
        @click="isTimeAvailable(staff, time) && handleSlotClick(time, staff)"
      >
        <!-- Only render appointments at their start time -->
        <template v-for="(appointment, index) in getAppointmentsAtTime(staff, time)" :key="appointment.id">
          <CalendarAppointment
            :appointment="appointment"
            :index="index"
            :total-overlapping="getOverlappingAppointmentsCount(staff, appointment)"
            :current-staff="staff"
            @click="handleAppointmentClick"
          />
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import CalendarAppointment from './CalendarAppointment.vue'

const props = defineProps(['staffMembers'])
const emit = defineEmits(['appointmentCreated'])

// Generate time slots from 8:00 to 22:00 in 15-minute increments
const timeSlots = computed(() => {
  const slots = []
  const start = 8 // 8:00
  const end = 22 // 22:00
  
  for (let hour = start; hour < end; hour++) {
    slots.push(`${hour.toString().padStart(2, '0')}:00`)
    slots.push(`${hour.toString().padStart(2, '0')}:15`)
    slots.push(`${hour.toString().padStart(2, '0')}:30`)
    slots.push(`${hour.toString().padStart(2, '0')}:45`)
  }
  slots.push(`${end}:00`)
  
  return slots
})

const isPeriodOf = (time, minutes) => {
  return time.endsWith(`:${minutes}`)
}

const getHourFromTime = (time) => {
  return time.split(':')[0]
}

const timeToMinutes = (time) => {
  const [hours, minutes] = time.split(':').map(Number)
  return hours * 60 + minutes
}

const isTimeAvailable = (staff, time) => {
  const currentTime = timeToMinutes(time)
  
  // Check if time falls within any availability period
  return staff.availabilities.some(availability => {
    const startTime = timeToMinutes(availability.from)
    const endTime = timeToMinutes(availability.to)
    return currentTime >= startTime && currentTime < endTime
  })
}

const handleSlotClick = (time, staff) => {
  const startTime = time
  const endTime = addHour(time)
  
  emit('appointmentCreated', {
    id: crypto.randomUUID(),
    name: 'New Appointment',
    from: startTime,
    to: endTime,
    client: 'Walk-in Client',
    staffEmail: staff.email
  })
}

const addHour = (time) => {
  const [hours, minutes] = time.split(':').map(Number)
  const newHours = hours + 1
  return `${newHours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`
}

const getAppointmentsAtTime = (staff, time) => {
  return staff.appointments.filter(apt => apt.from === time)
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

const getOverlappingAppointmentsCount = (staff, appointment) => {
  return getOverlappingAppointments(staff, appointment).length
}

const handleAppointmentClick = (appointment) => {
  console.log('Appointment clicked:', appointment)
}
</script>
