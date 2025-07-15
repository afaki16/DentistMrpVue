<template>
  <div class="multi-doctor-calendar">
    <div class="doctors-legend">
      <div>
        <div class="calendar-controls">
          <button @click="goToPrevious" class="nav-btn">
            <span>⬅️</span> Önceki
          </button>
          <button @click="goToToday" class="today-btn">
            <span>🏠</span> Bugün
          </button>
          <button @click="goToNext" class="nav-btn">
            Sonraki <span>➡️</span>
          </button>
        </div>
      </div>
    </div>
    <div class="calendar-container">
      <FullCalendar
        ref="fullCalendar"
        :options="calendarOptions"
      />
      
    </div>

    <!-- Tarih Modal -->
    <div v-if="isModalOpenDate" class="modal-overlay" @click="closeDetailModal">
      <div class="modal-content detail-modal" @click.stop>

        <div class="modal-header">
          <h3> Tarih Seçin</h3>
        </div>
          <!-- Tarih Seçici -->
           <VueDatePicker 
    v-model="selectedDate"
      
      :action-row="false"
      :auto-apply="true"
      locale="tr"
      :inline="true"
      :enable-time-picker="false"
      @update:model-value="goToSelectedDate"
      :min-date="new Date()"
      class="calendar-controls"
  />
      </div>
      </div>
         <!-- Doktor Detay Modal -->
    <div v-if="isModalOpen" class="modal-overlay" @click="closeDetailModal">
      <div class="modal-content detail-modal" @click.stop>
        <h3>📋 Doktor Detayları</h3>
       <!-- Doktor Legendası -->
    <div class="doctors-legend">
      <div class="legend-header">
        <h3>👥 Hastane Doktorları & Randevu Dağılımı</h3>
        <div class="legend-stats">
          <span class="total-appointments">📊 Toplam: {{ allAppointments.length }} randevu</span>
          
        </div>
      </div>
      
      <div class="legend-items">
        <!-- Hepsi kartı -->
        <div class="legend-item" 
             :class="{ 'active': selectedDoctor === 'all', 'highlighted': true }"
             @click="selectDoctor('all')">
          <div class="all-doctors-icon">
            <span>🏥</span>
          </div>
          <div class="doctor-info">
            <span class="doctor-name">Hepsi</span>
            <span class="doctor-specialty">Tüm Branşlar</span>
            <span class="doctor-room">Genel Görünüm</span>
          </div>
          <div class="doctor-stats">
            <span class="appointment-count">{{ allAppointments.length }} randevu</span>
            <span class="percentage">%100</span>
          </div>
        </div>
        
        <!-- Doktor kartları -->
        <div v-for="doctor in doctors" :key="doctor.id" class="legend-item" 
             :class="{ 'active': selectedDoctor === doctor.id, 'highlighted': selectedDoctor === 'all' }"
             @click="selectDoctor(doctor.id)">
          <div class="doctor-photo">
            <img :src="doctor.photo" :alt="doctor.name" />
          </div>
          <div class="doctor-info">
            <span class="doctor-name">{{ doctor.name }}</span>
            <span class="doctor-specialty">{{ doctor.specialty }}</span>
            <span class="doctor-room">{{ doctor.room }}</span>
          </div>
          <div class="doctor-stats">
            <span class="appointment-count">{{ getDoctorAppointmentCount(doctor.id) }} randevu</span>
            <span class="total-appointments">Aktif</span>
          </div>
        </div>
      </div>
    </div>
       
      </div>
    </div>
    <!-- Randevu Ekleme Modal -->
    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
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
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, nextTick } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'
import trLocale from '@fullcalendar/core/locales/tr'
import VueDatePicker from '@vuepic/vue-datepicker'
import '@vuepic/vue-datepicker/dist/main.css'

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
const isModalOpen = ref(false)
const isModalOpenDate = ref(false)


// Data
const doctors = ref([
  {
    id: 'dr1',
    name: 'Dr. Mehmet Yılmaz',
    specialty: 'Dahiliye',
    room: 'Oda 101',
    color: '#4CAF50',
    photo: 'https://images.unsplash.com/photo-1612349317150-e413f6a5b16d?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr2',
    name: 'Dr. Ayşe Demir',
    specialty: 'Kardiyoloji',
    room: 'Oda 205',
    color: '#2196F3',
    photo: 'https://images.unsplash.com/photo-1559839734-2b71ea197ec2?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr3',
    name: 'Dr. Oğuz Çelik',
    specialty: 'Ortopedi',
    room: 'Oda 308',
    color: '#FF9800',
    photo: 'https://images.unsplash.com/photo-1582750433449-648ed127bb54?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr4',
    name: 'Dr. Zeynep Kaya',
    specialty: 'Göz Hastalıkları',
    room: 'Oda 412',
    color: '#9C27B0',
    photo: 'https://images.unsplash.com/photo-1594824204080-7b84f95e8936?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr5',
    name: 'Dr. Kemal Özkan',
    specialty: 'KBB',
    room: 'Oda 515',
    color: '#F44336',
    photo: 'https://images.unsplash.com/photo-1622253692010-333f2da6031d?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr6',
    name: 'Dr. Fatma Şahin',
    specialty: 'Dermatoloji',
    room: 'Oda 620',
    color: '#607D8B',
    photo: 'https://images.unsplash.com/photo-1564564321837-a57b7070ac4f?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr7',
    name: 'Dr. İbrahim Arslan',
    specialty: 'Üroloji',
    room: 'Oda 718',
    color: '#795548',
    photo: 'https://images.unsplash.com/photo-1537368910025-700350fe46c7?w=150&h=150&fit=crop&crop=face'
  },
  {
    id: 'dr8',
    name: 'Dr. Elif Özkan',
    specialty: 'Jinekolog',
    room: 'Oda 825',
    color: '#E91E63',
    photo: 'https://images.unsplash.com/photo-1603415526960-f7e0328c63b1?w=150&h=150&fit=crop&crop=face'
  }
])

const allAppointments = ref([
  // Dr. Mehmet Yılmaz - Dahiliye (Yeşil)
  {
    id: '1',
    doctorId: 'dr1',
    title: 'Ahmet Yılmaz - Genel Muayene',
    start: '2025-07-14T09:00:00',
    end: '2025-07-14T09:30:00',
    backgroundColor: '#4CAF50',
    borderColor: '#4CAF50',
    extendedProps: {
      patientName: 'Ahmet Yılmaz',
      appointmentType: 'Genel Muayene',
      notes: 'Rutin kontrol'
    }
  },
  {
    id: '2',
    doctorId: 'dr1',
    title: 'Fatma Demir - Kontrol',
    start: '2025-07-14T10:30:00',
    end: '2025-07-14T11:00:00',
    backgroundColor: '#4CAF50',
    borderColor: '#4CAF50',
    extendedProps: {
      patientName: 'Fatma Demir',
      appointmentType: 'Kontrol',
      notes: 'Diyabet kontrolü'
    }
  },
  // Dr. Ayşe Demir - Kardiyoloji (Mavi)
  {
    id: '3',
    doctorId: 'dr2',
    title: 'Mehmet Öz - EKG',
    start: '2025-07-14T09:30:00',
    end: '2025-07-14T10:00:00',
    backgroundColor: '#2196F3',
    borderColor: '#2196F3',
    extendedProps: {
      patientName: 'Mehmet Öz',
      appointmentType: 'Laboratuvar',
      notes: 'EKG çekimi'
    }
  },
  {
    id: '4',
    doctorId: 'dr2',
    title: 'Ayşe Kaya - Kalp Kontrolü',
    start: '2025-07-14T14:00:00',
    end: '2025-07-14T14:30:00',
    backgroundColor: '#2196F3',
    borderColor: '#2196F3',
    extendedProps: {
      patientName: 'Ayşe Kaya',
      appointmentType: 'Kontrol',
      notes: 'Kalp ritmi kontrolü'
    }
  },
  // Dr. Oğuz Çelik - Ortopedi (Turuncu)
  {
    id: '5',
    doctorId: 'dr3',
    title: 'Hasan Çelik - Fizik Tedavi',
    start: '2025-07-14T11:00:00',
    end: '2025-07-14T11:30:00',
    backgroundColor: '#FF9800',
    borderColor: '#FF9800',
    extendedProps: {
      patientName: 'Hasan Çelik',
      appointmentType: 'Tedavi',
      notes: 'Fizik tedavi seansı'
    }
  },
  {
    id: '6',
    doctorId: 'dr3',
    title: 'Zeynep Özkan - Diz Kontrolü',
    start: '2025-07-14T15:30:00',
    end: '2025-07-14T16:00:00',
    backgroundColor: '#FF9800',
    borderColor: '#FF9800',
    extendedProps: {
      patientName: 'Zeynep Özkan',
      appointmentType: 'Kontrol',
      notes: 'Diz ağrısı kontrolü'
    }
  },
  // Dr. Zeynep Kaya - Göz Hastalıkları (Mor)
  {
    id: '7',
    doctorId: 'dr4',
    title: 'Ali Vural - Göz Muayenesi',
    start: '2025-07-14T10:00:00',
    end: '2025-07-14T10:30:00',
    backgroundColor: '#9C27B0',
    borderColor: '#9C27B0',
    extendedProps: {
      patientName: 'Ali Vural',
      appointmentType: 'Genel Muayene',
      notes: 'Görme keskinliği testi'
    }
  },
  {
    id: '8',
    doctorId: 'dr4',
    title: 'Meryem Şahin - Katarakt Kontrolü',
    start: '2025-07-14T16:30:00',
    end: '2025-07-14T17:00:00',
    backgroundColor: '#9C27B0',
    borderColor: '#9C27B0',
    extendedProps: {
      patientName: 'Meryem Şahin',
      appointmentType: 'Kontrol',
      notes: 'Katarakt ameliyatı kontrolü'
    }
  },
  // Dr. Kemal Özkan - KBB (Kırmızı)
  {
    id: '9',
    doctorId: 'dr5',
    title: 'Oğuz Yıldız - Kulak Ağrısı',
    start: '2025-07-14T08:30:00',
    end: '2025-07-14T09:00:00',
    backgroundColor: '#F44336',
    borderColor: '#F44336',
    extendedProps: {
      patientName: 'Oğuz Yıldız',
      appointmentType: 'Genel Muayene',
      notes: 'Kulak ağrısı şikayeti'
    }
  },
  {
    id: '10',
    doctorId: 'dr5',
    title: 'Elif Aydın - Sinüzit Tedavi',
    start: '2025-07-14T13:00:00',
    end: '2025-07-14T13:30:00',
    backgroundColor: '#F44336',
    borderColor: '#F44336',
    extendedProps: {
      patientName: 'Elif Aydın',
      appointmentType: 'Tedavi',
      notes: 'Sinüzit tedavi kontrolü'
    }
  },
  // Ek randevular farklı günlerde
  {
    id: '11',
    doctorId: 'dr1',
    title: 'Mustafa Karaca - Şeker Kontrolü',
    start: '2025-07-15T09:00:00',
    end: '2025-07-15T09:30:00',
    backgroundColor: '#4CAF50',
    borderColor: '#4CAF50',
    extendedProps: {
      patientName: 'Mustafa Karaca',
      appointmentType: 'Kontrol',
      notes: 'Diyabet takibi'
    }
  },
  {
    id: '12',
    doctorId: 'dr2',
    title: 'Seda Aksoy - Tansiyon Kontrolü',
    start: '2025-07-15T14:30:00',
    end: '2025-07-15T15:00:00',
    backgroundColor: '#2196F3',
    borderColor: '#2196F3',
    extendedProps: {
      patientName: 'Seda Aksoy',
      appointmentType: 'Kontrol',
      notes: 'Hipertansiyon kontrolü'
    }
  },
  // Ek randevular daha fazla çeşitlilik için
  {
    id: '15',
    doctorId: 'dr4',
    title: 'Canan Özdemir - Lens Kontrolü',
    start: '2025-07-16T11:00:00',
    end: '2025-07-16T11:30:00',
    backgroundColor: '#9C27B0',
    borderColor: '#9C27B0',
    extendedProps: {
      patientName: 'Canan Özdemir',
      appointmentType: 'Kontrol',
      notes: 'Lens kontrolü ve temizlik'
    }
  },
  {
    id: '16',
    doctorId: 'dr5',
    title: 'Erkan Demirci - Burun Estetiği Konsültasyonu',
    start: '2025-07-16T15:00:00',
    end: '2025-07-16T15:30:00',
    backgroundColor: '#F44336',
    borderColor: '#F44336',
    extendedProps: {
      patientName: 'Erkan Demirci',
      appointmentType: 'Genel Muayene',
      notes: 'Burun estetiği konsültasyonu'
    }
  },
  {
    id: '17',
    doctorId: 'dr2',
    title: 'Şeyma Yıldız - Holter Takibi',
    start: '2025-07-17T08:30:00',
    end: '2025-07-17T09:00:00',
    backgroundColor: '#2196F3',
    borderColor: '#2196F3',
    extendedProps: {
      patientName: 'Şeyma Yıldız',
      appointmentType: 'Kontrol',
      notes: '24 saatlik Holter sonucu değerlendirmesi'
    }
  },
  {
    id: '18',
    doctorId: 'dr3',
    title: 'Barış Akar - Omuz MR Değerlendirmesi',
    start: '2025-07-17T10:30:00',
    end: '2025-07-17T11:00:00',
    backgroundColor: '#FF9800',
    borderColor: '#FF9800',
    extendedProps: {
      patientName: 'Barış Akar',
      appointmentType: 'Kontrol',
      notes: 'Omuz MR sonuçları değerlendirmesi'
    }
  },
  {
    id: '19',
    doctorId: 'dr1',
    title: 'Derya Çalışkan - Tiroid Kontrolü',
    start: '2025-07-17T13:30:00',
    end: '2025-07-17T14:00:00',
    backgroundColor: '#4CAF50',
    borderColor: '#4CAF50',
    extendedProps: {
      patientName: 'Derya Çalışkan',
      appointmentType: 'Kontrol',
      notes: 'Tiroid fonksiyon testleri kontrolü'
    }
  },

  {
    id: '21',
    doctorId: 'dr6',
    title: 'Melike Çetin - Akne Tedavisi',
    start: '2025-07-14T10:00:00',
    end: '2025-07-14T10:30:00',
    backgroundColor: '#607D8B',
    borderColor: '#607D8B',
    extendedProps: {
      patientName: 'Melike Çetin',
      appointmentType: 'Tedavi',
      notes: 'Akne tedavi seansı'
    }
  },
  {
    id: '22',
    doctorId: 'dr6',
    title: 'Volkan Erdoğan - Saç Dökülmesi',
    start: '2025-07-14T14:30:00',
    end: '2025-07-14T15:00:00',
    backgroundColor: '#607D8B',
    borderColor: '#607D8B',
    extendedProps: {
      patientName: 'Volkan Erdoğan',
      appointmentType: 'Genel Muayene',
      notes: 'Saç dökülmesi konsültasyonu'
    }
  },
  {
    id: '23',
    doctorId: 'dr7',
    title: 'Recep Tayyar - Prostat Kontrolü',
    start: '2025-07-14T11:30:00',
    end: '2025-07-14T12:00:00',
    backgroundColor: '#795548',
    borderColor: '#795548',
    extendedProps: {
      patientName: 'Recep Tayyar',
      appointmentType: 'Kontrol',
      notes: 'Prostat kontrolü'
    }
  },
  {
    id: '24',
    doctorId: 'dr7',
    title: 'Serkan Kaya - Böbrek Taşı',
    start: '2025-07-14T16:00:00',
    end: '2025-07-14T16:30:00',
    backgroundColor: '#795548',
    borderColor: '#795548',
    extendedProps: {
      patientName: 'Serkan Kaya',
      appointmentType: 'Tedavi',
      notes: 'Böbrek taşı tedavisi'
    }
  },
  {
    id: '25',
    doctorId: 'dr8',
    title: 'Gülseren Özdemir - Gebelik Kontrolü',
    start: '2025-07-14T09:30:00',
    end: '2025-07-14T10:00:00',
    backgroundColor: '#E91E63',
    borderColor: '#E91E63',
    extendedProps: {
      patientName: 'Gülseren Özdemir',
      appointmentType: 'Kontrol',
      notes: '20 haftalık gebelik kontrolü'
    }
  },
  {
    id: '26',
    doctorId: 'dr8',
    title: 'Aylin Şimşek - Smear Testi',
    start: '2025-07-14T15:30:00',
    end: '2025-07-14T16:00:00',
    backgroundColor: '#E91E63',
    borderColor: '#E91E63',
    extendedProps: {
      patientName: 'Aylin Şimşek',
      appointmentType: 'Laboratuvar',
      notes: 'Rutin smear testi'
    }
  },
  {
    id: '27',
    doctorId: 'dr6',
    title: 'Deniz Akar - Egzama Kontrolü',
    start: '2025-07-15T10:00:00',
    end: '2025-07-15T10:30:00',
    backgroundColor: '#607D8B',
    borderColor: '#607D8B',
    extendedProps: {
      patientName: 'Deniz Akar',
      appointmentType: 'Kontrol',
      notes: 'Egzama takibi'
    }
  },
  {
    id: '28',
    doctorId: 'dr7',
    title: 'Fikret Yıldız - Sistoskopi',
    start: '2025-07-15T13:00:00',
    end: '2025-07-15T13:30:00',
    backgroundColor: '#795548',
    borderColor: '#795548',
    extendedProps: {
      patientName: 'Fikret Yıldız',
      appointmentType: 'Tedavi',
      notes: 'Sistoskopi işlemi'
    }
  },
  {
    id: '29',
    doctorId: 'dr8',
    title: 'Serpil Coşkun - Menopoz Konsültasyonu',
    start: '2025-07-15T11:30:00',
    end: '2025-07-15T12:00:00',
    backgroundColor: '#E91E63',
    borderColor: '#E91E63',
    extendedProps: {
      patientName: 'Serpil Coşkun',
      appointmentType: 'Genel Muayene',
      notes: 'Menopoz semptomları değerlendirmesi'
    }
  }
])

// Calendar Options
const calendarOptions = reactive({
  plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
  initialView: 'timeGridWeek',
  locale: trLocale,
  headerToolbar: {
    left: 'addEventBtn,filterBtn',
    center: 'title',
    right: 'dayGridMonth,timeGridWeek,timeGridDay'
  },
   customButtons: {
    addEventBtn: {
      text: '🔍 Filtrele',
      click: function() {
          isModalOpen.value = true
      }
    },
    filterBtn: {
      text: '📍 Tarihe Git',
      click: function() {
         isModalOpenDate.value = true
       
      }
    }
},
  titleFormat: {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  },
  height: 'auto',
  slotMinTime: '08:00:00',
  slotMaxTime: '21:00:00',
  slotDuration: '00:30:00',
  slotLabelInterval: '01:00:00',
  allDaySlot: false,
  businessHours: {
    daysOfWeek: [1, 2, 3, 4, 5, 6],
    startTime: '08:00',
    endTime: '21:00'
  },
  selectable: true,
  selectMirror: true,
  select: handleDateSelect,
  eventClick: handleEventClick,
  editable: false,
  dayMaxEvents: false,
  weekends: true,
  events: allAppointments.value,
  slotLabelFormat: {
    hour: '2-digit',
    minute: '2-digit',
    hour12: false
  },
  eventTimeFormat: {
    hour: '2-digit',
    minute: '2-digit',
    hour12: false
  },
  nowIndicator: true,
  scrollTime: '08:00:00',
  eventDisplay: 'block',
  eventOverlap: true,
  selectOverlap: true
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
  isModalOpen.value = false
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
  isModalOpen.value = false
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
  isModalOpenDate.value = false
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
  justify-content: center;
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

@media (max-width: 1200px) {
  .legend-items {
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  }
}

@media (max-width: 768px) {
  .calendar-header {
    flex-direction: column;
    gap: 20px;
    align-items: center;
    text-align: center;
  }
  
  .header-left {
    width: 100%;
    align-items: center;
  }
  
  .header-right {
    align-items: center;
    width: 100%;
  }
  
  .date-picker-container {
    flex-direction: column;
    gap: 8px;
    width: 100%;
    justify-content: center;
  }
  
  .doctor-select, .date-picker {
    min-width: auto;
    width: 100%;
  }
  
  .calendar-controls {
    width: 100%;
    justify-content: center;
    flex-wrap: wrap;
  }
  
  .modal-content {
    width: 95%;
    margin: 10px;
    padding: 20px;
  }
  
  .multi-doctor-calendar {
    padding: 15px;
  }
  
  .legend-items {
    grid-template-columns: 1fr;
  }
  
  .legend-item {
    flex-direction: column;
    text-align: center;
    gap: 15px;
    padding: 25px;
  }
  
  .doctor-photo, .all-doctors-icon {
    width: 80px;
    height: 80px;
  }
  
  .all-doctors-icon span {
    font-size: 28px;
  }
  
  .doctor-stats {
    align-items: center;
  }
  
  :deep(.fc-header-toolbar) {
    flex-direction: column;
    gap: 15px;
  }
  
  :deep(.fc-toolbar-title) {
    font-size: 20px !important;
  }
} 
</style>