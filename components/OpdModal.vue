<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
    <div class="bg-white rounded-lg p-6 w-96">
      <h2 class="text-xl font-bold mb-4">
        {{ editData ? "Edit OPD" : "Tambah OPD" }}
      </h2>

      <div class="mb-4">
        <label class="block text-sm font-medium">Nama OPD</label>
        <input
          v-model="form.nama"
          type="text"
          class="w-full border p-2 rounded"
          placeholder="Masukkan nama OPD"
        />
      </div>

      <div class="flex justify-end gap-2">
        <button @click="$emit('close')" class="bg-gray-400 hover:bg-gray-500 text-white px-4 py-2 rounded">
          Batal
        </button>
        <button @click="saveData" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded">
          Simpan
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, watch } from "vue";

const props = defineProps({
  editData: {
    type: Object,
    default: null
  }
});

const emit = defineEmits(["close", "saved"]);

const form = reactive({
  nama: ""
});

// Set data saat modal dibuka untuk edit
watch(
  () => props.editData,
  (val) => {
    if (val) {
      form.nama = val.nama;
    } else {
      form.nama = "";
    }
  },
  { immediate: true }
);

const saveData = async () => {
  // Validasi sederhana
  if (!form.nama.trim()) {
    alert("Nama OPD wajib diisi!");
    return;
  }

  try {
    if (props.editData) {
      // Update
      await $fetch(`http://localhost:1337/api/opd-app001s/${props.editData.id}`, {
        method: "PUT",
        body: { data: { nama: form.nama } }
      });
      alert("Data berhasil diperbarui!");
    } else {
      // Tambah baru
      await $fetch("http://localhost:1337/api/opd-app001s", {
        method: "POST",
        body: { data: { nama: form.nama } }
      });
      alert("Data berhasil disimpan!");
    }
    emit("saved");
    emit("close");
  } catch (err) {
    console.error("Gagal simpan data:", err);
    alert("Gagal menyimpan data!");
  }
};
</script>
