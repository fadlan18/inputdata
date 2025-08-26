<template>
  <div class="penduduk-container">
    <!-- Header dengan gradient background -->
    <div class="hero-section">
      <div class="hero-content">
        <h1 class="hero-title">
          <span class="emoji-icon">📋</span>
          Daftar Penduduk
        </h1>
        <p class="hero-subtitle">Sistem Informasi Kependudukan Digital</p>
      </div>
      <div class="hero-decoration">
        <div class="floating-circle circle-1"></div>
        <div class="floating-circle circle-2"></div>
        <div class="floating-circle circle-3"></div>
      </div>
    </div>

    <div class="main-content">
      <!-- Search Section -->
      <div class="search-section">
        <form @submit.prevent="cariPenduduk" class="search-form">
          <div class="search-input-group">
            <div class="search-icon">
              <svg width="20" height="20" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m21 21-6-6m2-5a7 7 0 1 1-14 0 7 7 0 0 1 14 0Z"/>
              </svg>
            </div>
            <input
              v-model="keyword"
              type="text"
              placeholder="🔍 Cari NIK, nama, atau alamat..."
              class="search-input"
            />
          </div>
          <button type="submit" class="search-btn btn-press">
            <span>Cari</span>
            <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5-5 5M6 12h12"/>
            </svg>
          </button>
        </form>
      </div>

      <!-- Stats Card -->
      <div class="stats-card" v-if="!loading && !error">
        <div class="stat-item card-entrance hover-lift">
          <div class="stat-icon">📊</div>
          <div class="stat-content">
            <div class="stat-number">{{ totalRows.toLocaleString('id-ID') }}</div>
            <div class="stat-label">Total Penduduk</div>
          </div>
        </div>
        <div class="stat-item card-entrance hover-lift">
          <div class="stat-icon">📄</div>
          <div class="stat-content">
            <div class="stat-number">{{ page }}</div>
            <div class="stat-label">Halaman Aktif</div>
          </div>
        </div>
        <div class="stat-item card-entrance hover-lift">
          <div class="stat-icon">🔍</div>
          <div class="stat-content">
            <div class="stat-number">{{ penduduk.length }}</div>
            <div class="stat-label">Data Ditampilkan</div>
          </div>
        </div>
      </div>

      <!-- Loading State -->
      <div v-if="loading" class="loading-state fade-in">
        <div class="loading-spinner">
          <div class="spinner"></div>
          <p class="loading-text">⏳ Loading data...</p>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="error-state fade-in">
        <div class="error-icon">❌</div>
        <h3 class="error-title">Terjadi Kesalahan</h3>
        <p class="error-message">{{ error }}</p>
        <button @click="fetchPenduduk" class="retry-btn btn-press hover-glow">
          <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 0 0 4.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 0 1-15.357-2m15.357 2H15"/>
          </svg>
          Coba Lagi
        </button>
      </div>

      <!-- Data Table -->
      <div v-else-if="penduduk.length" class="data-section fade-in">
        <div class="table-container">
          <div class="table-scroll">
            <table class="modern-table">
              <thead>
                <tr>
                  <th class="th-no">No</th>
                  <th class="th-nik">NIK</th>
                  <th class="th-nama">Nama</th>
                  <th class="th-ttl">TTL</th>
                  <th class="th-alamat">Alamat</th>
                  <th class="th-kecamatan">Kecamatan</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(p, i) in penduduk" :key="p.nik" class="table-row smooth-transition">
                  <td class="td-no">
                    <span class="number-badge">{{ (page - 1) * limit + i + 1 }}</span>
                  </td>
                  <td class="td-nik">
                    <div class="nik-container">
                      <span class="nik-icon">🆔</span>
                      <code class="nik-text" :title="p.nik">{{ p.nik }}</code>
                    </div>
                  </td>
                  <td class="td-nama">
                    <div class="nama-container">
                      <span class="avatar">{{ getInitials(p.nama) }}</span>
                      <span class="nama-text" :title="p.nama">{{ p.nama }}</span>
                    </div>
                  </td>
                  <td class="td-ttl">
                    <div class="ttl-container">
                      <span class="calendar-icon">📅</span>
                      <span class="ttl-text" :title="p.ttl">{{ p.ttl }}</span>
                    </div>
                  </td>
                  <td class="td-alamat">
                    <div class="alamat-container">
                      <span class="location-icon">📍</span>
                      <span class="alamat-text" :title="p.desa_kel">{{ p.desa_kel }}</span>
                    </div>
                  </td>
                  <td class="td-kecamatan">
                    <span class="kecamatan-badge" :title="p.kecamatan">{{ p.kecamatan }}</span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Enhanced Pagination -->
        <div class="pagination-wrapper" v-if="totalPages > 1">
          <div class="pagination-info">
            <span class="pagination-text">
              Menampilkan {{ ((page - 1) * limit) + 1 }} - {{ Math.min(page * limit, totalRows) }} 
              dari {{ totalRows.toLocaleString('id-ID') }} data
            </span>
          </div>
          <div class="pagination-controls">
            <button 
              :disabled="page === 1" 
              @click="gantiHalaman(page - 1)"
              class="pagination-btn prev-btn btn-press"
            >
              <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/>
              </svg>
              ← Sebelumnya
            </button>
            
            <div class="page-info">
              <span class="page-current">Halaman {{ page }} dari {{ totalPages }}</span>
            </div>
            
            <button 
              :disabled="page === totalPages" 
              @click="gantiHalaman(page + 1)"
              class="pagination-btn next-btn btn-press"
            >
              Berikutnya →
              <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/>
              </svg>
            </button>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-else class="empty-state fade-in">
        <div class="empty-icon">🙅</div>
        <h3 class="empty-title">Tidak ada data ditemukan</h3>
        <p class="empty-message">
          Tidak ada data penduduk yang sesuai dengan pencarian Anda.
          <br>Coba gunakan kata kunci yang berbeda.
        </p>
        <button @click="resetSearch" class="reset-btn btn-press hover-glow">
          <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 0 0 4.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 0 1-15.357-2m15.357 2H15"/>
          </svg>
          Reset Pencarian
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { GraphQLClient, gql } from 'graphql-request'

// Hasura config - menggunakan secret dari file asli
const endpoint = 'https://api801.tojounauna.go.id/v1/graphql'
const headers = { 'x-hasura-admin-secret': 'hsrSecret123' }
const client = new GraphQLClient(endpoint, { headers })

// State - sama dengan file asli
const penduduk = ref([])
const loading = ref(true)
const error = ref(null)
const keyword = ref('')
const page = ref(1)
const limit = 12
const totalRows = ref(0)
const totalPages = ref(1)

// Query ambil data - sama dengan file asli
const GET_PENDUDUK = gql`
  query GetPenduduk($limit: Int!, $offset: Int!, $keyword: String) {
    penduduk002_aggregate(
      where: {
        _or: [
          { nik: { _ilike: $keyword } }
          { nama: { _ilike: $keyword } }
          { desa_kel: { _ilike: $keyword } }
        ]
      }
    ) {
      aggregate {
        count
      }
    }
    penduduk002(
      limit: $limit
      offset: $offset
      order_by: { nama: asc }
      where: {
        _or: [
          { nik: { _ilike: $keyword } }
          { nama: { _ilike: $keyword } }
          { desa_kel: { _ilike: $keyword } }
        ]
      }
    ) {
      nik
      nama
      ttl
      desa_kel
      kecamatan
    }
  }
`

// Helper functions
const getInitials = (nama) => {
  return nama
    .split(' ')
    .map(word => word.charAt(0).toUpperCase())
    .slice(0, 2)
    .join('')
}

// Ambil data - sama dengan file asli
const fetchPenduduk = async () => {
  loading.value = true
  error.value = null
  const offset = (page.value - 1) * limit
  const formattedKeyword = keyword.value ? `%${keyword.value}%` : "%%"

  try {
    const data = await client.request(GET_PENDUDUK, {
      limit,
      offset,
      keyword: formattedKeyword,
    })
    penduduk.value = data.penduduk002
    totalRows.value = data.penduduk002_aggregate.aggregate.count
    totalPages.value = Math.ceil(totalRows.value / limit)
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

// Cari - sama dengan file asli
const cariPenduduk = () => {
  page.value = 1
  fetchPenduduk()
}

// Ganti halaman - sama dengan file asli
const gantiHalaman = (halamanBaru) => {
  page.value = halamanBaru
  fetchPenduduk()
}

// Reset search - fungsi tambahan
const resetSearch = () => {
  keyword.value = ''
  page.value = 1
  fetchPenduduk()
}

// On mount - sama dengan file asli
onMounted(() => {
  fetchPenduduk()
})
</script>

<style scoped>
/* Container */
.penduduk-container {
  font-family: 'Inter', sans-serif;
  color: #374151;
  background: #f8fafc;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  font-size: 14px;
}

/* Hero Section */
.hero-section {
  background: linear-gradient(135deg, #2563eb, #3b82f6);
  color: white;
  padding: 2rem 1.5rem;
  border-radius: 0 0 1.5rem 1.5rem;
  position: relative;
  overflow: hidden;
}
.hero-title {
  font-size: 1.4rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: .4rem;
}
.hero-subtitle {
  font-size: .9rem;
  margin-top: .3rem;
  opacity: .9;
}

/* Main Content */
.main-content {
  padding: 1.5rem 1rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* Search Section */
.search-section {
  margin-bottom: 1rem;
}
.search-form {
  display: flex;
  gap: .5rem;
  flex-wrap: wrap;
}
.search-input-group {
  display: flex;
  align-items: center;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: .75rem;
  padding: .4rem .7rem;
  flex: 1;
  transition: all .3s ease;
}
.search-input-group:focus-within {
  border-color: #2563eb;
  box-shadow: 0 0 0 2px rgba(37,99,235,.2);
}
.search-icon {
  margin-right: .4rem;
  color: #9ca3af;
}
.search-input {
  border: none;
  outline: none;
  flex: 1;
  font-size: .9rem;
  background: transparent;
}
.search-btn {
  display: flex;
  align-items: center;
  gap: .3rem;
  background: linear-gradient(135deg,#2563eb,#1d4ed8);
  color: white;
  padding: .5rem 1rem;
  border: none;
  border-radius: .7rem;
  cursor: pointer;
  font-weight: 500;
  font-size: .9rem;
  transition: transform .2s ease, box-shadow .2s ease;
}
.search-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(37,99,235,.25);
}

/* Stats Card */
.stats-card {
  display: grid;
  grid-template-columns: repeat(auto-fit,minmax(160px,1fr));
  gap: .8rem;
  margin: 1rem 0;
}
.stat-item {
  background: white;
  border-radius: .9rem;
  padding: 1rem;
  display: flex;
  align-items: center;
  gap: .8rem;
  box-shadow: 0 2px 6px rgba(0,0,0,.04);
  transition: transform .2s ease, box-shadow .2s ease;
}
.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 10px rgba(0,0,0,.08);
}
.stat-icon {
  font-size: 1.2rem;
  flex-shrink: 0;
}
.stat-number {
  font-size: 1.1rem;
  font-weight: 700;
  color: #111827;
}
.stat-label {
  font-size: .8rem;
  color: #6b7280;
}

/* ===== PERBAIKAN TABEL UTAMA ===== */
.table-container {
  background: white;
  border-radius: .9rem;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0,0,0,.05);
}

.table-scroll {
  overflow-x: auto;
  overflow-y: visible;
}

.modern-table {
  width: 100%;
  border-collapse: collapse;
  font-size: .9rem;
  min-width: 980px; /* Memastikan tabel tidak terlalu sempit */
}

/* Header dengan sticky dan lebar tetap */
.modern-table th {
  background: #f1f5f9;
  padding: .7rem .5rem;
  text-align: left;
  font-weight: 600;
  color: #334155;
  white-space: nowrap;
  position: sticky;
  top: 0;
  z-index: 10;
  border-bottom: 2px solid #e2e8f0;
}

/* Lebar kolom yang konsisten dan proporsional */
.th-no, .td-no { 
  width: 70px; 
  min-width: 70px; 
  text-align: center;
}
.th-nik, .td-nik { 
  width: 190px; 
  min-width: 190px; 
}
.th-nama, .td-nama { 
  width: 220px; 
  min-width: 220px; 
}
.th-ttl, .td-ttl { 
  width: 170px; 
  min-width: 170px; 
}
.th-alamat, .td-alamat { 
  width: 240px; 
  min-width: 240px; 
}
.th-kecamatan, .td-kecamatan { 
  width: 150px; 
  min-width: 150px; 
}

.modern-table td {
  padding: .75rem .5rem;
  border-top: 1px solid #e5e7eb;
  vertical-align: middle;
  white-space: nowrap; /* Mencegah text wrap ke baris baru */
  overflow: hidden;
  text-overflow: ellipsis; /* Tambah ... jika teks terlalu panjang */
}

.table-row:hover {
  background: #f9fafb;
}

/* Container untuk setiap cell dengan alignment yang baik */
.number-badge {
  background: #e0f2fe;
  color: #0369a1;
  font-weight: 600;
  padding: .3rem .6rem;
  border-radius: .5rem;
  font-size: .8rem;
  display: inline-block;
}

.nik-container {
  display: flex;
  align-items: center;
  gap: .5rem;
  min-width: 0; /* Memungkinkan flex shrink */
}
.nik-icon {
  font-size: .9rem;
  flex-shrink: 0;
}
.nik-text {
  font-family: 'Courier New', monospace;
  font-size: .85rem;
  background: #f8fafc;
  padding: .2rem .4rem;
  border-radius: .3rem;
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}

.nama-container {
  display: flex;
  align-items: center;
  gap: .6rem;
  min-width: 0;
}
.avatar {
  width: 30px; 
  height: 30px;
  background: linear-gradient(135deg,#6366f1,#8b5cf6);
  color: white;
  font-weight: 600;
  border-radius: 50%;
  display: flex; 
  align-items: center; 
  justify-content: center;
  font-size: .8rem;
  flex-shrink: 0;
}
.nama-text {
  overflow: hidden;
  text-overflow: ellipsis;
  font-weight: 500;
  min-width: 0;
}

.ttl-container {
  display: flex;
  align-items: center;
  gap: .5rem;
  min-width: 0;
}
.calendar-icon {
  font-size: .9rem;
  flex-shrink: 0;
}
.ttl-text {
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}

.alamat-container {
  display: flex;
  align-items: center;
  gap: .5rem;
  min-width: 0;
}
.location-icon {
  font-size: .9rem;
  flex-shrink: 0;
}
.alamat-text {
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}

.kecamatan-badge {
  background: #dcfce7;
  color: #15803d;
  font-weight: 500;
  padding: .3rem .6rem;
  border-radius: .5rem;
  font-size: .8rem;
  display: inline-block;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
}

/* Pagination */
.pagination-wrapper {
  display: flex;
  flex-direction: column;
  gap: .6rem;
  margin-top: 1.2rem;
  align-items: center;
}
.pagination-controls {
  display: flex;
  align-items: center;
  gap: .8rem;
  flex-wrap: wrap;
  justify-content: center;
}
.pagination-btn {
  display: flex;
  align-items: center;
  gap: .3rem;
  padding: .5rem 1rem;
  border-radius: .6rem;
  border: none;
  background: #f1f5f9;
  cursor: pointer;
  font-weight: 500;
  font-size: .85rem;
  transition: background .2s ease, transform .2s ease;
}
.pagination-btn:hover:not(:disabled) {
  background: #e2e8f0;
  transform: translateY(-1px);
}
.pagination-btn:disabled {
  opacity: .5;
  cursor: not-allowed;
}

/* Empty / Error / Loading */
.loading-state, .error-state, .empty-state {
  text-align: center;
  margin-top: 2rem;
  font-size: .9rem;
}
.spinner {
  width: 32px; 
  height: 32px;
  border: 3px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto .8rem;
}
@keyframes spin { 
  to { transform: rotate(360deg); } 
}

.error-title, .empty-title {
  font-weight: 600;
  margin-bottom: .4rem;
  font-size: 1.1rem;
}
.error-message, .empty-message {
  color: #6b7280;
  font-size: .9rem;
}

/* Buttons */
.retry-btn, .reset-btn {
  margin-top: 1rem;
  padding: .6rem 1.2rem;
  border: none;
  border-radius: .7rem;
  font-weight: 500;
  font-size: .9rem;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: .4rem;
  background: linear-gradient(135deg,#ef4444,#dc2626);
  color: white;
  transition: transform .2s ease, box-shadow .2s ease;
}
.retry-btn:hover, .reset-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(239,68,68,.3);
}

/* Responsive */
@media (max-width: 768px) {
  .main-content {
    padding: 1rem 0.5rem;
  }
  
  .search-form {
    flex-direction: column;
  }
  
  .search-btn {
    width: 100%;
    justify-content: center;
  }
  
  .stats-card {
    grid-template-columns: 1fr;
  }
  
  .pagination-controls {
    flex-direction: column;
    gap: .5rem;
  }
  
  .page-info {
    order: -1;
  }
}

/* Smooth transitions */
.fade-in {
  animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.card-entrance {
  animation: cardSlideUp 0.6s ease-out;
}

@keyframes cardSlideUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>