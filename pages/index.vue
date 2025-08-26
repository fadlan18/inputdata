<template>
  <div class="app-container">
    <!-- Header -->
    <header class="bg-green-600 text-white p-4 text-center font-bold text-lg">
      header
    </header>

    <div class="main-content flex flex-1">
      <!-- Sidebar -->
      <aside class="bg-green-500 text-white w-48 p-4 flex flex-col space-y-4">
        <button
          v-for="item in menu"
          :key="item.name"
          class="menu-btn"
          @click="currentView = item.component"
        >
          {{ item.name }}
        </button>
      </aside>

      <!-- Content View -->
      <section class="flex-1 p-6 flex items-center justify-center border">
        <component :is="currentView" />
      </section>
    </div>

    <!-- Footer -->
    <footer class="bg-green-600 text-white p-4 text-center font-bold">
      footer
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Import semua komponen
import Dashboard from '~/components/Dashboard.vue'
import InputData from '~/components/Input_data.vue'
import DetailData from '~/components/Detail_data.vue'
import UploadFile from '~/components/Upload_file.vue'
import UserLevel from '~/components/User_level.vue'
import Logout from '~/components/Logout.vue'

// Daftar menu dan komponen
const menu = [
  { name: 'dashboard', component: Dashboard },
  { name: 'input data', component: InputData },
  { name: 'Pegawai', component: DetailData },
  { name: 'Penduduk', component: UploadFile },
  { name: 'user level', component: UserLevel },
  { name: 'logout', component: Logout }
]

// Tampilan default
const currentView = ref(Dashboard)
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.menu-btn {
  background-color: #059669;
  padding: 0.5rem;
  border-radius: 0.25rem;
  text-align: left;
  cursor: pointer;
}

.menu-btn:hover {
  background-color: #047857;
}
</style>
