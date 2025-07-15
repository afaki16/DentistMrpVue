<!-- pages/contact.vue -->
<template>
  <div class="contact">
    <h1>İletişim</h1>
    <p>Benimle iletişime geçebilirsiniz.</p>
    
    <form @submit.prevent="handleSubmit" class="contact-form">
      <div class="form-group">
        <label for="name">Ad:</label>
        <input 
          type="text" 
          id="name" 
          v-model="form.name" 
          required
          placeholder="Adınızı giriniz"
        >
      </div>
      
      <div class="form-group">
        <label for="email">Email:</label>
        <input 
          type="email" 
          id="email" 
          v-model="form.email" 
          required
          placeholder="Email adresinizi giriniz"
        >
      </div>
      
      <div class="form-group">
        <label for="message">Mesaj:</label>
        <textarea 
          id="message" 
          v-model="form.message" 
          required
          placeholder="Mesajınızı yazınız"
        ></textarea>
      </div>
      
      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? 'Gönderiliyor...' : 'Gönder' }}
      </button>
    </form>
  </div>
</template>

<script setup>
// Nuxt 3 Composition API
const form = ref({
  name: '',
  email: '',
  message: ''
})

const isSubmitting = ref(false)

useSeoMeta({
  title: 'İletişim',
  description: 'İletişim formu'
})

const handleSubmit = async () => {
  isSubmitting.value = true
  
  try {
    // Form verilerini işle
    await new Promise(resolve => setTimeout(resolve, 1000)) // Simüle edilmiş API çağrısı
    
    alert('Form başarıyla gönderildi!')
    console.log('Form verileri:', form.value)
    
    // Formu temizle
    form.value = {
      name: '',
      email: '',
      message: ''
    }
  } catch (error) {
    alert('Hata oluştu!')
  } finally {
    isSubmitting.value = false
  }
}
</script>

<style scoped>
.contact {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
}

.contact-form {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #2c3e50;
}

input, textarea {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e9ecef;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

input:focus, textarea:focus {
  outline: none;
  border-color: #42b883;
}

textarea {
  height: 120px;
  resize: vertical;
}

button {
  background: linear-gradient(135deg, #42b883, #369870);
  color: white;
  padding: 0.75rem 2rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: transform 0.2s, box-shadow 0.2s;
}

button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.3);
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>