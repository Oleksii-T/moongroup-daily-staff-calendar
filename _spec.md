# Staff Calendar Project Documentation

## Project Overview

The Staff Calendar is an interactive web application designed to visualize and manage staff appointments throughout the
day. It provides a visual interface for creating, modifying, and organizing appointments for multiple staff members,
while respecting each staff member's availability and break periods.

## Project Description

The Staff Calendar solves the challenge of efficiently managing multiple staff members' schedules in a service-oriented
environment. The application displays a grid-based calendar where:

- Staff members are arranged along the X-axis
- Time slots (in 30-minute increments) are arranged along the Y-axis

This visualization allows administrators to easily see staff availability, schedule appointments, and manage time
conflicts at a glance.

## System Architecture

### Technology Stack

- **Frontend**:

  - React for component-based UI development
  - Tailwind CSS for styling (focused on simplicity and functionality)
  - TypeScript for type safety
  - React DnD (or similar library) for drag-and-drop functionality

- **State Management**:
  - React Context API or Redux for global state management
  - React hooks for component-level state

## Core Features

### Current Implementation

1. **Interactive Calendar Grid**

   - X-axis: Staff members
   - Y-axis: Time slots (30-minute increments from 8:00 to 22:00)
   - Visual indication of working hours vs. non-working hours

2. **Appointment Management**

   - Create appointments by clicking on available time slots
   - Edit appointment details (client name, service type, notes, etc.)
   - Resize appointments by dragging edges (extending/shrinking duration)
   - Move appointments via drag-and-drop (changing time and/or staff member)

3. **Staff Availability Visualization**

   - Display working hours (8:00-22:00 range with customizable start/end times per staff)
   - Non-working hours are grayed out and disabled for appointment creation
   - Break periods are visually distinct and unavailable for scheduling

4. **Time Conflict Handling**
   - Support for overflow appointments (multiple appointments in the same time slot)
   - Visual indication of overlapping appointments

## Technical Specifications

TBD

## Feature Roadmap

### Sprint #1

- Complete the basic calendar grid layout
- Implement click-to-create appointment functionality
- Display existing appointments from configuration data

### MSprint #2

- Implement drag-and-drop functionality for moving appointments
- Add resize functionality for extending/shrinking appointments
- Implement staff availability visualization (working hours)

### Sprint #3

- Add appointment conflict resolution
- Implement break-time visualization and enforcement
- Create modal forms for detailed appointment editing
- Add recurring appointment functionality
- Implement calendar view options (day, week, etc.)
- Add filtering and search capabilities

## Technical Considerations

### Performance

- Efficient rendering of potentially hundreds of appointment blocks
- Smooth drag-and-drop interactions without lag
- Optimized state updates during calendar interactions

### Accessibility

TBD

### Browser Compatibility

TBD

## Known Challenges

- Managing overlapping appointments in the same time slot
- Balancing visual density with usability
- Ensuring drag-and-drop operations respect availability constraints

## Configuration

The calendar will receive its initial configuration data in the following format:

```typescript
interface CalendarConfig {
  staff: Staff[];
  settings: {
    dayStartTime: string; // Default "08:00"
    dayEndTime: string; // Default "22:00"
    slotDuration: number; // In minutes, default 30
  };
}

interface Staff {
  name: string;
  email: string;
  avatar_url: string;
  availabilities: Availability[];
  appointments: Appointment[];
}

interface Availability {
  from: string; // ISO time format
  to: string; // ISO time format
}

interface Appointment {
  name: string;
  from: string; // ISO time format
  to: string; // ISO time format
}
```
