<template>
    <!-- Randevu Ekleme Modal -->
    <div>
      <div>
        <h3>🆕 Yeni Randevu</h3>
        <form @submit.prevent="addAppointment">
          <div class="form-group">
            <label>👨‍⚕️ Doktor:</label>
            <select v-model="newAppointment.doctorId" required class="doctor-select">
              <option value="">Doktor seçiniz...</option>
              <option v-for="doctor in doctors" :key="doctor.id" :value="doctor.id">
                {{ doctor.name }} - {{ doctor.specialty }}
              </option>
            </select>
          </div>
          
          <div class="form-group">
            <label>👤 Hasta Adı:</label>
            <input 
              v-model="newAppointment.patientName" 
              type="text" 
              required 
              placeholder="Hasta adını giriniz"
            />
          </div>
          
          <div class="form-group">
            <label>📅 Tarih:</label>
            <input 
              v-model="newAppointment.date" 
              type="date" 
              required 
            />
          </div>
          
          <div class="form-group">
            <label>⏰ Başlangıç Saati:</label>
            <input 
              v-model="newAppointment.startTime" 
              type="time" 
              required 
              min="08:00" 
              max="21:00"
            />
          </div>
          
          <div class="form-group">
            <label>⏱️ Bitiş Saati:</label>
            <input 
              v-model="newAppointment.endTime" 
              type="time" 
              required 
              min="08:00" 
              max="21:00"
            />
          </div>
          
          <div class="form-group">
            <label>🩺 Muayene Türü:</label>
            <select v-model="newAppointment.appointmentType" class="doctor-select">
              <option value="Genel Muayene">Genel Muayene</option>
              <option value="Kontrol">Kontrol</option>
              <option value="Tedavi">Tedavi</option>
              <option value="Acil">Acil</option>
              <option value="Laboratuvar">Laboratuvar</option>
              <option value="Aşı">Aşı</option>
            </select>
          </div>
          
          <div class="form-group">
            <label>📝 Notlar:</label>
            <textarea 
              v-model="newAppointment.notes" 
              placeholder="Randevu notları..."
            ></textarea>
          </div>
          
          <div class="modal-actions">
            <button type="button" @click="closeModal" class="cancel-btn">❌ İptal</button>
            <button type="submit" class="save-btn">💾 Kaydet</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Randevu Detay Modal -->
    <div v-if="showDetailModal" class="modal-overlay" @click="closeDetailModal">
      <div class="modal-content detail-modal" @click.stop>
        <h3>📋 Randevu Detayları</h3>
        <div class="appointment-details">
          <div class="detail-item">
            <span class="label">👨‍⚕️ Doktor:</span>
            <span class="value">{{ selectedAppointment.doctorName }}</span>
          </div>
          <div class="detail-item">
            <span class="label">👤 Hasta:</span>
            <span class="value">{{ selectedAppointment.patientName }}</span>
          </div>
          <div class="detail-item">
            <span class="label">📅 Tarih:</span>
            <span class="value">{{ selectedAppointment.date }}</span>
          </div>
          <div class="detail-item">
            <span class="label">⏰ Saat:</span>
            <span class="value">{{ selectedAppointment.time }}</span>
          </div>
          <div class="detail-item">
            <span class="label">🩺 Tür:</span>
            <span class="value">{{ selectedAppointment.type }}</span>
          </div>
          <div class="detail-item" v-if="selectedAppointment.notes">
            <span class="label">📝 Notlar:</span>
            <span class="value">{{ selectedAppointment.notes }}</span>
          </div>
        </div>
        <div class="modal-actions">
          <button @click="closeDetailModal" class="cancel-btn">✅ Tamam</button>
          <button @click="deleteAppointment" class="delete-btn">🗑️ Sil</button>
        </div>
      </div>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted, nextTick } from 'vue'


// Refs
const fullCalendar = ref(null)
const showModal = ref(false)
const showDetailModal = ref(false)
const selectedDate = ref(new Date().toISOString().split('T')[0])
const currentTime = ref('')
const selectedDoctor = ref('all')

// Reactive objects
const selectedAppointment = reactive({})
const newAppointment = reactive({
  doctorId: '',
  patientName: '',
  date: '',
  startTime: '',
  endTime: '',
  appointmentType: 'Genel Muayene',
  notes: ''
})





// Methods
function handleDateSelect(selectInfo) {
  const startTime = new Date(selectInfo.start)
  const endTime = new Date(selectInfo.end)
  
  const startHour = startTime.getHours()
  const endHour = endTime.getHours()
  
  if (startHour < 8 || endHour > 21) {
    alert('Randevular sadece 08:00 - 21:00 saatleri arasında alınabilir!')
    return
  }
  
  newAppointment.date = selectInfo.startStr.split('T')[0]
  newAppointment.startTime = formatTime(startTime)
  newAppointment.endTime = formatTime(endTime)
  showModal.value = true
}

function handleEventClick(clickInfo) {
  const event = clickInfo.event
  const doctor = doctors.value.find(d => d.id === event.extendedProps.doctorId)
  
  Object.assign(selectedAppointment, {
    id: event.id,
    doctorName: doctor ? doctor.name : 'Bilinmeyen Doktor',
    patientName: event.extendedProps.patientName || 'Bilinmeyen Hasta',
    date: event.start.toLocaleDateString('tr-TR'),
    time: `${event.start.toLocaleTimeString('tr-TR', {hour: '2-digit', minute: '2-digit'})} - ${event.end.toLocaleTimeString('tr-TR', {hour: '2-digit', minute: '2-digit'})}`,
    type: event.extendedProps.appointmentType || 'Bilinmeyen',
    notes: event.extendedProps.notes || '',
    eventObj: event
  })
  
  showDetailModal.value = true
}

function addAppointment() {
  if (!validateAppointment()) return
  
  const doctor = doctors.value.find(d => d.id === newAppointment.doctorId)
  const event = {
    id: Date.now().toString(),
    doctorId: newAppointment.doctorId,
    title: `${newAppointment.patientName} - ${newAppointment.appointmentType}`,
    start: `${newAppointment.date}T${newAppointment.startTime}:00`,
    end: `${newAppointment.date}T${newAppointment.endTime}:00`,
    backgroundColor: doctor.color,
    borderColor: doctor.color,
    extendedProps: {
      patientName: newAppointment.patientName,
      appointmentType: newAppointment.appointmentType,
      notes: newAppointment.notes,
      doctorId: newAppointment.doctorId
    }
  }
  
  allAppointments.value.push(event)
  
  // Takvimi güncelle
  if (selectedDoctor.value === 'all' || selectedDoctor.value === newAppointment.doctorId) {
    const calendarApi = fullCalendar.value.getApi()
    calendarApi.addEvent(event)
  }
  
  closeModal()
}

function deleteAppointment() {
  if (confirm('Bu randevuyu silmek istediğinize emin misiniz?')) {
    selectedAppointment.eventObj.remove()
    allAppointments.value = allAppointments.value.filter(app => app.id !== selectedAppointment.id)
    closeDetailModal()
  }
}

function validateAppointment() {
  const startTime = newAppointment.startTime
  const endTime = newAppointment.endTime
  
  if (startTime >= endTime) {
    alert('Bitiş saati başlangıç saatinden sonra olmalıdır!')
    return false
  }
  
  const startHour = parseInt(startTime.split(':')[0])
  const endHour = parseInt(endTime.split(':')[0])
  
  if (startHour < 8 || endHour > 21) {
    alert('Randevular sadece 08:00 - 21:00 saatleri arasında alınabilir!')
    return false
  }
  
  return true
}

function filterByDoctor() {
  updateCalendarEvents()
}

function updateCalendarEvents() {
  let events = allAppointments.value
  
  if (selectedDoctor.value !== 'all') {
    events = allAppointments.value.filter(event => event.doctorId === selectedDoctor.value)
  }
  
  // Calendar events'i güncelle
  const calendarApi = fullCalendar.value.getApi()
  calendarApi.removeAllEvents()
  calendarApi.addEventSource(events)
}

function showAllAppointments() {
  // Başlangıçta tüm randevuları göster
  const calendarApi = fullCalendar.value.getApi()
  calendarApi.addEventSource(allAppointments.value)
}

function getDoctorAppointmentCount(doctorId) {
  return allAppointments.value.filter(app => app.doctorId === doctorId).length
}

function getDoctorName(doctorId) {
  const doctor = doctors.value.find(d => d.id === doctorId)
  return doctor ? doctor.name : 'Bilinmeyen Doktor'
}

function selectDoctor(doctorId) {
  selectedDoctor.value = doctorId
  filterByDoctor()
}

function closeModal() {
  showModal.value = false
  Object.assign(newAppointment, {
    doctorId: '',
    patientName: '',
    date: '',
    startTime: '',
    endTime: '',
    appointmentType: 'Genel Muayene',
    notes: ''
  })
}

function closeDetailModal() {
  showDetailModal.value = false
  Object.assign(selectedAppointment, {})
}

function formatTime(date) {
  return date.toTimeString().slice(0, 5)
}

function goToPrevious() {
  fullCalendar.value.getApi().prev()
}

function goToNext() {
  fullCalendar.value.getApi().next()
}

function goToToday() {
  fullCalendar.value.getApi().today()
  selectedDate.value = new Date().toISOString().split('T')[0]
}

function goToSelectedDate() {
  const api = fullCalendar.value.getApi()
  api.gotoDate(selectedDate.value)
}

function updateCurrentTime() {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('tr-TR', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

function getCurrentDate() {
  return new Date().toLocaleDateString('tr-TR', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

// Lifecycle
onMounted(() => {
  updateCurrentTime()
  setInterval(updateCurrentTime, 1000)
  
  nextTick(() => {
    showAllAppointments()
  })
})
</script>

<style scoped>
.multi-doctor-calendar {
  max-width: 1600px;
  margin: 0 auto;
  padding: 25px;
  font-family: 'Inter', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 25px;
  padding: 25px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.header-left h2 {
  color: #2c3e50;
  margin: 0 0 10px 0;
  font-size: 32px;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.current-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
  margin-bottom: 10px;
}

.current-date {
  font-size: 16px;
  font-weight: 600;
  color: #34495e;
}

.current-time {
  font-size: 18px;
  font-weight: 700;
  color: #e74c3c;
  font-family: 'Courier New', monospace;
  background: linear-gradient(45deg, #ff6b6b, #ee5a52);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.header-right {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: flex-end;
}

.doctor-filter, .date-picker-container {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.doctor-filter label, .date-picker-container label {
  font-weight: 600;
  color: #2c3e50;
  font-size: 14px;
  white-space: nowrap;
}

.doctor-select, .date-picker {
  padding: 8px 12px;
  border: 2px solid #e3f2fd;
  border-radius: 8px;
  font-size: 14px;
  background: white;
  color: #2c3e50;
  font-weight: 500;
  transition: all 0.3s ease;
  cursor: pointer;
  min-width: 200px;
}

.doctor-select:focus, .date-picker:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.2);
  transform: translateY(-1px);
}

.calendar-controls {
  display: flex;
  gap: 8px;
}

.nav-btn, .today-btn {
  padding: 10px 18px;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.nav-btn {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  color: #2c3e50;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.nav-btn:hover {
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

.today-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.today-btn:hover {
  background: linear-gradient(135deg, #5a6fd8 0%, #6a4190 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
}

.doctors-legend {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 25px;
  margin-bottom: 25px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.legend-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 15px;
}

.doctors-legend h3 {
  margin: 0;
  font-size: 20px;
  font-weight: 700;
  color: #2c3e50;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.legend-stats {
  display: flex;
  flex-direction: column;
  gap: 5px;
  align-items: flex-end;
}

.total-appointments {
  background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
  color: white;
  padding: 6px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.active-view {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 4px 10px;
  border-radius: 10px;
  font-size: 11px;
  font-weight: 500;
}

.legend-items {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 20px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  border: 2px solid transparent;
  transition: all 0.3s ease;
  cursor: pointer;
  position: relative;
  backdrop-filter: blur(5px);
}

.legend-item:hover {
  background: rgba(255, 255, 255, 0.95);
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
  border-color: rgba(102, 126, 234, 0.3);
}

.legend-item.highlighted {
  border-color: rgba(102, 126, 234, 0.2);
  background: rgba(102, 126, 234, 0.03);
}

.legend-item.active {
  border-color: #667eea;
  background: rgba(102, 126, 234, 0.1);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

.legend-item.active::before {
  content: '✓';
  position: absolute;
  top: 12px;
  right: 12px;
  background: #667eea;
  color: white;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: bold;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
}

.color-box {
  width: 20px;
  height: 20px;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.all-doctors-icon {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
  transition: all 0.3s ease;
}

.all-doctors-icon span {
  font-size: 24px;
  color: white;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.all-doctors-icon:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.doctor-photo {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}

.doctor-photo img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.doctor-photo:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.doctor-photo:hover img {
  transform: scale(1.05);
}

.doctor-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.doctor-name {
  font-weight: 700;
  font-size: 16px;
  color: #2c3e50;
}

.doctor-specialty {
  font-size: 14px;
  color: #667eea;
  font-weight: 600;
}

.doctor-room {
  font-size: 12px;
  color: #7f8c8d;
}

.doctor-stats {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 5px;
}

.appointment-count {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.percentage {
  background: rgba(0, 0, 0, 0.1);
  color: #2c3e50;
  padding: 2px 8px;
  border-radius: 8px;
  font-size: 10px;
  font-weight: 500;
}

.calendar-container {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(5px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal-content {
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  padding: 30px;
  border-radius: 20px;
  width: 90%;
  max-width: 550px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
  animation: slideIn 0.3s ease;
}

.detail-modal {
  max-width: 450px;
}

@keyframes slideIn {
  from { 
    opacity: 0;
    transform: translateY(-20px) scale(0.95);
  }
  to { 
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.modal-content h3 {
  margin-top: 0;
  margin-bottom: 25px;
  color: #2c3e50;
  font-size: 24px;
  font-weight: 700;
  text-align: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
  color: #555;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e3f2fd;
  border-radius: 12px;
  font-size: 14px;
  box-sizing: border-box;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.9);
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.2);
  transform: translateY(-1px);
  background: white;
}

.form-group textarea {
  height: 80px;
  resize: vertical;
}

.modal-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 20px;
}

.cancel-btn, .save-btn, .delete-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.cancel-btn {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  color: #2c3e50;
}

.cancel-btn:hover {
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  transform: translateY(-1px);
}

.save-btn {
  background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.4);
}

.save-btn:hover {
  background: linear-gradient(135deg, #45a049 0%, #3d8b40 100%);
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.6);
}

.delete-btn {
  background: linear-gradient(135deg, #F44336 0%, #d32f2f 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(244, 67, 54, 0.4);
}

.delete-btn:hover {
  background: linear-gradient(135deg, #d32f2f 0%, #c62828 100%);
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(244, 67, 54, 0.6);
}

/* Randevu Detay Modal */
.appointment-details {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  padding: 12px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 8px;
  border-left: 4px solid #667eea;
}

.detail-item .label {
  font-weight: 600;
  color: #2c3e50;
  min-width: 100px;
}

.detail-item .value {
  font-weight: 500;
  color: #34495e;
  text-align: right;
}

/* FullCalendar özelleştirmeleri */
:deep(.fc) {
  font-family: 'Inter', 'Segoe UI', sans-serif;
}

:deep(.fc-header-toolbar) {
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  margin-bottom: 0;
}

:deep(.fc-toolbar-title) {
  color: white !important;
  font-weight: 700 !important;
  font-size: 24px !important;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

:deep(.fc-button-group .fc-button) {
  background: rgba(255, 255, 255, 0.2) !important;
  border: 1px solid rgba(255, 255, 255, 0.3) !important;
  color: white !important;
  border-radius: 8px !important;
  margin: 0 2px !important;
  font-weight: 600 !important;
  backdrop-filter: blur(10px);
}

:deep(.fc-button-group .fc-button:hover) {
  background: rgba(255, 255, 255, 0.3) !important;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

:deep(.fc-button-group .fc-button-active) {
  background: rgba(255, 255, 255, 0.9) !important;
  color: #667eea !important;
}

:deep(.fc-timegrid-slot) {
  height: 45px !important;
  border-color: rgba(0, 0, 0, 0.05) !important;
}

:deep(.fc-timegrid-slot-minor) {
  border-top: 1px solid rgba(0, 0, 0, 0.03) !important;
}

:deep(.fc-timegrid-slot-major) {
  border-top: 2px solid rgba(102, 126, 234, 0.1) !important;
}

:deep(.fc-event) {
  border-radius: 8px !important;
  font-size: 13px !important;
  padding: 4px 8px !important;
  border: none !important;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15) !important;
  transition: all 0.2s ease !important;
  cursor: pointer !important;
}

:deep(.fc-event:hover) {
  transform: translateY(-1px) !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.25) !important;
}

:deep(.fc-event-title) {
  font-weight: 600 !important;
}

:deep(.fc-business-hours) {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%) !important;
}

:deep(.fc-non-business) {
  background: linear-gradient(135deg, #f5f5f5 0%, #e0e0e0 100%) !important;
}

:deep(.fc-timegrid-now-indicator-line) {
  border-color: #e74c3c !important;
  border-width: 3px !important;
  z-index: 5 !important;
  box-shadow: 0 0 10px rgba(231, 76, 60, 0.5);
}

:deep(.fc-timegrid-now-indicator-arrow) {
  border-left-color: #e74c3c !important;
  border-width: 8px !important;
  margin-top: -8px !important;
  filter: drop-shadow(0 0 3px rgba(231, 76, 60, 0.7));
}

:deep(.fc-col-header) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
  color: white !important;
  font-weight: 600 !important;
  padding: 15px 0 !important;
  border: none !important;
}

:deep(.fc-col-header-cell-cushion) {
  color: white !important;
  text-decoration: none !important;
}

:deep(.fc-timegrid-axis-cushion) {
  color: #667eea !important;
  font-weight: 600 !important;
}

:deep(.fc-scrollgrid) {
  border: none !important;
}


</style>