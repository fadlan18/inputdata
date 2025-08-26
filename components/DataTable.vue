<template>
  <div class="data-table">
    <!-- Search Bar -->
    <div class="mb-4 flex flex-col sm:flex-row gap-4 items-center justify-between">
      <div class="flex-1 max-w-md">
        <div class="relative">
          <input
            v-model="searchTerm"
            type="text"
            placeholder="Cari berdasarkan nama, NIP, NIK, jabatan, atau unit kerja..."
            class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            @input="onSearch"
          />
          <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
            <svg class="h-5 w-5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
            </svg>
          </div>
        </div>
      </div>
      
      <!-- Info dan Clear Search -->
      <div class="flex items-center gap-3 text-sm text-gray-600">
        <span>
          Menampilkan {{ paginatedData.length }} dari {{ filteredData.length }} data
          <span v-if="searchTerm" class="text-blue-600 font-medium">
            (filtered)
          </span>
        </span>
        <button
          v-if="searchTerm"
          @click="clearSearch"
          class="text-red-500 hover:text-red-700 underline"
        >
          Clear
        </button>
      </div>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto border rounded-lg shadow bg-white">
      <div v-if="loading" class="text-center py-8 text-gray-500">
        🔄 Memuat data...
      </div>
      
      <div v-else-if="error" class="text-center py-8 text-red-600">
        {{ error }}
      </div>

      <div v-else-if="filteredData.length === 0 && searchTerm" class="text-center py-8 text-gray-500">
        <svg class="mx-auto h-12 w-12 text-gray-400 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.172 16.172a4 4 0 015.656 0M9 12h6m-6-4h6m2 5.291A7.962 7.962 0 0112 15c-4.418 0-8-3.582-8-8s3.582-8 8-8 8 3.582 8 8a7.962 7.962 0 01-2.709 5.291z" />
        </svg>
        <p>Tidak ada data yang ditemukan untuk: <strong>"{{ searchTerm }}"</strong></p>
        <button @click="clearSearch" class="mt-2 text-blue-500 hover:text-blue-700 underline">
          Tampilkan semua data
        </button>
      </div>

      <table v-else class="min-w-full border-collapse text-sm">
        <thead>
          <tr class="bg-blue-50 text-gray-700">
            <th class="px-4 py-3 text-left font-medium">ID</th>
            <th class="px-4 py-3 text-left font-medium">Nama</th>
            <th class="px-4 py-3 text-left font-medium">NIP</th>
            <th class="px-4 py-3 text-left font-medium">NIK</th>
            <th class="px-4 py-3 text-left font-medium">Jabatan</th>
            <th class="px-4 py-3 text-left font-medium">Unit Kerja</th>
            <th class="px-4 py-3 text-left font-medium">Status</th>
            <th class="px-4 py-3 text-left font-medium">Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="row in paginatedData"
            :key="row.id"
            class="odd:bg-white even:bg-gray-50 hover:bg-blue-100 transition-colors"
          >
            <td class="px-4 py-3 text-gray-700">{{ row.id }}</td>
            <td class="px-4 py-3 text-gray-700">
              <span v-html="highlightText(row.nama)"></span>
            </td>
            <td class="px-4 py-3 text-gray-700">
              <span v-html="highlightText(row.nip?.toString())"></span>
            </td>
            <td class="px-4 py-3 text-gray-700">
              <span v-html="highlightText(row.nik?.toString())"></span>
            </td>
            <td class="px-4 py-3 text-gray-700">
              <span v-html="highlightText(row.jabatan)"></span>
            </td>
            <td class="px-4 py-3 text-gray-700">
              <span v-html="highlightText(row.unitkerja)"></span>
            </td>
            <td class="px-4 py-3 text-gray-700">
              <span 
                :class="[
                  'px-2 py-1 rounded-full text-xs font-medium',
                  row.aktif === 'aktif' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'
                ]"
              >
                {{ row.aktif === 'aktif' ? 'Aktif' : 'Non-Aktif' }}
              </span>
            </td>
            <td class="px-4 py-3 flex gap-2">
              <button
                class="px-3 py-1 bg-yellow-400 text-white rounded hover:bg-yellow-500 text-sm font-medium transition-colors"
                @click="$emit('edit', row)"
              >
                Edit
              </button>
              <button
                class="px-3 py-1 bg-red-500 text-white rounded hover:bg-red-600 text-sm font-medium transition-colors"
                @click="$emit('delete', row.id)"
              >
                Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div v-if="totalPages > 1" class="mt-6 flex flex-col sm:flex-row items-center justify-between gap-4">
      <!-- Page Info -->
      <div class="text-sm text-gray-600">
        Halaman {{ currentPage }} dari {{ totalPages }}
        ({{ startIndex + 1 }}-{{ endIndex }} dari {{ filteredData.length }} data)
      </div>

      <!-- Pagination Controls -->
      <div class="flex items-center gap-2">
        <!-- First Page -->
        <button
          :disabled="currentPage === 1"
          @click="goToPage(1)"
          class="px-3 py-2 text-sm bg-white border border-gray-300 rounded-lg disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-50"
        >
          ⟪
        </button>

        <!-- Previous Page -->
        <button
          :disabled="currentPage === 1"
          @click="goToPage(currentPage - 1)"
          class="px-3 py-2 text-sm bg-white border border-gray-300 rounded-lg disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-50"
        >
          ⟨ Prev
        </button>

        <!-- Page Numbers -->
        <button
          v-for="page in visiblePages"
          :key="page"
          @click="goToPage(page)"
          :class="[
            'px-3 py-2 text-sm border rounded-lg',
            page === currentPage
              ? 'bg-blue-500 text-white border-blue-500'
              : 'bg-white border-gray-300 hover:bg-gray-50'
          ]"
        >
          {{ page }}
        </button>

        <!-- Next Page -->
        <button
          :disabled="currentPage === totalPages"
          @click="goToPage(currentPage + 1)"
          class="px-3 py-2 text-sm bg-white border border-gray-300 rounded-lg disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-50"
        >
          Next ⟩
        </button>

        <!-- Last Page -->
        <button
          :disabled="currentPage === totalPages"
          @click="goToPage(totalPages)"
          class="px-3 py-2 text-sm bg-white border border-gray-300 rounded-lg disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-50"
        >
          ⟫
        </button>
      </div>

      <!-- Items Per Page -->
      <div class="flex items-center gap-2 text-sm">
        <label for="itemsPerPage" class="text-gray-600">Per halaman:</label>
        <select
          id="itemsPerPage"
          v-model="itemsPerPage"
          @change="onItemsPerPageChange"
          class="border border-gray-300 rounded px-2 py-1 text-sm"
        >
          <option value="5">5</option>
          <option value="10">10</option>
          <option value="25">25</option>
          <option value="50">50</option>
        </select>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "DataTable",
  
  props: {
    data: {
      type: Array,
      default: () => []
    },
    loading: {
      type: Boolean,
      default: false
    },
    error: {
      type: String,
      default: ""
    }
  },

  emits: ["edit", "delete"],

  data() {
    return {
      searchTerm: "",
      currentPage: 1,
      itemsPerPage: 10,
      searchTimeout: null
    };
  },

  computed: {
    filteredData() {
      if (!this.searchTerm.trim()) return this.data;

      const term = this.searchTerm.toLowerCase().trim();
      return this.data.filter(row => {
        return (
          (row.nama || "").toLowerCase().includes(term) ||
          (row.nip || "").toString().toLowerCase().includes(term) ||
          (row.nik || "").toString().toLowerCase().includes(term) ||
          (row.jabatan || "").toLowerCase().includes(term) ||
          (row.unitkerja || "").toLowerCase().includes(term) ||
          (row.aktif || "").toLowerCase().includes(term)
        );
      });
    },

    totalPages() {
      return Math.ceil(this.filteredData.length / this.itemsPerPage);
    },

    startIndex() {
      return (this.currentPage - 1) * this.itemsPerPage;
    },

    endIndex() {
      return Math.min(this.startIndex + this.itemsPerPage, this.filteredData.length);
    },

    paginatedData() {
      return this.filteredData.slice(this.startIndex, this.endIndex);
    },

    visiblePages() {
      const pages = [];
      const maxVisible = 5;
      let start = Math.max(1, this.currentPage - Math.floor(maxVisible / 2));
      let end = Math.min(this.totalPages, start + maxVisible - 1);

      if (end - start + 1 < maxVisible) {
        start = Math.max(1, end - maxVisible + 1);
      }

      for (let i = start; i <= end; i++) {
        pages.push(i);
      }

      return pages;
    }
  },

  watch: {
    // Reset ke halaman 1 ketika data berubah atau search
    filteredData() {
      this.currentPage = 1;
    }
  },

  methods: {
    onSearch() {
      // Debounce search untuk performa yang lebih baik
      clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(() => {
        this.currentPage = 1;
      }, 300);
    },

    clearSearch() {
      this.searchTerm = "";
      this.currentPage = 1;
    },

    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },

    onItemsPerPageChange() {
      this.currentPage = 1;
    },

    highlightText(text) {
      if (!this.searchTerm.trim() || !text) return text;
      
      const term = this.searchTerm.trim();
      const regex = new RegExp(`(${term})`, 'gi');
      return text.replace(regex, '<mark class="bg-yellow-200 px-1 rounded">$1</mark>');
    }
  }
};
</script>