<template>
  <div class="w-full">
    <h2 class="text-xl font-bold mb-4">Form Input Data</h2>
    <form class="space-y-4">
      <div>
        <label class="block font-medium">Nama</label>
        <input type="text" class="border p-2 w-full rounded" placeholder="Masukkan nama" />
      </div>

      <div>
        <label class="block font-medium">Email</label>
        <input type="email" class="border p-2 w-full rounded" placeholder="Masukkan email" />
      </div>

      <div>
        <label class="block font-medium">Tanggal</label>
        <input type="date" class="border p-2 w-full rounded" />
      </div>

      <button type="submit" class="bg-green-600 text-white px-4 py-2 rounded">
        Simpan
      </button>
    </form>
  </div>
</template>

<script setup>
// Disini nanti kita bisa tambahkan logic untuk mengirim data ke backend API
</script>
