<template>
  <div class="calendar-container">
   

    <!-- Horizontal Timeline Layout -->
    <div class="horizontal-timeline">
      <!-- Doktor Başlıkları -->
      <div class="doctors-header-row">
        <div v-for="doctor in doctors" :key="doctor.id" class="doctor-header-cell">
          <div class="doctor-photo">
            <img :src="doctor.photo" :alt="doctor.name" />
          </div>
          <div class="doctor-name">{{ doctor.name }}</div>
          <div class="doctor-specialty">{{ doctor.specialty }}</div>
          <div class="doctor-availability" :class="getTodayAppointments(doctor.id).length > 0 ? 'busy' : 'available'">
            {{ getTodayAppointments(doctor.id).length > 0 ? 'DOLU' : 'MÜSAİT' }}
          </div>
        </div>
      </div>
      
      <!-- Timeline Grid -->
      <div class="timeline-grid">
        <div v-for="doctor in doctors" :key="doctor.id" class="doctor-timeline-row">
          <div v-for="hour in timeSlots" :key="hour" class="timeline-cell">
            <!-- Randevu Blokları -->
            <template v-for="appointment in getTodayAppointments(doctor.id).filter(app => {
              const startHour = new Date(app.start).getHours()
              return startHour === parseInt(hour)
            })" :key="appointment.id">
              <div class="appointment-block-horizontal"
                   :style="{
                     backgroundColor: doctor.color,
                     left: ((new Date(appointment.start).getMinutes() / 60) * 100) + '%',
                     width: Math.min(((new Date(appointment.end) - new Date(appointment.start)) / (1000 * 60) / 60) * 100, 100) + '%'
                   }">
                <div class="appointment-content">
                  <div class="appointment-time-small">{{ formatTime(appointment.start) }}</div>
                  <div class="appointment-title-small">{{ appointment.title }}</div>
                </div>
              </div>
            </template>
            
            <!-- Saat Etiketi (sadece ilk satırda) -->
            <div v-if="doctor.id === 1" class="hour-label">{{ hour }}:00</div>
          </div>
        </div>
      </div>
    </div>

   
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { Calendar } from '@fullcalendar/core'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'
import trLocale from '@fullcalendar/core/locales/tr'

// Refs
const calendar = ref(null)
const calendarEl = ref(null)
const currentMonthYear = ref('')

// Data
const doctors = ref([
  {
    id: 1,
    name: 'Dr. Ahmet Yılmaz',
    specialty: 'Kardiyoloji',
    photo: 'https://images.unsplash.com/photo-1612349317150-e413f6a5b16d?w=100&h=100&fit=crop&crop=face',
    color: '#3B82F6'
  },
  {
    id: 2,
    name: 'Dr. Elif Demir',
    specialty: 'Nöroloji',
    photo: 'https://images.unsplash.com/photo-1559839734-2b71ea197ec2?w=100&h=100&fit=crop&crop=face',
    color: '#10B981'
  },
  {
    id: 3,
    name: 'Dr. Mehmet Kaya',
    specialty: 'Ortopedi',
    photo: 'https://images.unsplash.com/photo-1582750433449-648ed127bb54?w=100&h=100&fit=crop&crop=face',
    color: '#F59E0B'
  },
  {
    id: 4,
    name: 'Dr. Zeynep Özkan',
    specialty: 'Dermatoloji',
    photo: 'https://images.unsplash.com/photo-1594824020050-3bf4e2bb8fdb?w=100&h=100&fit=crop&crop=face',
    color: '#EF4444'
  },
  {
    id: 5,
    name: 'Dr. Can Aydın',
    specialty: 'Göz Hastalıkları',
    photo: 'https://images.unsplash.com/photo-1622253692010-333f2da6031d?w=100&h=100&fit=crop&crop=face',
    color: '#8B5CF6'
  },
  {
    id: 6,
    name: 'Dr. Selin Çelik',
    specialty: 'Kadın Doğum',
    photo: 'https://images.unsplash.com/photo-1527613426441-4da17471b66d?w=100&h=100&fit=crop&crop=face',
    color: '#EC4899'
  },
  {
    id: 7,
    name: 'Dr. Emre Şahin',
    specialty: 'Üroloji',
    photo: 'https://images.unsplash.com/photo-1643297654416-05795d62e39c?w=100&h=100&fit=crop&crop=face',
    color: '#06B6D4'
  },
  {
    id: 8,
    name: 'Dr. Ayşe Tunç',
    specialty: 'Psikiyatri',
    photo: 'https://images.unsplash.com/photo-1551836022-deb4988cc6c0?w=100&h=100&fit=crop&crop=face',
    color: '#84CC16'
  },
  {
    id: 9,
    name: 'Dr. Burak Yıldız',
    specialty: 'Genel Cerrahi',
    photo: 'https://images.unsplash.com/photo-1612349317150-e413f6a5b16d?w=100&h=100&fit=crop&crop=face',
    color: '#F97316'
  },
  {
    id: 10,
    name: 'Dr. Fatma Korkmaz',
    specialty: 'Endokrinoloji',
    photo: 'https://images.unsplash.com/photo-1582750433449-648ed127bb54?w=100&h=100&fit=crop&crop=face',
    color: '#6366F1'
  },
  {
    id: 11,
    name: 'Dr. Okan Bulut',
    specialty: 'Kulak Burun Boğaz',
    photo: 'https://images.unsplash.com/photo-1612349317150-e413f6a5b16d?w=100&h=100&fit=crop&crop=face',
    color: '#14B8A6'
  },
  {
    id: 12,
    name: 'Dr. Deniz Karahan',
    specialty: 'Fizik Tedavi',
    photo: 'https://images.unsplash.com/photo-1559839734-2b71ea197ec2?w=100&h=100&fit=crop&crop=face',
    color: '#EAB308'
  },
  {
    id: 13,
    name: 'Dr. Serkan Akgül',
    specialty: 'Anestezi',
    photo: 'https://images.unsplash.com/photo-1622253692010-333f2da6031d?w=100&h=100&fit=crop&crop=face',
    color: '#A855F7'
  },
  {
    id: 14,
    name: 'Dr. Gizem Yüksel',
    specialty: 'Radyoloji',
    photo: 'https://images.unsplash.com/photo-1594824020050-3bf4e2bb8fdb?w=100&h=100&fit=crop&crop=face',
    color: '#DC2626'
  },
  {
    id: 15,
    name: 'Dr. Tolga Demir',
    specialty: 'Gastroenteroloji',
    photo: 'https://images.unsplash.com/photo-1643297654416-05795d62e39c?w=100&h=100&fit=crop&crop=face',
    color: '#059669'
  }
])

const events = ref([
  // Dr. Ahmet Yılmaz - Kardiyoloji
  {
    id: '1',
    title: 'Kontrol Muayenesi',
    start: '2025-07-14T09:00:00',
    end: '2025-07-14T09:30:00',
    doctorId: 1,
    backgroundColor: '#3B82F6',
    borderColor: '#2563EB'
  },
  {
    id: '2',
    title: 'Kalp Kontrolü',
    start: '2025-07-14T14:00:00',
    end: '2025-07-14T15:00:00',
    doctorId: 1,
    backgroundColor: '#3B82F6',
    borderColor: '#2563EB'
  },
  // Dr. Elif Demir - Nöroloji
  {
    id: '3',
    title: 'Nöroloji Konsültasyonu',
    start: '2025-07-14T10:00:00',
    end: '2025-07-14T11:00:00',
    doctorId: 2,
    backgroundColor: '#10B981',
    borderColor: '#059669'
  },
  {
    id: '4',
    title: 'Migren Tedavisi',
    start: '2025-07-14T16:00:00',
    end: '2025-07-14T16:45:00',
    doctorId: 2,
    backgroundColor: '#10B981',
    borderColor: '#059669'
  },
  // Dr. Mehmet Kaya - Ortopedi
  {
    id: '5',
    title: 'Ortopedi Muayenesi',
    start: '2025-07-14T08:30:00',
    end: '2025-07-14T09:15:00',
    doctorId: 3,
    backgroundColor: '#F59E0B',
    borderColor: '#D97706'
  },
  {
    id: '6',
    title: 'Fizik Tedavi Kontrolü',
    start: '2025-07-14T13:30:00',
    end: '2025-07-14T14:30:00',
    doctorId: 3,
    backgroundColor: '#F59E0B',
    borderColor: '#D97706'
  },
  // Dr. Zeynep Özkan - Dermatoloji
  {
    id: '7',
    title: 'Cilt Muayenesi',
    start: '2025-07-14T09:30:00',
    end: '2025-07-14T10:00:00',
    doctorId: 4,
    backgroundColor: '#EF4444',
    borderColor: '#DC2626'
  },
  {
    id: '8',
    title: 'Akne Tedavisi',
    start: '2025-07-14T15:00:00',
    end: '2025-07-14T15:30:00',
    doctorId: 4,
    backgroundColor: '#EF4444',
    borderColor: '#DC2626'
  },
  // Dr. Can Aydın - Göz Hastalıkları
  {
    id: '9',
    title: 'Göz Muayenesi',
    start: '2025-07-14T11:00:00',
    end: '2025-07-14T11:30:00',
    doctorId: 5,
    backgroundColor: '#8B5CF6',
    borderColor: '#7C3AED'
  },
  {
    id: '10',
    title: 'Lens Kontrolü',
    start: '2025-07-14T17:00:00',
    end: '2025-07-14T17:30:00',
    doctorId: 5,
    backgroundColor: '#8B5CF6',
    borderColor: '#7C3AED'
  },
  // Dr. Selin Çelik - Kadın Doğum
  {
    id: '11',
    title: 'Gebelik Kontrolü',
    start: '2025-07-14T10:30:00',
    end: '2025-07-14T11:15:00',
    doctorId: 6,
    backgroundColor: '#EC4899',
    borderColor: '#DB2777'
  },
  {
    id: '12',
    title: 'Ultrason',
    start: '2025-07-14T14:30:00',
    end: '2025-07-14T15:15:00',
    doctorId: 6,
    backgroundColor: '#EC4899',
    borderColor: '#DB2777'
  },
  // Dr. Emre Şahin - Üroloji
  {
    id: '13',
    title: 'Üroloji Kontrolü',
    start: '2025-07-14T08:00:00',
    end: '2025-07-14T08:45:00',
    doctorId: 7,
    backgroundColor: '#06B6D4',
    borderColor: '#0891B2'
  },
  {
    id: '14',
    title: 'Taş Kırma',
    start: '2025-07-14T16:30:00',
    end: '2025-07-14T17:30:00',
    doctorId: 7,
    backgroundColor: '#06B6D4',
    borderColor: '#0891B2'
  },
  // Dr. Ayşe Tunç - Psikiyatri
  {
    id: '15',
    title: 'Psikiyatri Seansı',
    start: '2025-07-14T11:30:00',
    end: '2025-07-14T12:30:00',
    doctorId: 8,
    backgroundColor: '#84CC16',
    borderColor: '#65A30D'
  },
  {
    id: '16',
    title: 'Terapi Seansı',
    start: '2025-07-14T15:30:00',
    end: '2025-07-14T16:30:00',
    doctorId: 8,
    backgroundColor: '#84CC16',
    borderColor: '#65A30D'
  },
  // Dr. Burak Yıldız - Genel Cerrahi
  {
    id: '17',
    title: 'Ameliyat Öncesi Konsültasyon',
    start: '2025-07-14T09:15:00',
    end: '2025-07-14T10:00:00',
    doctorId: 9,
    backgroundColor: '#F97316',
    borderColor: '#EA580C'
  },
  {
    id: '18',
    title: 'Yara Kontrolü',
    start: '2025-07-14T13:00:00',
    end: '2025-07-14T13:30:00',
    doctorId: 9,
    backgroundColor: '#F97316',
    borderColor: '#EA580C'
  },
  // Dr. Fatma Korkmaz - Endokrinoloji
  {
    id: '19',
    title: 'Diyabet Kontrolü',
    start: '2025-07-14T10:00:00',
    end: '2025-07-14T10:45:00',
    doctorId: 10,
    backgroundColor: '#6366F1',
    borderColor: '#4F46E5'
  },
  {
    id: '20',
    title: 'Tiroid Muayenesi',
    start: '2025-07-14T14:00:00',
    end: '2025-07-14T14:45:00',
    doctorId: 10,
    backgroundColor: '#6366F1',
    borderColor: '#4F46E5'
  },
  // Dr. Okan Bulut - Kulak Burun Boğaz
  {
    id: '21',
    title: 'KBB Muayenesi',
    start: '2025-07-14T08:45:00',
    end: '2025-07-14T09:30:00',
    doctorId: 11,
    backgroundColor: '#14B8A6',
    borderColor: '#0D9488'
  },
  {
    id: '22',
    title: 'Sinüzit Tedavisi',
    start: '2025-07-14T15:15:00',
    end: '2025-07-14T16:00:00',
    doctorId: 11,
    backgroundColor: '#14B8A6',
    borderColor: '#0D9488'
  },
  // Dr. Deniz Karahan - Fizik Tedavi
  {
    id: '23',
    title: 'Fizik Tedavi Seansı',
    start: '2025-07-14T11:15:00',
    end: '2025-07-14T12:00:00',
    doctorId: 12,
    backgroundColor: '#EAB308',
    borderColor: '#CA8A04'
  },
  {
    id: '24',
    title: 'Egzersiz Terapisi',
    start: '2025-07-14T16:00:00',
    end: '2025-07-14T17:00:00',
    doctorId: 12,
    backgroundColor: '#EAB308',
    borderColor: '#CA8A04'
  },
  // Dr. Serkan Akgül - Anestezi
  {
    id: '25',
    title: 'Anestezi Konsültasyonu',
    start: '2025-07-14T12:30:00',
    end: '2025-07-14T13:00:00',
    doctorId: 13,
    backgroundColor: '#A855F7',
    borderColor: '#9333EA'
  },
  {
    id: '26',
    title: 'Ameliyat Hazırlığı',
    start: '2025-07-14T17:30:00',
    end: '2025-07-14T18:00:00',
    doctorId: 13,
    backgroundColor: '#A855F7',
    borderColor: '#9333EA'
  },
  // Dr. Gizem Yüksel - Radyoloji
  {
    id: '27',
    title: 'MR Çekimi',
    start: '2025-07-14T09:45:00',
    end: '2025-07-14T10:30:00',
    doctorId: 14,
    backgroundColor: '#DC2626',
    borderColor: '#B91C1C'
  },
  {
    id: '28',
    title: 'Tomografi Değerlendirmesi',
    start: '2025-07-14T14:45:00',
    end: '2025-07-14T15:30:00',
    doctorId: 14,
    backgroundColor: '#DC2626',
    borderColor: '#B91C1C'
  },
  // Dr. Tolga Demir - Gastroenteroloji
  {
    id: '29',
    title: 'Mide Kontrolü',
    start: '2025-07-14T12:00:00',
    end: '2025-07-14T12:45:00',
    doctorId: 15,
    backgroundColor: '#059669',
    borderColor: '#047857'
  },
  {
    id: '30',
    title: 'Endoskopi',
    start: '2025-07-14T16:45:00',
    end: '2025-07-14T17:45:00',
    doctorId: 15,
    backgroundColor: '#059669',
    borderColor: '#047857'
  }
])

// Computed
const todayFormatted = computed(() => {
  const today = new Date()
  return today.toLocaleDateString('tr-TR', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
})

const timeSlots = computed(() => {
  const slots = []
  for (let i = 8; i <= 18; i++) {
    slots.push(i.toString().padStart(2, '0'))
  }
  return slots
})

// Methods
const initializeCalendar = () => {
  calendar.value = new Calendar(calendarEl.value, {
    plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
    locale: trLocale,
    initialView: 'dayGridMonth',
    height: 'auto',
    headerToolbar: false,
    events: events.value,
    editable: true,
    selectable: true,
    selectMirror: true,
    dayMaxEvents: true,
    weekends: true,
    eventContent: renderEventContent,
    dayCellContent: renderDayCellContent,
    datesSet: handleDatesSet,
    eventClick: handleEventClick,
    select: handleDateSelect
  })
  
  calendar.value.render()
  updateCurrentMonthYear()
}

const renderEventContent = (arg) => {
  const doctor = doctors.value.find(d => d.id === arg.event.extendedProps.doctorId)
  return {
    html: `
      <div class="event-content">
        <div class="event-time">${arg.timeText}</div>
        <div class="event-title">${arg.event.title}</div>
        ${doctor ? `<div class="event-doctor">${doctor.name}</div>` : ''}
      </div>
    `
  }
}

const renderDayCellContent = (arg) => {
  const dayEvents = events.value.filter(event => {
    const eventDate = new Date(event.start).toDateString()
    const cellDate = arg.date.toDateString()
    return eventDate === cellDate
  })
  
  const doctorAvatars = dayEvents.map(event => {
    const doctor = doctors.value.find(d => d.id === event.doctorId)
    return doctor ? `
      <div class="day-doctor-avatar" style="border-color: ${doctor.color}">
        <img src="${doctor.photo}" alt="${doctor.name}" title="${doctor.name}" />
      </div>
    ` : ''
  }).join('')
  
  return {
    html: `
      <div class="day-cell-content">
        <div class="day-number">${arg.dayNumberText}</div>
        <div class="day-doctors">${doctorAvatars}</div>
      </div>
    `
  }
}

const handleDatesSet = () => {
  updateCurrentMonthYear()
}

const handleEventClick = (info) => {
  const doctor = doctors.value.find(d => d.id === info.event.extendedProps.doctorId)
  alert(`Randevu: ${info.event.title}\nDoktor: ${doctor ? doctor.name : 'Bilinmiyor'}\nTarih: ${info.event.start.toLocaleString('tr-TR')}`)
}

const handleDateSelect = (selectInfo) => {
  const title = prompt('Randevu başlığı:')
  if (title) {
    const doctorId = parseInt(prompt('Doktor ID (1-3):')) || 1
    const doctor = doctors.value.find(d => d.id === doctorId)
    
    calendar.value.addEvent({
      id: Date.now().toString(),
      title,
      start: selectInfo.startStr,
      end: selectInfo.endStr,
      doctorId,
      backgroundColor: doctor ? doctor.color : '#6B7280',
      borderColor: doctor ? doctor.color : '#4B5563'
    })
  }
  calendar.value.unselect()
}

const updateCurrentMonthYear = () => {
  const date = calendar.value.getDate()
  currentMonthYear.value = date.toLocaleDateString('tr-TR', { 
    year: 'numeric', 
    month: 'long' 
  })
}

const prevMonth = () => {
  calendar.value.prev()
}

const nextMonth = () => {
  calendar.value.next()
}

const toggleTodayAppointments = () => {
  showTodayAppointments.value = !showTodayAppointments.value
  
  if (showTodayAppointments.value) {
    // Vue'nun DOM güncellemesini bekle
    setTimeout(() => {
      initializeTodayCalendar()
    }, 100)
  }
}

const getTodayAppointments = (doctorId) => {
  const today = new Date().toDateString()
  return events.value.filter(event => {
    const eventDate = new Date(event.start).toDateString()
    return eventDate === today && event.doctorId === doctorId
  }).sort((a, b) => new Date(a.start) - new Date(b.start))
}

const formatTime = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleTimeString('tr-TR', {
    hour: '2-digit',
    minute: '2-digit'
  })
}

const calculateDuration = (start, end) => {
  const startDate = new Date(start)
  const endDate = new Date(end)
  const diffMs = endDate - startDate
  const diffMins = Math.floor(diffMs / 60000)
  
  if (diffMins < 60) {
    return `${diffMins} dakika`
  } else {
    const hours = Math.floor(diffMins / 60)
    const minutes = diffMins % 60
    return minutes > 0 ? `${hours} saat ${minutes} dakika` : `${hours} saat`
  }
}

// Lifecycle
onMounted(() => {
  initializeCalendar()
})
</script>

<style scoped>
.calendar-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  padding: 20px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.calendar-title {
  color: white;
  font-size: 2rem;
  font-weight: 700;
  margin: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.calendar-controls {
  display: flex;
  align-items: center;
  gap: 15px;
}

.nav-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  border-radius: 10px;
  padding: 10px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(5px);
}

.nav-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.current-month {
  color: white;
  font-size: 1.2rem;
  font-weight: 600;
  min-width: 200px;
  text-align: center;
}

.horizontal-timeline {
  background: white;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  margin-bottom: 30px;
}

.doctors-header-row {
  display: grid;
  grid-template-columns: repeat(15, 1fr);
  background: #f8fafc;
  border-bottom: 2px solid #e5e7eb;
}

.doctor-header-cell {
  padding: 15px 10px;
  text-align: center;
  border-right: 1px solid #e5e7eb;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  position: relative;
  min-height: 120px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.doctor-header-cell:last-child {
  border-right: none;
}

.doctor-photo {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid rgba(255, 255, 255, 0.3);
  margin: 0 auto 8px;
}

.doctor-photo img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.doctor-name {
  font-size: 0.8rem;
  font-weight: 600;
  line-height: 1.2;
  margin-bottom: 4px;
}

.doctor-specialty {
  font-size: 0.7rem;
  opacity: 0.9;
  line-height: 1.2;
  margin-bottom: 8px;
}

.doctor-availability {
  position: absolute;
  top: 8px;
  right: 8px;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.6rem;
  font-weight: 700;
  letter-spacing: 0.5px;
}

.doctor-availability.busy {
  background: rgba(239, 68, 68, 0.9);
  color: white;
}

.doctor-availability.available {
  background: rgba(34, 197, 94, 0.9);
  color: white;
}

.timeline-grid {
  display: grid;
  grid-template-columns: repeat(15, 1fr);
  max-height: 60vh;
  overflow-y: auto;
}

.doctor-timeline-row {
  display: contents;
}

.timeline-cell {
  height: 60px;
  border-right: 1px solid #e5e7eb;
  border-bottom: 1px solid #e5e7eb;
  position: relative;
  background: #fafafa;
}

.timeline-cell:last-child {
  border-right: none;
}

.timeline-cell:nth-child(odd) {
  background: #ffffff;
}

.timeline-cell:nth-child(even) {
  background: #f9fafb;
}

.hour-label {
  position: absolute;
  top: -25px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.7rem;
  font-weight: 600;
  color: #6b7280;
  background: white;
  padding: 2px 6px;
  border-radius: 4px;
  border: 1px solid #e5e7eb;
}

.appointment-block-horizontal {
  position: absolute;
  top: 5px;
  bottom: 5px;
  border-radius: 4px;
  color: white;
  font-size: 0.7rem;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.2s ease;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.appointment-block-horizontal:hover {
  transform: scale(1.05);
  z-index: 10;
}

.appointment-content {
  padding: 4px 6px;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.appointment-time-small {
  font-weight: 600;
  font-size: 0.65rem;
  margin-bottom: 2px;
  opacity: 0.9;
}

.appointment-title-small {
  font-weight: 500;
  font-size: 0.65rem;
  line-height: 1.1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Responsive için scroll */
.timeline-grid::-webkit-scrollbar {
  height: 8px;
}

.timeline-grid::-webkit-scrollbar-track {
  background: #f1f5f9;
}

.timeline-grid::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.timeline-grid::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

.timeline-container {
  background: white;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.time-header {
  display: flex;
  background: #f8fafc;
  border-bottom: 2px solid #e5e7eb;
}

.time-label {
  width: 200px;
  padding: 15px 20px;
  font-weight: 600;
  color: #374151;
  background: #e5e7eb;
  display: flex;
  align-items: center;
  justify-content: center;
}

.time-slots {
  flex: 1;
  display: grid;
  grid-template-columns: repeat(11, 1fr);
}

.time-slot {
  padding: 15px 8px;
  text-align: center;
  font-weight: 600;
  color: #374151;
  border-right: 1px solid #e5e7eb;
  font-size: 0.9rem;
}

.doctors-timeline {
  max-height: 70vh;
  overflow-y: auto;
}

.doctor-timeline-column {
  display: flex;
  border-bottom: 1px solid #e5e7eb;
}

.doctor-timeline-column:last-child {
  border-bottom: none;
}

.doctor-timeline-header {
  width: 200px;
  padding: 15px 20px;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  position: relative;
}

.doctor-timeline-avatar {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  overflow: hidden;
  border: 3px solid rgba(255, 255, 255, 0.3);
  margin-bottom: 8px;
}

.doctor-timeline-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.doctor-timeline-info h3 {
  margin: 0 0 3px 0;
  font-size: 0.9rem;
  font-weight: 600;
  line-height: 1.2;
}

.doctor-timeline-info p {
  margin: 0 0 8px 0;
  font-size: 0.75rem;
  opacity: 0.9;
  line-height: 1.2;
}

.doctor-status {
  position: absolute;
  top: 8px;
  right: 8px;
}

.status-busy, .status-free {
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.65rem;
  font-weight: 700;
  letter-spacing: 0.5px;
}

.status-busy {
  background: rgba(239, 68, 68, 0.9);
  color: white;
}

.status-free {
  background: rgba(34, 197, 94, 0.9);
  color: white;
}

.time-slots-container {
  flex: 1;
  display: grid;
  grid-template-columns: repeat(11, 1fr);
}

.hour-slot {
  border-right: 1px solid #e5e7eb;
  height: 100px;
  position: relative;
  background: #fafafa;
}

.hour-slot:last-child {
  border-right: none;
}

.appointment-block {
  position: absolute;
  left: 2px;
  right: 2px;
  border-radius: 4px;
  padding: 4px 6px;
  color: white;
  font-size: 0.7rem;
  line-height: 1.2;
  border: 2px solid;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.appointment-block:hover {
  transform: scale(1.02);
  z-index: 10;
}

.appointment-time {
  font-weight: 600;
  margin-bottom: 2px;
  opacity: 0.9;
}

.appointment-title {
  font-weight: 500;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.empty-slot {
  height: 100%;
  background: linear-gradient(45deg, transparent 40%, rgba(229, 231, 235, 0.3) 50%, transparent 60%);
}

/* Zebra stripes for better readability */
.doctor-timeline-column:nth-child(even) .hour-slot {
  background: #f9fafb;
}

.doctor-timeline-column:nth-child(odd) .hour-slot {
  background: #ffffff;
}

/* Scroll styling */
.doctors-timeline::-webkit-scrollbar {
  width: 8px;
}

.doctors-timeline::-webkit-scrollbar-track {
  background: #f1f5f9;
}

.doctors-timeline::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.doctors-timeline::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

.today-calendar-wrapper {
  background: white;
  border-radius: 15px;
  padding: 20px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* Bugünkü takvim özel stilleri */
:deep(.fc-timegrid-event) {
  border: none !important;
  border-radius: 8px !important;
  margin: 2px !important;
  overflow: hidden !important;
}

:deep(.fc-timegrid-event .fc-event-main) {
  padding: 0 !important;
}

.today-event-content {
  padding: 8px;
  border-radius: 6px;
  color: white;
  height: 100%;
  min-height: 60px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.today-event-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 4px;
}

.today-event-avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid rgba(255, 255, 255, 0.3);
  flex-shrink: 0;
}

.today-event-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.today-event-info {
  flex: 1;
  min-width: 0;
}

.today-event-time {
  font-size: 0.7rem;
  font-weight: 600;
  opacity: 0.9;
  line-height: 1;
}

.today-event-doctor {
  font-size: 0.75rem;
  font-weight: 500;
  opacity: 0.95;
  line-height: 1.2;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.today-event-title {
  font-size: 0.8rem;
  font-weight: 600;
  margin: 2px 0;
  line-height: 1.2;
}

.today-event-specialty {
  font-size: 0.7rem;
  opacity: 0.8;
  font-style: italic;
}

:deep(.fc-timegrid-slot) {
  height: 40px !important;
}

:deep(.fc-timegrid-axis) {
  width: 60px !important;
}

:deep(.fc-timegrid-slot-label) {
  font-size: 0.8rem !important;
  color: #6b7280 !important;
}

:deep(.fc-timegrid-divider) {
  border-color: #e5e7eb !important;
}

:deep(.fc-col-header) {
  background: #f8fafc !important;
  font-weight: 600 !important;
  color: #374151 !important;
  padding: 12px !important;
  text-align: center !important;
}

:deep(.fc-now-indicator-line) {
  border-color: #ef4444 !important;
  border-width: 2px !important;
}

.doctors-columns {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 15px;
  max-height: 70vh;
  overflow-y: auto;
}

.doctor-column {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
  min-height: 200px;
}

.doctor-column:hover {
  transform: translateY(-5px);
}

.doctor-column-header {
  padding: 15px;
  color: white;
  display: flex;
  align-items: center;
  gap: 12px;
}

.doctor-column-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.doctor-column-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.doctor-column-info h3 {
  margin: 0 0 3px 0;
  font-size: 1rem;
  font-weight: 600;
}

.doctor-column-info p {
  margin: 0;
  font-size: 0.8rem;
  opacity: 0.9;
}

.doctor-appointments {
  padding: 15px;
  min-height: 150px;
}

.no-appointments {
  text-align: center;
  color: #6b7280;
  padding: 40px 20px;
}

.no-appointments svg {
  margin: 0 auto 10px;
  opacity: 0.5;
}

.no-appointments p {
  margin: 0;
  font-size: 1rem;
}

.appointment-card {
  background: #f8fafc;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 10px;
  transition: all 0.3s ease;
}

.appointment-card:hover {
  background: #f1f5f9;
  border-color: #cbd5e1;
}

.appointment-card:last-child {
  margin-bottom: 0;
}

.appointment-time {
  color: #374151;
  font-weight: 600;
  font-size: 0.9rem;
  margin-bottom: 5px;
}

.appointment-title {
  color: #1f2937;
  font-weight: 600;
  font-size: 1rem;
  margin-bottom: 5px;
}

.appointment-duration {
  color: #6b7280;
  font-size: 0.8rem;
  font-style: italic;
}

.calendar-wrapper {
  background: white;
  border-radius: 15px;
  padding: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* FullCalendar özelleştirmeleri */
:deep(.fc-theme-standard .fc-scrollgrid) {
  border: none;
}

:deep(.fc-theme-standard td, .fc-theme-standard th) {
  border: 1px solid #e5e7eb;
}

:deep(.fc-col-header-cell) {
  background: #f8fafc;
  font-weight: 600;
  color: #374151;
  padding: 12px 8px;
}

:deep(.fc-daygrid-day) {
  min-height: 80px;
  position: relative;
}

:deep(.fc-daygrid-day-number) {
  display: none;
}

:deep(.fc-event) {
  border-radius: 6px;
  margin: 2px 0;
  font-size: 0.8rem;
  padding: 2px 4px;
}

.day-cell-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px;
  min-height: 60px;
}

.day-number {
  font-weight: 600;
  font-size: 1.1rem;
  color: #374151;
  margin-bottom: 8px;
}

.day-doctors {
  display: flex;
  gap: 4px;
  flex-wrap: wrap;
  justify-content: center;
}

.day-doctor-avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid;
}

.day-doctor-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.event-content {
  padding: 2px;
}

.event-time {
  font-size: 0.7rem;
  opacity: 0.8;
}

.event-title {
  font-weight: 600;
  font-size: 0.8rem;
}

.event-doctor {
  font-size: 0.7rem;
  opacity: 0.9;
}

:deep(.fc-daygrid-day.fc-day-today) {
  background-color: rgba(59, 130, 246, 0.1);
}

:deep(.fc-daygrid-day.fc-day-today .day-number) {
  color: #2563eb;
  font-weight: 700;
}
</style>