<template>
  <div
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
    @click.self="closeModal"
  >
    <div class="bg-white rounded-lg shadow-lg w-full max-w-lg max-h-[90vh] overflow-y-auto p-6">
      <h2 class="text-xl font-semibold mb-4">Edit Data Pegawai</h2>

      <form @submit.prevent="submitForm" class="space-y-4">
        <div>
          <label class="block font-medium">Nama</label>
          <input
            v-model="form.nama"
            type="text"
            class="w-full border rounded-lg px-3 py-2 focus:outline-none focus:ring"
            required
          />
        </div>

        <div>
          <label class="block font-medium">NIP</label>
          <input
            v-model="form.nip"
            type="number"
            class="w-full border rounded-lg px-3 py-2 focus:outline-none focus:ring"
            required
          />
        </div>

        <div>
          <label class="block font-medium">NIK</label>
          <input
            v-model="form.nik"
            type="number"
            class="w-full border rounded-lg px-3 py-2 focus:outline-none focus:ring"
          />
        </div>

        <div>
          <label class="block font-medium">Jabatan</label>
          <input
            v-model="form.jabatan"
            type="text"
            class="w-full border rounded-lg px-3 py-2 focus:outline-none focus:ring"
          />
        </div>

        <div>
          <label class="block font-medium">Unit Kerja</label>
          <input
            v-model="form.unitkerja"
            type="text"
            class="w-full border rounded-lg px-3 py-2 focus:outline-none focus:ring"
          />
        </div>

        <div>
          <label class="block font-medium">Status</label>
          <select
            v-model="form.aktif"
            class="w-full border rounded-lg px-3 py-2 focus:outline-none focus:ring"
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
</template>

<script>
export default {
  name: "EditInputData",

  props: {
    data: {
      type: Object,
      default: () => null
    }
  },

  emits: ["close", "saved"],

  data() {
    return {
      form: {
        nama: "",
        nip: "",
        nik: "",
        jabatan: "",
        unitkerja: "",
        aktif: "aktif"
      },
      saving: false
    };
  },

  watch: {
    data: {
      immediate: true,
      handler(newData) {
        if (newData && newData.id) {
          // Isi form dengan data yang akan diedit
          this.form = {
            nama: newData.nama || "",
            nip: newData.nip || "",
            nik: newData.nik || "",
            jabatan: newData.jabatan || "",
            unitkerja: newData.unitkerja || "",
            aktif: newData.aktif || "aktif"
          };
        } else {
          // Reset form
          this.form = {
            nama: "",
            nip: "",
            nik: "",
            jabatan: "",
            unitkerja: "",
            aktif: "aktif"
          };
        }
      }
    }
  },

  methods: {
    closeModal() {
      this.$emit("close");
    },

    async submitForm() {
      if (!this.form.nama || !this.form.nip) {
        alert("Nama dan NIP wajib diisi.");
        return;
      }

      this.saving = true;

      try {
        const token = localStorage.getItem("jwt") || "";

        // Siapkan payload sesuai dengan struktur API pegawai
        const payload = {
          nama: this.form.nama || "",
          nip: parseInt(this.form.nip) || 0,
          nik: parseInt(this.form.nik) || 0,
          jabatan: this.form.jabatan || "",
          unitkerja: this.form.unitkerja || "",
          aktif: this.form.aktif || "aktif"
        };

        console.log("=== DEBUG INFO ===");
        console.log("Payload yang akan dikirim:", payload);
        console.log("Data props:", this.data);
        console.log("Data ID:", this.data?.id);
        console.log("Data documentId:", this.data?.documentId);

        // Strapi v5 menggunakan documentId untuk update
        const strapiDocumentId = this.data.documentId;
        
        if (!strapiDocumentId) {
          throw new Error("DocumentId tidak ditemukan. Tidak dapat melakukan update.");
        }

        const url = `https://api008.tojounauna.go.id/api/pegawai-app001s/${strapiDocumentId}`;

        console.log("Request URL:", url);
        console.log("Request payload:", JSON.stringify({ data: payload }));

        const res = await fetch(url, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
            Authorization: token ? `Bearer ${token}` : ""
          },
          body: JSON.stringify({ data: payload })
        });

        console.log("Response status:", res.status);
        console.log("Response ok:", res.ok);

        if (!res.ok) {
          let errorData = {};
          try { 
            errorData = await res.json();
            console.log("Error response:", errorData);
          } catch (e) {
            console.log("Tidak dapat parse error response:", e);
          }
          
          throw new Error(errorData?.error?.message || `HTTP ${res.status}: ${res.statusText}`);
        }

        const result = await res.json();
        console.log("Success response:", result);

        alert("✅ Data berhasil diupdate!");
        this.$emit("saved");

      } catch (err) {
        console.error("=== ERROR DETAILS ===");
        console.error("Error message:", err.message);
        console.error("Full error object:", err);
        
        alert(`❌ Gagal menyimpan data: ${err.message}`);
      } finally {
        this.saving = false;
      }
    }
  }
};
</script>