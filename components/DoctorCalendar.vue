// components/DoctorCalendar.vue
<template>
  <div class="calendar-container">
    <div class="calendar-header">
      <h2>Doktor Randevu Takvimi</h2>
      <div class="calendar-controls">
        <button @click="prevMonth" class="control-btn">Önceki Ay</button>
        <span class="current-month">{{ currentMonthName }} {{ currentYear }}</span>
        <button @click="nextMonth" class="control-btn">Sonraki Ay</button>
      </div>
      <div class="doctor-selector">
        <label for="doctor-select">Doktor Seçiniz:</label>
        <select id="doctor-select" v-model="selectedDoctor" @change="loadAppointments">
          <option v-for="doctor in doctors" :key="doctor.id" :value="doctor.id">
            {{ doctor.name }} ({{ doctor.specialty }})
          </option>
        </select>
      </div>
    </div>

    <!-- Haftanın günleri -->
    <div class="weekdays">
      <div v-for="day in weekdays" :key="day" class="weekday">{{ day }}</div>
    </div>

    <!-- Takvim günleri -->
    <div class="calendar-days">
      <div 
        v-for="(day, index) in calendarDays" 
        :key="index" 
        class="calendar-day" 
        :class="{ 
          'empty': !day.date, 
          'today': day.isToday,
          'has-appointments': day.appointments && day.appointments.length
        }"
        @click="day.date && openDayDetail(day)"
      >
        <div v-if="day.date" class="day-number">{{ day.date.getDate() }}</div>
        <div v-if="day.date && day.appointments && day.appointments.length" class="appointment-count">
          {{ day.appointments.length }} randevu
        </div>
      </div>
    </div>

    <!-- Randevu Ekleme/Düzenleme Modalı -->
    <div v-if="showAppointmentModal" class="modal-backdrop" @click="closeAppointmentModal">
      <div class="modal-content" @click.stop>
        <h3>{{ editingAppointment ? 'Randevu Düzenle' : 'Yeni Randevu Ekle' }}</h3>
        <div class="modal-date">{{ formatModalDate(selectedDate) }}</div>
        
        <form @submit.prevent="saveAppointment">
          <div class="form-group">
            <label for="patient-name">Hasta Adı:</label>
            <input 
              id="patient-name" 
              v-model="appointmentForm.patientName" 
              required 
              placeholder="Hasta adı giriniz"
            />
          </div>

          <div class="form-group">
            <label for="appointment-time">Randevu Saati:</label>
            <select id="appointment-time" v-model="appointmentForm.time" required>
              <option v-for="time in availableTimes" :key="time" :value="time">
                {{ time }}
              </option>
            </select>
          </div>

          <div class="form-group">
            <label for="appointment-duration">Süre (dakika):</label>
            <select id="appointment-duration" v-model="appointmentForm.duration">
              <option value="15">15 dakika</option>
              <option value="30">30 dakika</option>
              <option value="45">45 dakika</option>
              <option value="60">60 dakika</option>
            </select>
          </div>

          <div class="form-group">
            <label for="appointment-notes">Notlar:</label>
            <textarea 
              id="appointment-notes" 
              v-model="appointmentForm.notes"
              placeholder="Randevu ile ilgili notlar"
              rows="3"
            ></textarea>
          </div>

          <div class="form-actions">
            <button type="button" @click="closeAppointmentModal" class="cancel-btn">İptal</button>
            <button type="submit" class="save-btn">Kaydet</button>
            <button 
              v-if="editingAppointment" 
              type="button" 
              @click="deleteAppointment" 
              class="delete-btn"
            >
              Randevuyu Sil
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Gün Detay Modalı -->
    <div v-if="showDayModal" class="modal-backdrop" @click="closeDayModal">
      <div class="modal-content day-detail" @click.stop>
        <h3>{{ formatModalDate(selectedDate) }} Randevuları</h3>
        
        <div v-if="dayAppointments.length === 0" class="no-appointments">
          Bu gün için randevu bulunmamaktadır.
        </div>
        
        <div v-else class="appointment-list">
          <div 
            v-for="(appointment, index) in dayAppointments" 
            :key="index"
            class="appointment-item"
            @click="editAppointment(appointment)"
          >
            <div class="appointment-time">{{ appointment.time }}</div>
            <div class="appointment-patient">{{ appointment.patientName }}</div>
            <div class="appointment-duration">{{ appointment.duration }} dakika</div>
          </div>
        </div>
        
        <div class="day-actions">
          <button @click="addNewAppointment" class="add-appointment-btn">
            Yeni Randevu Ekle
          </button>
          <button @click="closeDayModal" class="close-btn">Kapat</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, reactive, watch } from 'vue';

// Temel veriler
const currentDate = ref(new Date());
const currentMonth = ref(new Date().getMonth());
const currentYear = ref(new Date().getFullYear());
const selectedDate = ref(null);
const selectedDoctor = ref(null);

// Doktor listesi (gerçek uygulamada API'den gelecek)
const doctors = ref([
  { id: 1, name: 'Dr. Ahmet Yılmaz', specialty: 'Kardiyoloji' },
  { id: 2, name: 'Dr. Ayşe Demir', specialty: 'Nöroloji' },
  { id: 3, name: 'Dr. Mehmet Kaya', specialty: 'Genel Cerrahi' }
]);

// Randevular (gerçek uygulamada API'den gelecek)
const appointments = ref([]);

// Modal durumları
const showDayModal = ref(false);
const showAppointmentModal = ref(false);
const editingAppointment = ref(null);

// Randevu formu
const appointmentForm = reactive({
  patientName: '',
  time: '09:00',
  duration: '30',
  notes: ''
});

// Sabit veriler
const weekdays = ['Pzt', 'Sal', 'Çar', 'Per', 'Cum', 'Cmt', 'Paz'];
const availableTimes = [
  '09:00', '09:30', '10:00', '10:30', '11:00', '11:30',
  '13:00', '13:30', '14:00', '14:30', '15:00', '15:30', '16:00', '16:30', '17:00'
];

// Ayın adını hesapla
const currentMonthName = computed(() => {
  const months = [
    'Ocak', 'Şubat', 'Mart', 'Nisan', 'Mayıs', 'Haziran',
    'Temmuz', 'Ağustos', 'Eylül', 'Ekim', 'Kasım', 'Aralık'
  ];
  return months[currentMonth.value];
});

// Takvim günlerini hesapla
const calendarDays = computed(() => {
  let days = [];
  
  // Ayın ilk gününü al
  const firstDay = new Date(currentYear.value, currentMonth.value, 1);
  
  // Ayın ilk gününün haftanın hangi günü olduğunu al (0: Pazar, 1: Pazartesi, ...)
  // Pazartesi başlangıç için uyarla (0: Pazartesi, ..., 6: Pazar)
  let firstDayOfWeek = firstDay.getDay() - 1;
  if (firstDayOfWeek < 0) firstDayOfWeek = 6;
  
  // Önceki ayın günlerini ekle
  const prevMonthLastDate = new Date(currentYear.value, currentMonth.value, 0).getDate();
  for (let i = 0; i < firstDayOfWeek; i++) {
    days.push({ date: null });
  }
  
  // Mevcut ayın günlerini ekle
  const lastDay = new Date(currentYear.value, currentMonth.value + 1, 0);
  const daysInMonth = lastDay.getDate();
  
  const today = new Date();
  
  for (let i = 1; i <= daysInMonth; i++) {
    const date = new Date(currentYear.value, currentMonth.value, i);
    
    // Bu güne ait randevuları bul
    const dayAppointments = appointments.value.filter(app => {
      const appDate = new Date(app.date);
      return appDate.getDate() === i && 
             appDate.getMonth() === currentMonth.value && 
             appDate.getFullYear() === currentYear.value;
    });
    
    days.push({
      date: date,
      isToday: 
        today.getDate() === i && 
        today.getMonth() === currentMonth.value && 
        today.getFullYear() === currentYear.value,
      appointments: dayAppointments
    });
  }
  
  // Sonraki ayın günlerini ekle (6 satırlık takvim için)
  const totalDaysAdded = days.length;
  const rowsNeeded = Math.ceil(totalDaysAdded / 7);
  const cellsNeeded = rowsNeeded * 7;
  
  for (let i = totalDaysAdded; i < cellsNeeded; i++) {
    days.push({ date: null });
  }
  
  return days;
});

// Seçili günün randevularını hesapla
const dayAppointments = computed(() => {
  if (!selectedDate.value) return [];
  
  return appointments.value.filter(app => {
    const appDate = new Date(app.date);
    return appDate.getDate() === selectedDate.value.getDate() && 
           appDate.getMonth() === selectedDate.value.getMonth() && 
           appDate.getFullYear() === selectedDate.value.getFullYear();
  });
});

// Component yüklendiğinde ilk doktoru seç
onMounted(() => {
  if (doctors.value.length > 0) {
    selectedDoctor.value = doctors.value[0].id;
    loadAppointments();
  }
});

// Metotlar
const prevMonth = () => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value--;
  } else {
    currentMonth.value--;
  }
  loadAppointments();
};

const nextMonth = () => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }
  loadAppointments();
};

const loadAppointments = () => {
  // Burada gerçek uygulamada API'den randevuları yükleyeceksiniz
  // Şimdilik örnek veri kullanalım
  
  // Örnek veriler (gerçek uygulamada API'den gelecek)
  appointments.value = [
    {
      id: 1,
      doctorId: 1,
      patientName: 'Ali Yıldız',
      date: new Date(currentYear.value, currentMonth.value, 15),
      time: '10:00',
      duration: '30',
      notes: 'Kontrol randevusu'
    },
    {
      id: 2,
      doctorId: 1,
      patientName: 'Zeynep Kara',
      date: new Date(currentYear.value, currentMonth.value, 15),
      time: '14:30',
      duration: '45',
      notes: 'İlk muayene'
    },
    {
      id: 3,
      doctorId: 2,
      patientName: 'Mustafa Şahin',
      date: new Date(currentYear.value, currentMonth.value, 10),
      time: '09:30',
      duration: '30',
      notes: ''
    }
  ].filter(app => app.doctorId === selectedDoctor.value);
};

const openDayDetail = (day) => {
  if (!day.date) return;
  
  selectedDate.value = day.date;
  showDayModal.value = true;
};

const closeDayModal = () => {
  showDayModal.value = false;
  selectedDate.value = null;
};

const addNewAppointment = () => {
  // Yeni randevu ekleme formunu aç
  resetAppointmentForm();
  editingAppointment.value = null;
  showDayModal.value = false;
  showAppointmentModal.value = true;
};

const editAppointment = (appointment) => {
  // Randevu düzenleme formunu aç
  editingAppointment.value = appointment;
  appointmentForm.patientName = appointment.patientName;
  appointmentForm.time = appointment.time;
  appointmentForm.duration = appointment.duration;
  appointmentForm.notes = appointment.notes;
  
  showDayModal.value = false;
  showAppointmentModal.value = true;
};

const closeAppointmentModal = () => {
  showAppointmentModal.value = false;
  editingAppointment.value = null;
  // Gün modalı açık idiyse onu tekrar aç
  if (selectedDate.value) {
    showDayModal.value = true;
  }
};

const saveAppointment = () => {
  if (editingAppointment.value) {
    // Mevcut randevuyu güncelle
    const index = appointments.value.findIndex(app => app.id === editingAppointment.value.id);
    if (index !== -1) {
      appointments.value[index] = {
        ...editingAppointment.value,
        patientName: appointmentForm.patientName,
        time: appointmentForm.time,
        duration: appointmentForm.duration,
        notes: appointmentForm.notes
      };
    }
  } else {
    // Yeni randevu ekle
    const newId = Math.max(0, ...appointments.value.map(app => app.id)) + 1;
    appointments.value.push({
      id: newId,
      doctorId: selectedDoctor.value,
      patientName: appointmentForm.patientName,
      date: new Date(selectedDate.value),
      time: appointmentForm.time,
      duration: appointmentForm.duration,
      notes: appointmentForm.notes
    });
  }
  
  // Modalı kapat ve formu sıfırla
  closeAppointmentModal();
  resetAppointmentForm();
};

const deleteAppointment = () => {
  if (confirm('Bu randevuyu silmek istediğinizden emin misiniz?')) {
    const index = appointments.value.findIndex(app => app.id === editingAppointment.value.id);
    if (index !== -1) {
      appointments.value.splice(index, 1);
    }
    closeAppointmentModal();
  }
};

const resetAppointmentForm = () => {
  appointmentForm.patientName = '';
  appointmentForm.time = '09:00';
  appointmentForm.duration = '30';
  appointmentForm.notes = '';
};

const formatModalDate = (date) => {
  if (!date) return '';
  
  const day = date.getDate();
  const month = currentMonthName.value;
  const year = date.getFullYear();
  
  return `${day} ${month} ${year}`;
};

// Doktor değişikliğini izleyin
watch(selectedDoctor, () => {
  loadAppointments();
});
</script>

<style scoped>
.calendar-container {
  max-width: 1000px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

.calendar-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.calendar-controls {
  display: flex;
  align-items: center;
  margin: 10px 0 20px;
}

.current-month {
  font-size: 1.5rem;
  font-weight: bold;
  margin: 0 20px;
  min-width: 200px;
  text-align: center;
}

.control-btn {
  background-color: #4CAF50;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.control-btn:hover {
  background-color: #45a049;
}

.doctor-selector {
  margin-bottom: 15px;
  width: 100%;
  max-width: 400px;
}

.doctor-selector select {
  width: 100%;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #ddd;
  font-size: 1rem;
}

.weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  background-color: #f5f5f5;
  border-top: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
}

.weekday {
  text-align: center;
  padding: 10px;
  font-weight: bold;
}

.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  grid-gap: 1px;
  background-color: #f0f0f0;
}

.calendar-day {
  min-height: 100px;
  background-color: white;
  padding: 5px;
  border: 1px solid #eee;
  cursor: pointer;
  position: relative;
}

.calendar-day:hover {
  background-color: #f9f9f9;
}

.empty {
  background-color: #f9f9f9;
  cursor: default;
}

.today {
  background-color: #e8f5e9;
}

.has-appointments {
  background-color: #e3f2fd;
}

.day-number {
  font-weight: bold;
  margin-bottom: 5px;
}

.appointment-count {
  font-size: 0.8rem;
  color: #2196F3;
}

/* Modal Stilleri */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  width: 90%;
  max-width: 500px;
}

.day-detail {
  width: 90%;
  max-width: 600px;
}

.modal-date {
  font-size: 1.2rem;
  margin-bottom: 20px;
  color: #333;
  text-align: center;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.form-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.save-btn {
  background-color: #4CAF50;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.save-btn:hover {
  background-color: #45a049;
}

.cancel-btn {
  background-color: #f44336;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.cancel-btn:hover {
  background-color: #d32f2f;
}

.delete-btn {
  background-color: #f44336;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.delete-btn:hover {
  background-color: #d32f2f;
}

.no-appointments {
  text-align: center;
  padding: 20px;
  color: #666;
}

.appointment-list {
  max-height: 300px;
  overflow-y: auto;
}

.appointment-item {
  display: flex;
  padding: 10px;
  border-bottom: 1px solid #eee;
  cursor: pointer;
}

.appointment-item:hover {
  background-color: #f5f5f5;
}

.appointment-time {
  font-weight: bold;
  width: 80px;
}

.appointment-patient {
  flex-grow: 1;
}

.appointment-duration {
  color: #666;
  width: 100px;
  text-align: right;
}

.day-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.add-appointment-btn {
  background-color: #2196F3;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.add-appointment-btn:hover {
  background-color: #1976D2;
}

.close-btn {
  background-color: #9E9E9E;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.close-btn:hover {
  background-color: #757575;
}
</style>