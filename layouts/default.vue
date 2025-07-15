<template>
  <div class="app-layout">
    <header class="header">
      <nav class="navbar">
        <NuxtLink to="/" class="logo">
          <span>🚀 Nuxt App</span>
        </NuxtLink>
       
        <ul class="nav-links">
          <li>
            <NuxtLink to="/" class="nav-link">Ana Sayfa</NuxtLink>
          </li>
          <li>
            <NuxtLink to="/about" class="nav-link">Hakkımda</NuxtLink>
          </li>
          <li>
            <NuxtLink to="/about2" class="nav-link">Diğer</NuxtLink>
          </li>
          <li>
            <NuxtLink to="/about3" class="nav-link">Diğer2</NuxtLink>
          </li>
          <li>
            <NuxtLink to="/about4" class="nav-link">Diğer3</NuxtLink>
          </li>
        </ul>
      </nav>
    </header>
   
    <main class="main-content">
      <slot />
    </main>
   
    <footer class="footer">
      <div class="footer-content">
        <div class="datetime-container">
          <p class="current-datetime">
            <span class="date">{{ currentDate }}</span>
            <span class="time">{{ currentTime }}</span>
          </p>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup>
const currentDate = ref('')
const currentTime = ref('')

const updateDateTime = () => {
  const now = new Date()
  
  // Tarih formatı: 15 Temmuz 2025, Salı
  const dateOptions = {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    weekday: 'long',
    timeZone: 'Europe/Istanbul'
  }
  
  // Saat formatı: 14:30:45
  const timeOptions = {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    timeZone: 'Europe/Istanbul'
  }
  
  currentDate.value = now.toLocaleDateString('tr-TR', dateOptions)
  currentTime.value = now.toLocaleTimeString('tr-TR', timeOptions)
}

let interval = null

onMounted(() => {
  updateDateTime()
  // Her saniye güncelle
  interval = setInterval(updateDateTime, 1000)
})

onUnmounted(() => {
  if (interval) {
    clearInterval(interval)
  }
})
</script>

<style scoped>
.app-layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background: linear-gradient(135deg, #2c3e50, #3498db);
  color: white;
  padding: 1rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
}

.navbar {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 2rem;
}

.logo {
  font-size: 1.5rem;
  font-weight: bold;
  text-decoration: none;
  color: white;
}

.nav-links {
  display: flex;
  list-style: none;
  gap: 2rem;
  margin: 0;
  padding: 0;
}

.nav-link {
  color: white;
  text-decoration: none;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  transition: all 0.3s ease;
  position: relative;
}

.nav-link:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: translateY(-1px);
}

.nav-link.router-link-active {
  background: rgba(255, 255, 255, 0.2);
  font-weight: 600;
}

.main-content {
  flex: 1;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  min-height: calc(100vh - 140px);
  padding-bottom: 120px; /* Footer yüksekliği kadar boşluk */
}

.footer {
  background: #2c3e50;
  color: white;
  padding: 0.25rem 0.25rem;
  margin-top: auto;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 30;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.footer-content p {
  margin: 0;
}

.datetime-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.current-datetime {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
  font-size: 0.9rem;
  color: #ecf0f1;
}

.date {
  font-weight: 500;
  color: #3498db;
}

.time {
  font-weight: 600;
  color: #e74c3c;
  font-family: 'Courier New', monospace;
  font-size: 1rem;
}

/* Responsive */
@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    gap: 1rem;
  }
 
  .nav-links {
    gap: 1rem;
  }
  
  .footer-content {
    text-align: center;
  }
  
  .current-datetime {
    font-size: 0.8rem;
  }
  
  .time {
    font-size: 0.9rem;
  }
  
  .main-content {
    padding-bottom: 140px; /* Mobilde biraz daha fazla boşluk */
  }
}

@media (max-width: 480px) {
  .current-datetime {
    flex-direction: row;
    gap: 1rem;
  }
}
</style>