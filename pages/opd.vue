<template>
  <div class="p-6">
    <h1 class="text-2xl font-bold mb-4">Data OPD</h1>

    <!-- Tombol Tambah Data -->
    <button
      @click="openModal"
      class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded"
    >
      Tambah Data
    </button>

    <!-- Tabel Data -->
    <table class="mt-4 w-full border">
      <thead class="bg-gray-100">
        <tr>
          <th class="border p-2">ID</th>
          <th class="border p-2">Nama OPD</th>
          <th class="border p-2">Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="opd in opdList" :key="opd.id">
          <td class="border p-2">{{ opd.id }}</td>
          <td class="border p-2">{{ opd.attributes?.nama || '-' }}</td>
          <td class="border p-2">
            <button
              @click="editOpd(opd)"
              class="bg-yellow-500 hover:bg-yellow-600 text-white px-2 py-1 rounded"
            >
              Edit
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Modal Form -->
    <OpdModal
      v-if="showModal"
      :editData="editData"
      @close="closeModal"
      @saved="fetchData"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import OpdModal from "~/components/OpdModal.vue";

const opdList = ref([]);
const showModal = ref(false);
const editData = ref(null);

const fetchData = async () => {
  try {
    const res = await $fetch("http://localhost:1337/api/opd-app001s");
    opdList.value = res.data;
  } catch (err) {
    console.error("Gagal ambil data:", err);
  }
};

const openModal = () => {
  editData.value = null;
  showModal.value = true;
};

const editOpd = (opd) => {
  // Kirim plain object, tanpa fungsi
  editData.value = {
    id: opd.id,
    nama: opd.attributes?.nama || ""
  };
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
};

onMounted(fetchData);
</script>
