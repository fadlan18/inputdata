<script setup lang="ts">
import { ref, onMounted } from "vue";
import { GraphQLClient, gql } from "graphql-request";

// endpoint GraphQL Hasura
const endpoint = "https://api801.tojounauna.go.id/v1/graphql";
const client = new GraphQLClient(endpoint, {
  headers: {
    "x-hasura-admin-secret": "hsrSecret123", 
  },
});

// state
const rows = ref<any[]>([]);
const loading = ref(false);
const error = ref("");
const page = ref(1);
const limit = 10;
const total = ref(0);
const keyword = ref("");
const searchFocused = ref(false);

// query GraphQL untuk pegawai berdasarkan NIP atau Nama
const GET_PEGAWAI = gql`
  query GetPegawai($limit: Int!, $offset: Int!, $keyword: String) {
    pegawai_users_aggregate(
      where: {
        _or: [
          { nip: { _ilike: $keyword } }
          { name: { _ilike: $keyword } }
        ]
      }
    ) {
      aggregate {
        count
      }
    }
    pegawai_users(
      limit: $limit
      offset: $offset
      order_by: { name: asc }
      where: {
        _or: [
          { nip: { _ilike: $keyword } }
          { name: { _ilike: $keyword } }
        ]
      }
    ) {
      nip
      name
      tempat_lahir
      kode_agama
      email
      no_hp
      norek
    }
  }
`;

async function fetchPegawai() {
  loading.value = true;
  error.value = "";
  try {
    const offset = (page.value - 1) * limit;
    const variables = {
      limit,
      offset,
      keyword: `%${keyword.value}%`,
    };

    const data = await client.request(GET_PEGAWAI, variables);
    rows.value = data.pegawai_users;
    total.value = data.pegawai_users_aggregate.aggregate.count;
  } catch (err: any) {
    error.value = err.message || "Terjadi kesalahan";
  } finally {
    loading.value = false;
  }
}

function nextPage() {
  if (page.value * limit < total.value) {
    page.value++;
    fetchPegawai();
  }
}

function prevPage() {
  if (page.value > 1) {
    page.value--;
    fetchPegawai();
  }
}

function searchData() {
  page.value = 1;
  fetchPegawai();
}

function handleKeyPress(event: KeyboardEvent) {
  if (event.key === 'Enter') {
    searchData();
  }
}

function getReligionName(code: string) {
  const religions: { [key: string]: string } = {
    '1': 'Islam',
    '2': 'Kristen',
    '3': 'Katholik',
    '4': 'Hindu',
    '5': 'Buddha',
    '6': 'Khonghucu'
  };
  return religions[code] || code;
}

onMounted(fetchPegawai);
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-100">
    <div class="container mx-auto px-4 py-8">
      <!-- Header Section -->
      <div class="mb-8">
        <div class="flex items-center gap-3 mb-2">
          <div class="w-10 h-10 bg-gradient-to-r from-blue-600 to-indigo-600 rounded-xl flex items-center justify-center">
            <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z"></path>
            </svg>
          </div>
          <div>
            <h1 class="text-3xl font-bold bg-gradient-to-r from-slate-800 to-slate-600 bg-clip-text text-transparent">
              Daftar Pegawai
            </h1>
            <p class="text-slate-600 text-sm mt-1">Kelola data pegawai dengan mudah dan efisien</p>
          </div>
        </div>
      </div>

      <!-- Search Section -->
      <div class="bg-white/70 backdrop-blur-sm rounded-2xl p-6 mb-6 border border-white/20 shadow-xl">
        <div class="flex flex-col sm:flex-row gap-4 items-end">
          <div class="flex-1 relative">
            <label class="block text-sm font-semibold text-slate-700 mb-2">Pencarian Pegawai</label>
            <div class="relative">
              <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
                </svg>
              </div>
              <input
                v-model="keyword"
                @focus="searchFocused = true"
                @blur="searchFocused = false"
                @keypress="handleKeyPress"
                type="text"
                placeholder="Cari berdasarkan NIP atau Nama pegawai..."
                :class="[
                  'w-full pl-12 pr-4 py-3 rounded-xl border-2 transition-all duration-200 bg-white/50 backdrop-blur-sm',
                  searchFocused 
                    ? 'border-blue-500 ring-4 ring-blue-500/10 shadow-lg' 
                    : 'border-slate-200 hover:border-slate-300 shadow-sm'
                ]"
              />
            </div>
          </div>
          <button
            @click="searchData"
            :disabled="loading"
            class="px-8 py-3 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-700 hover:to-indigo-700 text-white font-semibold rounded-xl transition-all duration-200 shadow-lg hover:shadow-xl transform hover:-translate-y-0.5 disabled:opacity-50 disabled:cursor-not-allowed disabled:transform-none flex items-center gap-2"
          >
            <svg v-if="!loading" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
            </svg>
            <svg v-else class="w-5 h-5 animate-spin" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path>
            </svg>
            {{ loading ? 'Mencari...' : 'Cari' }}
          </button>
        </div>
      </div>

      <!-- Stats Card -->
      <div v-if="!loading" class="bg-white/70 backdrop-blur-sm rounded-2xl p-6 mb-6 border border-white/20 shadow-xl">
        <div class="flex items-center justify-between">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-gradient-to-r from-emerald-500 to-teal-500 rounded-xl flex items-center justify-center">
              <svg class="w-7 h-7 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v4a2 2 0 01-2 2h-2a2 2 0 00-2-2z"></path>
              </svg>
            </div>
            <div>
              <h3 class="text-lg font-semibold text-slate-800">Total Pegawai</h3>
              <p class="text-3xl font-bold bg-gradient-to-r from-emerald-600 to-teal-600 bg-clip-text text-transparent">
                {{ total.toLocaleString() }}
              </p>
            </div>
          </div>
          <div class="text-right text-sm text-slate-600">
            <p>Menampilkan {{ rows.length }} dari {{ total }} data</p>
            <p class="text-xs">Halaman {{ page }} dari {{ Math.ceil(total / limit) }}</p>
          </div>
        </div>
      </div>

      <!-- Error Alert -->
      <div v-if="error" class="bg-red-50 border-l-4 border-red-400 p-6 rounded-xl mb-6 shadow-lg">
        <div class="flex items-center">
          <div class="flex-shrink-0">
            <svg class="h-6 w-6 text-red-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
            </svg>
          </div>
          <div class="ml-3">
            <h3 class="text-sm font-medium text-red-800">Terjadi Kesalahan</h3>
            <p class="text-sm text-red-700 mt-1">{{ error }}</p>
          </div>
        </div>
      </div>

      <!-- Loading State -->
      <div v-if="loading" class="bg-white/70 backdrop-blur-sm rounded-2xl p-12 border border-white/20 shadow-xl">
        <div class="flex flex-col items-center justify-center">
          <div class="w-16 h-16 border-4 border-blue-200 border-t-blue-600 rounded-full animate-spin mb-4"></div>
          <p class="text-slate-600 font-medium">Memuat data pegawai...</p>
        </div>
      </div>

      <!-- Data Table -->
      <div v-else-if="rows.length > 0" class="bg-white/70 backdrop-blur-sm rounded-2xl overflow-hidden border border-white/20 shadow-xl">
        <div class="overflow-x-auto">
          <table class="w-full">
            <thead class="bg-gradient-to-r from-slate-800 to-slate-700 text-white">
              <tr>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">No</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">NIP</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">Nama</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">Tempat Lahir</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">Agama</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">Email</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">No HP</th>
                <th class="px-6 py-4 text-left text-sm font-semibold uppercase tracking-wider">No Rekening</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-200">
              <tr
                v-for="(pegawai, index) in rows"
                :key="pegawai.nip"
                class="hover:bg-blue-50/50 transition-colors duration-150"
              >
                <td class="px-6 py-4 whitespace-nowrap text-sm text-slate-800 font-medium">
                  <div class="w-8 h-8 bg-gradient-to-r from-blue-500 to-indigo-500 rounded-lg flex items-center justify-center text-white text-xs font-bold">
                    {{ (page - 1) * limit + index + 1 }}
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="text-sm font-mono bg-slate-100 px-3 py-1 rounded-lg text-slate-800">
                    {{ pegawai.nip }}
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="flex items-center">
                    <div class="w-10 h-10 bg-gradient-to-r from-purple-500 to-pink-500 rounded-full flex items-center justify-center text-white font-bold text-sm mr-3">
                      {{ pegawai.name.charAt(0).toUpperCase() }}
                    </div>
                    <div>
                      <div class="text-sm font-semibold text-slate-900">{{ pegawai.name }}</div>
                    </div>
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-slate-700">
                  <div class="flex items-center">
                    <svg class="w-4 h-4 text-slate-400 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"></path>
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path>
                    </svg>
                    {{ pegawai.tempat_lahir }}
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-slate-700">
                  <span class="inline-flex items-center px-3 py-1 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800">
                    {{ getReligionName(pegawai.kode_agama) }}
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-slate-700">
                  <a :href="`mailto:${pegawai.email}`" class="flex items-center hover:text-blue-600 transition-colors">
                    <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 4.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path>
                    </svg>
                    {{ pegawai.email }}
                  </a>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-slate-700">
                  <a :href="`tel:${pegawai.no_hp}`" class="flex items-center hover:text-blue-600 transition-colors">
                    <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z"></path>
                    </svg>
                    {{ pegawai.no_hp }}
                  </a>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm font-mono text-slate-700">
                  <div class="bg-slate-100 px-3 py-1 rounded-lg">
                    {{ pegawai.norek }}
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Pagination -->
        <div class="bg-white/50 px-6 py-4 border-t border-slate-200">
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-4">
              <p class="text-sm text-slate-700">
                Menampilkan <span class="font-semibold">{{ (page - 1) * limit + 1 }}</span> sampai 
                <span class="font-semibold">{{ Math.min(page * limit, total) }}</span> dari 
                <span class="font-semibold">{{ total }}</span> hasil
              </p>
            </div>
            <div class="flex items-center gap-2">
              <button
                @click="prevPage"
                :disabled="page === 1"
                class="px-4 py-2 text-sm font-medium text-slate-700 bg-white border border-slate-300 rounded-lg hover:bg-slate-50 disabled:opacity-50 disabled:cursor-not-allowed transition-colors flex items-center gap-2"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
                </svg>
                Sebelumnya
              </button>
              <span class="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg">
                {{ page }} / {{ Math.ceil(total / limit) }}
              </span>
              <button
                @click="nextPage"
                :disabled="page * limit >= total"
                class="px-4 py-2 text-sm font-medium text-slate-700 bg-white border border-slate-300 rounded-lg hover:bg-slate-50 disabled:opacity-50 disabled:cursor-not-allowed transition-colors flex items-center gap-2"
              >
                Selanjutnya
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-else-if="!loading" class="bg-white/70 backdrop-blur-sm rounded-2xl p-12 border border-white/20 shadow-xl text-center">
        <div class="w-24 h-24 bg-slate-100 rounded-full flex items-center justify-center mx-auto mb-6">
          <svg class="w-12 h-12 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z"></path>
          </svg>
        </div>
        <h3 class="text-lg font-semibold text-slate-800 mb-2">Tidak Ada Data Pegawai</h3>
        <p class="text-slate-600 mb-4">
          {{ keyword ? 'Tidak ditemukan pegawai yang sesuai dengan pencarian.' : 'Belum ada data pegawai yang tersedia.' }}
        </p>
        <button
          v-if="keyword"
          @click="keyword = ''; searchData()"
          class="px-6 py-3 bg-blue-600 hover:bg-blue-700 text-white font-medium rounded-lg transition-colors"
        >
          Tampilkan Semua Data
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Additional smooth animations */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* Custom scrollbar for table */
.overflow-x-auto::-webkit-scrollbar {
  height: 8px;
}
.overflow-x-auto::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}
.overflow-x-auto::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}
.overflow-x-auto::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}
</style>