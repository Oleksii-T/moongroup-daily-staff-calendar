<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">Staff Calendar</h1>
    <StaffCalendar 
      :staff-members="staffMembers" 
      @appointment-created="handleAppointmentCreated"
    />
  </div>
</template>

<script setup lang="ts">
// Temporary mock data for development
const staffMembers = ref([
  {
    name: 'John Doe',
    email: 'john@example.com',
    avatar_url: 'https://api.dicebear.com/7.x/avataaars/svg?seed=John',
    availabilities: [],
    appointments: [
      {
        id: '1',
        name: 'Hair Cut',
        from: '09:00',
        to: '10:00',
        client: 'Alice Johnson'
      },
      {
        id: '2',
        name: 'Coloring',
        from: '11:30',
        to: '13:00',
        client: 'Bob Smith'
      }
    ]
  },
  {
    name: 'Jane Smith',
    email: 'jane@example.com',
    avatar_url: 'https://api.dicebear.com/7.x/avataaars/svg?seed=Jane',
    availabilities: [],
    appointments: [
      {
        id: '3',
        name: 'Manicure',
        from: '09:30',
        to: '10:30',
        client: 'Carol White'
      },
      {
        id: '4',
        name: 'Massage',
        from: '14:00',
        to: '15:00',
        client: 'David Brown'
      }
    ]
  }
])

const handleAppointmentCreated = (appointment: Appointment & { staffEmail: string }) => {
  staffMembers.value = staffMembers.value.map(staff => {
    if (staff.email === appointment.staffEmail) {
      return {
        ...staff,
        appointments: [...staff.appointments, appointment]
      }
    }
    return staff
  })
}
</script>