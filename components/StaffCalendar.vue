<template>
  <div class="staff-calendar">
    <!-- Header with staff names -->
    <div class="calendar-grid border-b">
      <!-- Time column header -->
      <div class="h-20 border-r time-column"></div>
      <!-- Staff column headers -->
      <div v-for="staff in staffMembers" 
           :key="staff.email" 
           class="h-20 border-r staff-column p-2">
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

    <!-- Calendar grid -->
    <div class="calendar-grid relative">
      <!-- Time slots -->
      <template v-for="time in timeSlots" :key="time">
        <!-- Time label -->
        <div 
          class="border-r flex justify-center text-sm time-column relative"
          :class="[
            isHourMark(time) ? 'border-b-2 border-b-gray-300' : 'border-b border-b-gray-200',
            'h-8' // Fixed height for all time slots
          ]"
        >
          <!-- Only show time for hour marks and :30 marks -->
          <span 
            v-if="isHourMark(time) || time.endsWith(':30')" 
            class="absolute -top-2.5 right-2 bg-gray-50 px-1"
          >
            {{ formatTime(time) }}
          </span>
        </div>
        <!-- Staff time slots -->
        <div 
          v-for="staff in staffMembers" 
          :key="`${time}-${staff.email}`"
          class="border-r staff-column relative group"
          :class="[
            isHourMark(time) ? 'border-b-2 border-b-gray-300' : 'border-b border-b-gray-200',
            'h-8' // Fixed height for all time slots
          ]"
          @click="handleSlotClick(time, staff)"
        >
          <!-- Add hover effect element -->
          <div class="absolute inset-0 bg-gray-100 opacity-0 group-hover:opacity-50 transition-opacity"></div>
          
          <!-- Only render appointments at their start time -->
          <div
            v-for="(appointment, index) in getAppointmentsAtTime(staff, time)"
            :key="appointment.id"
            :style="getAppointmentStyle(
              appointment, 
              index, 
              getOverlappingAppointments(staff, appointment).length,
              staff
            )"
            class="absolute mx-0.5 rounded bg-blue-100 border border-blue-200 p-2 z-10 overflow-hidden cursor-pointer hover:bg-blue-200"
            @click.stop="handleAppointmentClick(appointment)"
          >
            <div class="font-medium text-sm truncate">{{ appointment.name }}</div>
            <div class="text-xs text-gray-600 truncate">{{ appointment.client }}</div>
            <div class="text-xs text-gray-500 truncate">
              {{ appointment.from }} - {{ appointment.to }}
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

interface Staff {
  name: string
  email: string
  avatar_url: string
  availabilities: Availability[]
  appointments: Appointment[]
}

interface Availability {
  from: string
  to: string
}

interface Appointment {
  id: string
  name: string
  from: string
  to: string
  client: string
}

const props = defineProps<{
  staffMembers: Staff[]
}>()

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

// Add this new function to check if time slot is on the hour
const isHourMark = (time: string): boolean => {
  return time.endsWith(':00')
}

const formatTime = (time: string) => {
  // Optional: You could also make the hour marks bold
  return isHourMark(time) ? `${time}` : time
}

const handleSlotClick = (time: string, staff: Staff) => {
  // Create a 1-hour appointment
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

// Helper function to add 1 hour to a time string
const addHour = (time: string): string => {
  const [hours, minutes] = time.split(':').map(Number)
  const newHours = hours + 1
  return `${newHours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`
}

// Add this new function to find overlapping appointments
const getOverlappingAppointments = (staff: Staff, appointment: Appointment) => {
  const appointmentStart = timeToMinutes(appointment.from)
  const appointmentEnd = timeToMinutes(appointment.to)
  
  return staff.appointments.filter(apt => {
    const start = timeToMinutes(apt.from)
    const end = timeToMinutes(apt.to)
    
    // Check if appointments overlap
    return (start < appointmentEnd && end > appointmentStart)
  })
}

// Add a helper function to get the index of an appointment within its overlapping group
const getOrderedIndex = (appointment: Appointment, overlappingAppts: Appointment[]): number => {
  // Sort overlapping appointments by start time
  const sortedAppts = [...overlappingAppts].sort((a, b) => {
    const aStart = timeToMinutes(a.from)
    const bStart = timeToMinutes(b.from)
    return aStart - bStart
  })
  
  // Find index of current appointment in sorted array
  return sortedAppts.findIndex(apt => apt.id === appointment.id)
}

// Update getAppointmentStyle to use the staff parameter from props
const getAppointmentStyle = (appointment: Appointment, _index: number, totalOverlapping: number, currentStaff: Staff) => {
  const startTime = timeToMinutes(appointment.from)
  const endTime = timeToMinutes(appointment.to)
  const duration = endTime - startTime
  const height = (duration / 15) * 32 // 32px is the height of one 15-min time slot
  
  // Get overlapping appointments and ordered index
  const overlappingAppts = getOverlappingAppointments(currentStaff, appointment)
  const orderedIndex = getOrderedIndex(appointment, overlappingAppts)
  
  // Calculate width and left position for overlapping appointments
  const width = `${100 / totalOverlapping}%`
  const left = `${(orderedIndex * 100) / totalOverlapping}%`
  
  return {
    height: `${height}px`,
    width,
    left,
  }
}

// Keep getAppointmentsAtTime simple - only return appointments that start at this time
const getAppointmentsAtTime = (staff: Staff, time: string) => {
  return staff.appointments.filter(apt => apt.from === time)
}

// Convert time string to minutes since start of day
const timeToMinutes = (time: string): number => {
  const [hours, minutes] = time.split(':').map(Number)
  return hours * 60 + minutes
}

const handleAppointmentClick = (appointment: Appointment) => {
  console.log('Appointment clicked:', appointment)
}

const emit = defineEmits<{
  (e: 'appointmentCreated', appointment: Appointment & { staffEmail: string }): void
}>()
</script>

<style scoped>
.staff-calendar {
  @apply w-full overflow-x-auto;
  max-width: 100%;
}

.calendar-grid {
  display: grid;
  grid-template-columns: 4rem repeat(v-bind(staffMembers.length), 200px);
  min-width: min-content;
}

.time-column {
  width: 4rem;
  @apply bg-gray-50;
}

.staff-column {
  width: 200px;
  position: relative;
}
</style> 