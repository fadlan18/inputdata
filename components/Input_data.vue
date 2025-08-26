<script>
import EditInputData from "./EditInputData.vue";
import DataTable from "./DataTable.vue";

export default {
  name: "InputData",

  components: {
    EditInputData,
    DataTable,
  },

  data() {
    return {
      // Modal untuk input baru
      showModal: false,
      // Modal untuk edit data
      showEditModal: false,
      selectedRow: null,

      saving: false,
      opdList: [],
      rows: [],
      tableLoading: false,
      tableError: "",
      formData: {
        nama: "",
        nip: "",
        nik: "",
        jabatan: "",
        unitkerja: "",
        aktif: "aktif"
      }
    };
  },

  mounted() {
    this.fetchTable();
  },

  methods: {
    openModal() {
      this.showModal = true;
      this.fetchOPD();
    },

    closeModal() {
      this.showModal = false;
    },

    closeOnBackdrop(e) {
      if (e.target === e.currentTarget) this.closeModal();
    },

    async fetchOPD() {
      try {
        const token = localStorage.getItem("jwt") || "";
        const res = await fetch("https://api008.tojounauna.go.id/api/pegawai-app001s", {
          headers: { Authorization: token ? `Bearer ${token}` : "" }
        });
        const data = await res.json();
        this.opdList = Array.isArray(data?.data) ? data.data : [];
      } catch (err) {
        console.error("Gagal memuat data pegawai:", err);
        this.opdList = [];
      }
    },

    async fetchTable() {
      this.tableLoading = true;
      this.tableError = "";
      try {
        const res = await fetch("https://api008.tojounauna.go.id/api/pegawai-app001s");
        const data = await res.json();
        this.rows = Array.isArray(data?.data) ? data.data : [];
      } catch (err) {
        console.error("Gagal memuat tabel:", err);
        this.tableError = err?.message || "Gagal memuat data.";
      } finally {
        this.tableLoading = false;
      }
    },

    async submitForm() {
      if (!this.formData.nama || !this.formData.nip) {
        alert("Nama dan NIP wajib diisi.");
        return;
      }

      this.saving = true;
      try {
        const token = localStorage.getItem("jwt") || "";

        const payload = {
          nama: this.formData.nama || "",
          nip: parseInt(this.formData.nip) || 0,
          nik: parseInt(this.formData.nik) || 0,
          jabatan: this.formData.jabatan || "",
          unitkerja: this.formData.unitkerja || "",
          aktif: this.formData.aktif || "aktif",
        };

        const res = await fetch("https://api008.tojounauna.go.id/api/pegawai-app001s", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            Authorization: token ? `Bearer ${token}` : ""
          },
          body: JSON.stringify({ data: payload })
        });

        if (!res.ok) {
          let result = null;
          try { result = await res.json(); } catch {}
          throw new Error(result?.error?.message || "Gagal menyimpan data");
        }

        alert("✅ Data berhasil disimpan.");
        this.resetForm();
        this.closeModal();
        await this.fetchTable();
      } catch (err) {
        console.error("Gagal menyimpan data:", err?.message || err);
        alert("❌ Gagal menyimpan data. Periksa console untuk detail.");
      } finally {
        this.saving = false;
      }
    },

    resetForm() {
      this.formData = {
        nama: "",
        nip: "",
        nik: "",
        jabatan: "",
        unitkerja: "",
        aktif: "aktif"
      };
    },

    // === Edit ===
    openEdit(row) {
      console.log("=== DEBUGGING EDIT ===");
      console.log("Original row data:", row);
      console.log("Row ID:", row.id);
      console.log("Row documentId:", row.documentId);
      console.log("Row type:", typeof row.id);
      
      // Buat deep copy yang benar-benar plain object
      // Pastikan documentId juga disertakan untuk Strapi v5
      this.selectedRow = {
        id: row.id,
        documentId: row.documentId, // Penting untuk Strapi v5
        nama: row.nama || "",
        nip: row.nip || "",
        nik: row.nik || "",
        jabatan: row.jabatan || "",
        unitkerja: row.unitkerja || "",
        aktif: row.aktif || "aktif"
      };
      
      console.log("Selected row after copy:", this.selectedRow);
      console.log("Selected row ID:", this.selectedRow.id);
      console.log("Selected row documentId:", this.selectedRow.documentId);
      
      this.showEditModal = true;
    },

    closeEditModal() {
      this.showEditModal = false;
      this.selectedRow = null;
    },

    async onEditSaved() {
      this.showEditModal = false;
      this.selectedRow = null;
      await this.fetchTable();
    },

    async deleteRow(id) {
      if (!confirm("Apakah yakin ingin menghapus data ini?")) return;

      try {
        const token = localStorage.getItem("jwt") || "";
        
        // Cari data untuk mendapatkan documentId
        const rowToDelete = this.rows.find(row => row.id === id);
        const documentId = rowToDelete?.documentId || id;
        
        console.log("Delete - using documentId:", documentId);
        
        const res = await fetch(
          `https://api008.tojounauna.go.id/api/pegawai-app001s/${documentId}`,
          {
            method: "DELETE",
            headers: { Authorization: token ? `Bearer ${token}` : "" }
          }
        );

        if (!res.ok) throw new Error(`Gagal hapus data (HTTP ${res.status})`);

        // Hapus row di frontend langsung
        this.rows = this.rows.filter(row => row.id !== id);

        alert("✅ Data berhasil dihapus");
      } catch (err) {
        console.error(err);
        alert("❌ Gagal menghapus data. Periksa console.");
      }
    }
  }
};
</script>

<template>
  <div class="p-6">
    <!-- Tombol tambah data -->
    <button
      class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
      @click="openModal"
    >
      + Tambah Data
    </button>

    <!-- Modal Input Data -->
    <div
      v-if="showModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click="closeOnBackdrop"
    >
      <div
        class="bg-white rounded-lg shadow-lg w-full max-w-lg max-h-[90vh] overflow-y-auto p-6"
      >
        <h2 class="text-xl font-semibold mb-4">Form Input Data Pegawai</h2>

        <form @submit.prevent="submitForm" class="space-y-3">
          <div>
            <label class="block text-sm font-medium">Nama</label>
            <input
              v-model="formData.nama"
              type="text"
              class="w-full border rounded-lg px-3 py-2"
              required
            />
          </div>

          <div>
            <label class="block text-sm font-medium">NIP</label>
            <input
              v-model="formData.nip"
              type="number"
              class="w-full border rounded-lg px-3 py-2"
              required
            />
          </div>

          <div>
            <label class="block text-sm font-medium">NIK</label>
            <input
              v-model="formData.nik"
              type="number"
              class="w-full border rounded-lg px-3 py-2"
            />
          </div>

          <div>
            <label class="block text-sm font-medium">Jabatan</label>
            <input
              v-model="formData.jabatan"
              type="text"
              class="w-full border rounded-lg px-3 py-2"
            />
          </div>

          <div>
            <label class="block text-sm font-medium">Unit Kerja</label>
            <input
              v-model="formData.unitkerja"
              type="text"
              class="w-full border rounded-lg px-3 py-2"
            />
          </div>

          <div>
            <label class="block text-sm font-medium">Status</label>
            <select
              v-model="formData.aktif"
              class="w-full border rounded-lg px-3 py-2"
            >
              <option value="aktif">Aktif</option>
              <option value="nonaktif">Non-Aktif</option>
            </select>
          </div>

          <div class="flex justify-end gap-3 mt-4">
            <button
              type="button"
              @click="closeModal"
              class="px-4 py-2 bg-gray-400 text-white rounded-lg hover:bg-gray-500"
            >
              Batal
            </button>
            <button
              type="submit"
              :disabled="saving"
              class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700"
            >
              {{ saving ? "Menyimpan..." : "Simpan" }}
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal Edit Data -->
    <EditInputData
      v-if="showEditModal"
      :data="selectedRow"
      @close="closeEditModal"
      @saved="onEditSaved"
    />

    <!-- Data Table dengan Search & Pagination -->
    <div class="mt-6">
      <h3 class="text-lg font-semibold mb-4 text-gray-700">Daftar Data Pegawai</h3>
      
      <DataTable
        :data="rows"
        :loading="tableLoading"
        :error="tableError"
        @edit="openEdit"
        @delete="deleteRow"
      />
    </div>
  </div>
</template>