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
            <NuxtLink to="/contact" class="nav-link">İletişim</NuxtLink>
          </li>
        </ul>
      </nav>
    </header>
    
    <main class="main-content">
      <slot />
    </main>
    
    <footer class="footer">
      <p>&copy; 2025 Nuxt.js Uygulaması</p>
    </footer>
  </div>
</template>

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
}

.footer {
  background: #2c3e50;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
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
}
</style>

