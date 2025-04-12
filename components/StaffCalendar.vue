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
          isPeriodOf(time, '15') || isPeriodOf(time, '45') ? 'border-gray-100' : '',
          isTimeAvailable(staff, time) ? 'hover:bg-gray-50' : 'bg-gray-50 cursor-not-allowed'
        ]"
        @click="isTimeAvailable(staff, time) && handleSlotClick(time, staff)"
        @dragover.prevent="handleDragOver($event, time, staff)"
        @drop.prevent="handleDrop($event, time, staff)"
      >
        <!-- Only render appointments at their start time -->
        <template v-for="(appointment, index) in getAppointmentsAtTime(staff, time)" :key="appointment.id">
          <DraggableAppointment
            :appointment="appointment"
            :index="index"
            :total-overlapping="getOverlappingAppointmentsCount(staff, appointment)"
            :current-staff="staff"
            @click="handleAppointmentClick"
            @dragStart="handleAppointmentDragStart"
            @drag="handleAppointmentDrag"
            @dragEnd="handleAppointmentDragEnd"
          />
        </template>
        
        <!-- Ghost element for drag and drop -->
        <div
          v-if="isDragging && isGhostVisible(time, staff)"
          :style="ghostStyle"
          class="absolute mx-0.5 rounded z-20 bg-gray-200 opacity-50 pointer-events-none"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import DraggableAppointment from './DraggableAppointment.vue'

const props = defineProps(['staffMembers'])
const emit = defineEmits(['appointmentCreated', 'appointmentMoved'])

// Drag and drop state
const isDragging = ref(false)
const draggedAppointment = ref(null)
const ghostTime = ref(null)
const ghostStaff = ref(null)
const ghostHeight = ref(0)

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

// Drag and drop handlers
const handleAppointmentDragStart = ({ appointment, staff }) => {
  console.log('components/StaffCalendar.vue@handleAppointmentDragStart'); //! LOG
  isDragging.value = true
  draggedAppointment.value = appointment
  ghostTime.value = appointment.from
  ghostStaff.value = staff
  ghostHeight.value = timeToMinutes(appointment.to) - timeToMinutes(appointment.from)
}

const handleAppointmentDrag = ({ clientX, clientY }) => {
  // console.log('components/StaffCalendar.vue@handleAppointmentDrag'); //! LOG
  // Update ghost position based on mouse coordinates
  // This is handled by the dragover event
}

const handleAppointmentDragEnd = () => {
  console.log('components/StaffCalendar.vue@handleAppointmentDragEnd'); //! LOG
  isDragging.value = false
  draggedAppointment.value = null
  ghostTime.value = null
  ghostStaff.value = null
  ghostHeight.value = 0
}

// triggers on every mouse move - A LOT
const handleDragOver = (event, time, staff) => {
  // console.log('components/StaffCalendar.vue@handleDragOver'); //! LOG
  if (!isDragging.value || !isTimeAvailable(staff, time)) return
  
  ghostTime.value = time
  ghostStaff.value = staff
}

const handleDrop = (event, time, staff) => {
  console.log('components/StaffCalendar.vue@handleDrop'); //! LOG

  if (!canBeDroped(event, time, staff)) {
    console.log(' CAN NOT BE DRAGGED'); //! LOG
    return
  }

  const appointment = draggedAppointment.value
  const duration = timeToMinutes(appointment.to) - timeToMinutes(appointment.from)
  const newEndTime = addMinutes(time, duration)
  
  emit('appointmentMoved', {
    appointment,
    newStartTime: time,
    newEndTime,
    newStaffEmail: staff.email
  })
}

const canBeDroped = (event, time, staff) => {
  if (!isDragging.value) {
    return false
  }
  
  if (!isTimeAvailable(staff, time)) {
    return false
  }
  
  const appointment = draggedAppointment.value
  const duration = timeToMinutes(appointment.to) - timeToMinutes(appointment.from)
  const newEndTime = addMinutes(time, duration)
  
  // Check if the entire time range falls within any availability period
  const overlapsWithBreak = staff.availabilities.some(availability => {
    const availabilityStart = timeToMinutes(availability.from)
    const availabilityEnd = timeToMinutes(availability.to)
    const newStartMinutes = timeToMinutes(time)
    const newEndMinutes = timeToMinutes(newEndTime)
    
    return newStartMinutes >= availabilityStart && newEndMinutes <= availabilityEnd
  })

  if (!overlapsWithBreak) {
    return false;
  }

  return true;
}

const addMinutes = (time, minutes) => {
  const [hours, mins] = time.split(':').map(Number)
  const totalMinutes = hours * 60 + mins + minutes
  const newHours = Math.floor(totalMinutes / 60)
  const newMinutes = totalMinutes % 60
  return `${newHours.toString().padStart(2, '0')}:${newMinutes.toString().padStart(2, '0')}`
}

const isGhostVisible = (time, staff) => {
  return ghostTime.value === time && ghostStaff.value === staff
}

const ghostStyle = computed(() => {
  if (!ghostHeight.value) return {}
  
  return {
    height: `${ghostHeight.value / 15 * 28}px`,
    width: '100%',
    left: '0'
  }
})
</script>
