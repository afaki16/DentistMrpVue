<template>
  <div>
    <!-- Modal Açma Butonu -->
    <button 
      @click="openModal"
      class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded transition-colors"
    >
      Modal Aç
    </button>

    <!-- Modal Overlay -->
    <div 
      v-if="isModalOpen" 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
      @click="closeModal"
    >
      <!-- Modal Content -->
      <div 
        class="bg-white rounded-lg p-6 max-w-md w-full mx-4 transform transition-all"
        @click.stop
      >
        <!-- Modal Header -->
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-xl font-bold text-gray-800">Modal Başlığı</h2>
          <button 
            @click="closeModal"
            class="text-gray-500 hover:text-gray-700 text-2xl"
          >
            &times;
          </button>
        </div>

        <!-- Modal Body -->
        <div class="mb-6">
          <p class="text-gray-600">
            Bu bir modal örneğidir. Burada istediğiniz içeriği gösterebilirsiniz.
          </p>
          <p class="text-gray-600 mt-2">
            Modal dışına tıklayarak veya X butonuna basarak kapatabilirsiniz.
          </p>
        </div>

        <!-- Modal Footer -->
        <div class="flex justify-end space-x-2">
          <button 
            @click="closeModal"
            class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded transition-colors"
          >
            İptal
          </button>
          <button 
            @click="handleConfirm"
            class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded transition-colors"
          >
            Onayla
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Modal durumu
const isModalOpen = ref(false)

// Modal açma fonksiyonu
const openModal = () => {
  isModalOpen.value = true
}

// Modal kapatma fonksiyonu
const closeModal = () => {
  isModalOpen.value = false
}

// Onay butonu fonksiyonu
const handleConfirm = () => {
  // Burada onay işlemlerini yapabilirsiniz
  console.log('Modal onaylandı!')
  closeModal()
}

// ESC tuşu ile modal kapatma
const handleKeydown = (event) => {
  if (event.key === 'Escape' && isModalOpen.value) {
    closeModal()
  }
}

// Klavye event listener'ını ekle
onMounted(() => {
  document.addEventListener('keydown', handleKeydown)
})

// Cleanup
onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
/* Modal animasyonu için geçiş efektleri */
.v-enter-active, .v-leave-active {
  transition: opacity 0.3s ease;
}

.v-enter-from, .v-leave-to {
  opacity: 0;
}

/* Modal içeriği için animasyon */
.transform {
  transition: transform 0.3s ease;
}
</style>