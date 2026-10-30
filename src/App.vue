<script setup>
import { ref, computed, watch } from 'vue'

const TASKS_KEY = 'taskify-tasks'
const DARK_MODE_KEY = 'taskify-dark-mode'

const defaultTasks = [
  {
    id: 1,
    title: 'Tugas matematika lanjutan',
    date: '2026-11-02',
    note: 'Kerjakan soal bab 3–5',
    status: true
  },
  {
    id: 2,
    title: 'Tugas sejarah kebudayaan',
    date: '2026-11-05',
    note: 'Baca bab 4 dan buat rangkuman',
    status: false
  },
  {
    id: 3,
    title: 'Desain mockup UI Taskify Vue',
    date: '2026-11-08',
    note: 'Gunakan warna pastel dan icon minimalis',
    status: false
  }
]

const loadTasks = () => {
  try {
    const saved = localStorage.getItem(TASKS_KEY)

    if (saved !== null) {
      const parsed = JSON.parse(saved)

      if (Array.isArray(parsed)) {
        // Menghapus data kategori lama jika sebelumnya pernah tersimpan
        return parsed.map(task => ({
          id: task.id,
          title: task.title,
          date: task.date || '',
          note: task.note || '',
          status: Boolean(task.status)
        }))
      }
    }
  } catch (error) {
    console.error('Gagal membaca tugas:', error)
  }

  return defaultTasks.map(task => ({ ...task }))
}

const loadDarkMode = () => {
  try {
    const saved = localStorage.getItem(DARK_MODE_KEY)

    if (saved !== null) {
      return JSON.parse(saved)
    }
  } catch (error) {
    console.error('Gagal membaca mode:', error)
  }

  return false
}

const isDarkMode = ref(loadDarkMode())

const searchQuery = ref('')
const currentFilter = ref('Semua')

const inputTask = ref('')
const inputDate = ref('')
const inputNote = ref('')

const selectedTask = ref(null)

const editingTaskId = ref(null)
const editingTitle = ref('')

const tasks = ref(loadTasks())

const saveTasks = () => {
  try {
    localStorage.setItem(
      TASKS_KEY,
      JSON.stringify(tasks.value)
    )
  } catch (error) {
    console.error('Gagal menyimpan tugas:', error)
  }
}

watch(
  tasks,
  () => {
    saveTasks()
  },
  {
    deep: true,
    flush: 'sync'
  }
)

watch(
  isDarkMode,
  value => {
    try {
      localStorage.setItem(
        DARK_MODE_KEY,
        JSON.stringify(value)
      )
    } catch (error) {
      console.error('Gagal menyimpan mode:', error)
    }
  },
  {
    flush: 'sync'
  }
)

const addTask = () => {
  if (!inputTask.value.trim()) {
    alert('Masukkan nama tugas terlebih dahulu.')
    return
  }

  tasks.value.push({
    id: Date.now(),
    title: inputTask.value.trim(),
    date: inputDate.value,
    note: inputNote.value.trim(),
    status: false
  })

  saveTasks()

  inputTask.value = ''
  inputDate.value = ''
  inputNote.value = ''
}

const deleteTask = id => {
  if (!confirm('Apakah Anda yakin ingin menghapus tugas ini?')) {
    return
  }

  const index = tasks.value.findIndex(
    task => task.id === id
  )

  if (index !== -1) {
    tasks.value.splice(index, 1)
    saveTasks()
  }

  if (selectedTask.value?.id === id) {
    selectedTask.value = null
  }

  if (editingTaskId.value === id) {
    cancelEdit()
  }
}

const startEdit = task => {
  editingTaskId.value = task.id
  editingTitle.value = task.title
}

const saveEdit = task => {
  if (!editingTitle.value.trim()) {
    alert('Nama tugas tidak boleh kosong.')
    return
  }

  task.title = editingTitle.value.trim()

  saveTasks()

  editingTaskId.value = null
  editingTitle.value = ''
}

const cancelEdit = () => {
  editingTaskId.value = null
  editingTitle.value = ''
}

const toggleTaskStatus = task => {
  task.status = !task.status
  saveTasks()
}

const openDetail = task => {
  if (editingTaskId.value === task.id) {
    return
  }

  selectedTask.value = {
    ...task
  }
}

const saveTask = () => {
  if (!selectedTask.value) {
    return
  }

  if (!selectedTask.value.title.trim()) {
    alert('Nama tugas tidak boleh kosong.')
    return
  }

  const index = tasks.value.findIndex(
    task => task.id === selectedTask.value.id
  )

  if (index !== -1) {
    tasks.value[index] = {
      ...selectedTask.value,
      title: selectedTask.value.title.trim()
    }

    saveTasks()
  }

  selectedTask.value = null
}

const completedCount = computed(() => {
  return tasks.value.filter(
    task => task.status
  ).length
})

const totalCount = computed(() => {
  return tasks.value.length
})

const filteredTasks = computed(() => {
  return tasks.value.filter(task => {
    const matchesSearch = task.title
      .toLowerCase()
      .includes(
        searchQuery.value
          .toLowerCase()
          .trim()
      )

    if (currentFilter.value === 'Aktif') {
      return matchesSearch && !task.status
    }

    if (currentFilter.value === 'Selesai') {
      return matchesSearch && task.status
    }

    return matchesSearch
  })
})

const formatDate = date => {
  if (!date) {
    return 'Belum ada tanggal'
  }

  return new Date(
    date + 'T00:00:00'
  ).toLocaleDateString('id-ID', {
    day: '2-digit',
    month: 'short',
    year: 'numeric'
  })
}
</script>

<template>
  <main
    class="page"
    :class="{ dark: isDarkMode }"
  >
    <!-- DEKORASI BACKGROUND -->
    <div class="decoration">
      <div class="circle circle1"></div>
      <div class="circle circle2"></div>

      <span class="flower flower1">✿</span>
      <span class="flower flower2">✿</span>
    </div>

    <!-- CONTAINER UTAMA -->
    <section class="app-container">

      <!-- HEADER -->
      <header class="header">
        <div class="brand">
          <div class="logo">
            ✓
          </div>

          <h1>
            Taskify <span>Vue</span>
          </h1>
        </div>

        <!-- MODE -->
        <div class="mode-switch">
          <button
            class="light-button"
            :class="{ active: !isDarkMode }"
            @click="isDarkMode = false"
          >
            ☼ Terang
          </button>

          <button
            class="dark-button"
            :class="{ active: isDarkMode }"
            @click="isDarkMode = true"
          >
            ☾ Gelap
          </button>
        </div>
      </header>

      <!-- SEARCH -->
      <div class="search-box">
        <span class="search-icon">
          ⌕
        </span>

        <input
          v-model="searchQuery"
          type="text"
          placeholder="Cari tugas..."
        />
      </div>

      <!-- TAMBAH TUGAS -->
      <div class="add-form">
        <input
          v-model="inputTask"
          type="text"
          placeholder="Masukkan tugas baru..."
          @keyup.enter="addTask"
        />

        <input
          v-model="inputDate"
          type="date"
        />

        <input
          v-model="inputNote"
          type="text"
          placeholder="Catatan (opsional)"
          @keyup.enter="addTask"
        />

        <button
          class="add-button"
          @click="addTask"
        >
          ＋ Tambah
        </button>
      </div>

      <!-- FILTER STATUS -->
      <div class="filter-section">
        <div class="filters">
          <button
            :class="{
              selected: currentFilter === 'Semua'
            }"
            @click="currentFilter = 'Semua'"
          >
            Semua
          </button>

          <button
            :class="{
              selected: currentFilter === 'Aktif'
            }"
            @click="currentFilter = 'Aktif'"
          >
            Belum
          </button>

          <button
            :class="{
              selected: currentFilter === 'Selesai'
            }"
            @click="currentFilter = 'Selesai'"
          >
            Selesai
          </button>
        </div>

        <div class="counter">
          ✓ {{ completedCount }}/{{ totalCount }} Selesai
        </div>
      </div>

      <!-- CONTENT -->
      <div
        class="content"
        :class="{
          'detail-open': selectedTask
        }"
      >
        <!-- DAFTAR TUGAS -->
        <div class="task-list">
          <TransitionGroup name="task-animation">
            <div
              v-for="task in filteredTasks"
              :key="task.id"
              class="task-card"
            >
              <!-- CHECKBOX -->
              <button
                class="checkbox"
                :class="{
                  checked: task.status
                }"
                @click.stop="toggleTaskStatus(task)"
              >
                <span v-if="task.status">
                  ✓
                </span>
              </button>

              <!-- INFORMASI TUGAS -->
              <div
                class="task-info"
                @click="openDetail(task)"
              >
                <div class="task-top">
                  <div class="task-title-area">

                    <!-- EDIT TITLE -->
                    <input
                      v-if="editingTaskId === task.id"
                      v-model="editingTitle"
                      class="edit-title-input"
                      type="text"
                      @click.stop
                      @keyup.enter="saveEdit(task)"
                      @keyup.esc="cancelEdit"
                    />

                    <!-- TITLE -->
                    <h3
                      v-else
                      :class="{
                        completed: task.status
                      }"
                    >
                      {{ task.title }}
                    </h3>
                  </div>

                  <!-- STATUS -->
                  <span
                    class="status"
                    :class="
                      task.status
                        ? 'done'
                        : 'active'
                    "
                  >
                    {{
                      task.status
                        ? 'Selesai'
                        : 'Belum'
                    }}
                  </span>
                </div>

                <!-- TANGGAL & CATATAN -->
                <div class="task-meta">
                  <span class="date">
                    📅 {{ formatDate(task.date) }}
                  </span>

                  <span
                    v-if="task.note"
                    class="note"
                  >
                    ▤ {{ task.note }}
                  </span>
                </div>
              </div>

              <!-- ACTION -->
              <div class="task-actions">
                <template
                  v-if="editingTaskId === task.id"
                >
                  <button
                    class="edit save-edit"
                    title="Simpan"
                    @click.stop="saveEdit(task)"
                  >
                    ✓
                  </button>

                  <button
                    class="edit cancel-edit"
                    title="Batal"
                    @click.stop="cancelEdit"
                  >
                    ×
                  </button>
                </template>

                <button
                  v-else
                  class="edit"
                  title="Edit"
                  @click.stop="startEdit(task)"
                >
                  ✎
                </button>

                <button
                  class="delete"
                  title="Hapus"
                  @click.stop="deleteTask(task.id)"
                >
                  🗑
                </button>
              </div>
            </div>
          </TransitionGroup>

          <!-- EMPTY -->
          <Transition name="empty-animation">
            <div
              v-if="filteredTasks.length === 0"
              class="empty"
            >
              Tidak ada tugas ditemukan.
            </div>
          </Transition>
        </div>

        <!-- DETAIL TUGAS -->
        <Transition name="detail-animation">
          <aside
            v-if="selectedTask"
            class="detail-panel"
          >
            <!-- HEADER DETAIL -->
            <div class="detail-header">
              <div class="detail-title">
                <div class="detail-icon">
                  ▤
                </div>

                <h2>
                  Detail Tugas
                </h2>
              </div>

              <button
                class="close"
                @click="selectedTask = null"
              >
                ×
              </button>
            </div>

            <!-- NAMA -->
            <div class="form-group">
              <label>
                Nama Tugas
              </label>

              <input
                v-model="selectedTask.title"
                type="text"
              />
            </div>

            <!-- TANGGAL -->
            <div class="form-group">
              <label>
                📅 Tanggal Pengumpulan
              </label>

              <input
                v-model="selectedTask.date"
                type="date"
              />
            </div>

            <!-- CATATAN -->
            <div class="form-group">
              <label>
                ▤ Catatan
              </label>

              <textarea
                v-model="selectedTask.note"
                placeholder="Tambahkan catatan..."
              ></textarea>
            </div>

            <!-- STATUS -->
            <div class="form-group">
              <label>
                Status
              </label>

              <div class="status-switch">
                <button
                  :class="{
                    selected: !selectedTask.status
                  }"
                  @click="selectedTask.status = false"
                >
                  Aktif
                </button>

                <button
                  :class="{
                    selected: selectedTask.status
                  }"
                  @click="selectedTask.status = true"
                >
                  Selesai
                </button>
              </div>
            </div>

            <!-- BUTTON DETAIL -->
            <div class="detail-actions">
              <button
                class="cancel"
                @click="selectedTask = null"
              >
                Batal
              </button>

              <button
                class="save"
                @click="saveTask"
              >
                ✓ Simpan
              </button>
            </div>
          </aside>
        </Transition>
      </div>
    </section>
  </main>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

button,
input,
textarea {
  font-family: inherit;
}

button {
  cursor: pointer;
}

/* ========================================= */
/* PAGE */
/* ========================================= */

.page {
  --text: #3f4964;
  --muted: #7d849c;
  --pink: #f43f7f;
  --pink-dark: #c51e5c;

  width: 100%;
  min-height: 100vh;

  display: flex;
  justify-content: center;
  align-items: flex-start;

  padding: 22px;

  position: relative;
  overflow: hidden;

  font-family: Inter, Poppins, Arial, sans-serif;

  color: var(--text);

  background:
    radial-gradient(
      circle at 10% 15%,
      rgba(255, 132, 179, 0.55),
      transparent 27%
    ),
    radial-gradient(
      circle at 90% 15%,
      rgba(255, 198, 220, 0.7),
      transparent 28%
    ),
    radial-gradient(
      circle at 75% 90%,
      rgba(255, 118, 170, 0.45),
      transparent 30%
    ),
    linear-gradient(
      135deg,
      #ffd6e5 0%,
      #fff0f6 45%,
      #ffc9dc 100%
    );

  transition:
    background 0.4s ease,
    color 0.4s ease;
}

/* ========================================= */
/* DARK MODE */
/* ========================================= */

.page.dark {
  --text: #f8e8ef;
  --muted: #c9aab8;
  --pink: #ef4b88;
  --pink-dark: #a91d54;

  background:
    radial-gradient(
      circle at 10% 15%,
      rgba(190, 35, 94, 0.45),
      transparent 28%
    ),
    radial-gradient(
      circle at 90% 20%,
      rgba(124, 24, 65, 0.45),
      transparent 30%
    ),
    radial-gradient(
      circle at 70% 90%,
      rgba(220, 50, 112, 0.28),
      transparent 30%
    ),
    linear-gradient(
      135deg,
      #180910 0%,
      #321321 50%,
      #16080e 100%
    );
}

/* ========================================= */
/* DECORATION */
/* ========================================= */

.decoration {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.circle {
  position: absolute;

  border-radius: 50%;

  background:
    linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.4),
      rgba(255, 73, 139, 0.18)
    );

  border:
    1px solid rgba(255, 255, 255, 0.4);

  box-shadow:
    inset 0 0 30px rgba(255, 255, 255, 0.3),
    0 15px 40px rgba(211, 59, 116, 0.12);
}

.circle1 {
  width: 190px;
  height: 190px;
  top: -80px;
  left: -70px;
}

.circle2 {
  width: 220px;
  height: 220px;
  right: -80px;
  bottom: -80px;
}

.flower {
  position: absolute;

  color:
    rgba(255, 70, 140, 0.18);

  font-size: 52px;

  animation:
    flowerFloat 6s ease-in-out infinite;
}

.flower1 {
  left: 3%;
  bottom: 8%;
}

.flower2 {
  right: 4%;
  top: 10%;

  animation-delay: 1.5s;
}

@keyframes flowerFloat {
  0%,
  100% {
    transform:
      translateY(0)
      rotate(0deg);
  }

  50% {
    transform:
      translateY(-10px)
      rotate(5deg);
  }
}

/* ========================================= */
/* GLASS CONTAINER */
/* ========================================= */

.app-container {
  width: 100%;
  max-width: 900px;
  min-height: 560px;

  position: relative;
  z-index: 2;

  padding: 24px;

  border-radius: 25px;

  background:
    linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.48),
      rgba(255, 255, 255, 0.22)
    );

  border:
    1px solid rgba(255, 255, 255, 0.7);

  backdrop-filter:
    blur(22px)
    saturate(150%);

  -webkit-backdrop-filter:
    blur(22px)
    saturate(150%);

  box-shadow:
    0 20px 50px rgba(182, 49, 99, 0.14),
    inset 0 1px 0 rgba(255, 255, 255, 0.8),
    inset 0 -1px 0 rgba(255, 255, 255, 0.25);

  transition:
    background 0.4s ease,
    border-color 0.4s ease,
    box-shadow 0.4s ease;
}

.app-container::before {
  content: "";

  position: absolute;

  top: 0;
  left: 8%;
  right: 8%;

  height: 1px;

  background:
    linear-gradient(
      90deg,
      transparent,
      rgba(255, 255, 255, 0.95),
      transparent
    );

  pointer-events: none;
}

.dark .app-container {
  background:
    linear-gradient(
      135deg,
      rgba(91, 30, 56, 0.48),
      rgba(35, 12, 23, 0.48)
    );

  border-color:
    rgba(255, 143, 184, 0.18);

  box-shadow:
    0 20px 55px rgba(0, 0, 0, 0.3),
    inset 0 1px 0 rgba(255, 180, 207, 0.12);
}

/* ========================================= */
/* HEADER */
/* ========================================= */

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;

  gap: 16px;

  margin-bottom: 19px;
}

.brand {
  display: flex;
  align-items: center;

  gap: 10px;
}

.logo {
  width: 42px;
  height: 42px;

  display: flex;
  justify-content: center;
  align-items: center;

  border-radius: 13px;

  color: white;

  font-size: 24px;
  font-weight: 800;

  background:
    linear-gradient(
      135deg,
      rgba(255, 94, 151, 0.95),
      rgba(229, 44, 112, 0.95)
    );

  border:
    1px solid rgba(255, 255, 255, 0.55);

  box-shadow:
    0 8px 18px rgba(224, 46, 112, 0.25),
    inset 0 1px 0 rgba(255, 255, 255, 0.45);
}

.brand h1 {
  margin: 0;

  font-size: 28px;
  font-weight: 800;

  color: #39425e;
}

.brand h1 span {
  color: var(--pink);
}

.dark .brand h1 {
  color: #f8e8ef;
}

/* ========================================= */
/* MODE SWITCH */
/* ========================================= */

.mode-switch {
  display: flex;

  padding: 4px;

  border-radius: 18px;

  background:
    rgba(255, 255, 255, 0.28);

  border:
    1px solid rgba(255, 255, 255, 0.6);

  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);

  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    0 6px 15px rgba(183, 58, 105, 0.08);
}

.dark .mode-switch {
  background:
    rgba(92, 29, 56, 0.35);

  border-color:
    rgba(255, 124, 174, 0.18);
}

.mode-switch button {
  border: none;

  padding: 8px 13px;

  border-radius: 14px;

  background: transparent;

  color: #806a77;

  font-size: 13px;
  font-weight: 650;

  transition:
    transform 0.2s ease,
    background 0.3s ease,
    box-shadow 0.3s ease,
    color 0.3s ease;
}

.dark .mode-switch button {
  color: #d6b5c2;
}

.mode-switch button:hover {
  transform: translateY(-1px);
}

.mode-switch .light-button.active {
  color: white;

  background:
    linear-gradient(
      135deg,
      #ff6399,
      #ef3b7d
    );

  box-shadow:
    0 5px 13px rgba(239, 59, 125, 0.25);
}

.mode-switch .dark-button.active {
  color: white;

  background:
    linear-gradient(
      135deg,
      #bd2861,
      #801442
    );

  box-shadow:
    0 5px 14px rgba(96, 12, 48, 0.4);
}

/* ========================================= */
/* SEARCH */
/* ========================================= */

.search-box {
  height: 45px;

  display: flex;
  align-items: center;

  gap: 9px;

  padding: 0 14px;

  margin-bottom: 14px;

  border-radius: 13px;

  background:
    rgba(255, 255, 255, 0.32);

  border:
    1px solid rgba(255, 255, 255, 0.65);

  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);

  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    0 6px 15px rgba(186, 57, 106, 0.06);

  transition:
    border-color 0.3s ease,
    box-shadow 0.3s ease;
}

.search-box:focus-within {
  border-color:
    rgba(244, 63, 127, 0.55);

  box-shadow:
    0 0 0 3px rgba(244, 63, 127, 0.08),
    inset 0 1px 0 rgba(255, 255, 255, 0.7);
}

.search-icon {
  font-size: 21px;
  color: #53668c;
}

.search-box input {
  width: 100%;

  border: none;
  outline: none;

  background: transparent;

  color: var(--text);

  font-size: 14px;
}

.search-box input::placeholder {
  color: #9097a9;
}

/* ========================================= */
/* ADD FORM */
/* ========================================= */

.add-form {
  display: grid;

  grid-template-columns:
    minmax(190px, 2fr)
    minmax(135px, 1fr)
    minmax(180px, 1.4fr)
    110px;

  gap: 8px;

  margin-bottom: 17px;
}

.add-form input {
  width: 100%;
  height: 44px;

  padding: 0 12px;

  border-radius: 12px;

  outline: none;

  color: var(--text);

  background:
    rgba(255, 255, 255, 0.32);

  border:
    1px solid rgba(255, 255, 255, 0.65);

  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);

  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.55);

  font-size: 13px;

  transition:
    border-color 0.3s ease,
    box-shadow 0.3s ease;
}

.add-form input:focus {
  border-color:
    rgba(244, 63, 127, 0.55);

  box-shadow:
    0 0 0 3px rgba(244, 63, 127, 0.08);
}

.add-form input::placeholder {
  color: #9298a8;
}

.add-button {
  border:
    1px solid rgba(255, 255, 255, 0.4);

  border-radius: 12px;

  color: white;

  font-size: 13px;
  font-weight: 700;

  background:
    linear-gradient(
      135deg,
      #ff4385,
      #ec3177
    );

  box-shadow:
    0 8px 18px rgba(224, 47, 112, 0.22),
    inset 0 1px 0 rgba(255, 255, 255, 0.4);

  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}

.add-button:hover {
  transform: translateY(-2px);

  box-shadow:
    0 11px 22px rgba(224, 47, 112, 0.28);
}

.add-button:active {
  transform: scale(0.96);
}

.dark .search-box,
.dark .add-form input,
.dark .form-group input,
.dark .form-group textarea {
  background:
    rgba(87, 29, 54, 0.32);

  border-color:
    rgba(255, 141, 183, 0.16);

  box-shadow:
    inset 0 1px 0 rgba(255, 190, 214, 0.08);
}

.dark .search-icon {
  color: #d8aebe;
}

.dark input::placeholder,
.dark textarea::placeholder {
  color: #a98996;
}

.dark input[type="date"] {
  color-scheme: dark;
}

/* ========================================= */
/* FILTER */
/* ========================================= */

.filter-section {
  display: flex;
  align-items: center;
  justify-content: space-between;

  gap: 12px;

  margin-bottom: 14px;
}

.filters {
  display: flex;
  gap: 6px;
}

.filters button {
  border:
    1px solid rgba(255, 255, 255, 0.45);

  padding: 7px 14px;

  border-radius: 12px;

  color: #5c6984;

  background:
    rgba(255, 255, 255, 0.28);

  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);

  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.45);

  font-size: 12px;
  font-weight: 600;

  transition:
    transform 0.2s ease,
    background 0.2s ease;
}

.filters button:hover {
  transform: translateY(-1px);

  background:
    rgba(255, 255, 255, 0.45);
}

.filters button.selected {
  color: white;

  background:
    linear-gradient(
      135deg,
      #ff4b88,
      #f03277
    );

  box-shadow:
    0 6px 14px rgba(229, 49, 116, 0.2);
}

.dark .filters button {
  color: #dabac6;

  background:
    rgba(94, 31, 58, 0.3);

  border-color:
    rgba(255, 138, 181, 0.14);
}

.dark .filters button.selected {
  color: white;

  background:
    linear-gradient(
      135deg,
      #c92b66,
      #911646
    );
}

/* ========================================= */
/* COUNTER */
/* ========================================= */

.counter {
  padding: 6px 11px;

  border-radius: 12px;

  color: #57627a;

  background:
    rgba(255, 255, 255, 0.3);

  border:
    1px solid rgba(255, 255, 255, 0.55);

  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);

  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.45);

  font-size: 12px;
}

.dark .counter {
  color: #e7c7d3;

  background:
    rgba(91, 29, 55, 0.3);

  border-color:
    rgba(255, 135, 180, 0.15);
}

/* ========================================= */
/* CONTENT */
/* ========================================= */

.content {
  display: grid;

  grid-template-columns: 1fr;

  gap: 14px;

  align-items: start;
}

.content.detail-open {
  grid-template-columns:
    minmax(0, 1.6fr)
    minmax(240px, 0.9fr);
}

.task-list {
  display: flex;
  flex-direction: column;

  gap: 9px;
}

/* ========================================= */
/* TASK CARD */
/* ========================================= */

.task-card {
  min-height: 72px;

  display: flex;
  align-items: center;

  gap: 10px;

  padding: 12px 14px;

  position: relative;

  border-radius: 15px;

  background:
    linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.4),
      rgba(255, 255, 255, 0.2)
    );

  border:
    1px solid rgba(255, 255, 255, 0.6);

  backdrop-filter:
    blur(16px)
    saturate(140%);

  -webkit-backdrop-filter:
    blur(16px)
    saturate(140%);

  box-shadow:
    0 8px 22px rgba(177, 52, 99, 0.07),
    inset 0 1px 0 rgba(255, 255, 255, 0.65);

  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease,
    background 0.3s ease;
}

.task-card:hover {
  transform: translateY(-3px);

  background:
    linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.52),
      rgba(255, 255, 255, 0.27)
    );

  box-shadow:
    0 12px 27px rgba(186, 51, 102, 0.12),
    inset 0 1px 0 rgba(255, 255, 255, 0.8);
}

.dark .task-card {
  background:
    linear-gradient(
      135deg,
      rgba(100, 34, 63, 0.38),
      rgba(53, 17, 34, 0.3)
    );

  border-color:
    rgba(255, 140, 182, 0.14);

  box-shadow:
    0 8px 22px rgba(0, 0, 0, 0.18),
    inset 0 1px 0 rgba(255, 185, 211, 0.08);
}

/* ========================================= */
/* CHECKBOX */
/* ========================================= */

.checkbox {
  width: 26px;
  height: 26px;

  flex-shrink: 0;

  display: flex;
  align-items: center;
  justify-content: center;

  border:
    1.5px solid rgba(225, 123, 160, 0.55);

  border-radius: 50%;

  color: white;

  background:
    rgba(255, 255, 255, 0.25);

  backdrop-filter: blur(8px);

  font-size: 13px;

  transition:
    transform 0.2s ease,
    background 0.25s ease;
}

.checkbox:hover {
  transform: scale(1.08);
}

.checkbox.checked {
  border-color:
    rgba(255, 255, 255, 0.45);

  background:
    linear-gradient(
      135deg,
      #ff5590,
      #ef3378
    );

  box-shadow:
    0 4px 10px rgba(239, 51, 120, 0.22);
}

/* ========================================= */
/* TASK INFO */
/* ========================================= */

.task-info {
  flex: 1;
  min-width: 0;
  cursor: pointer;
}

.task-top {
  display: flex;
  align-items: center;
  gap: 8px;
}

.task-title-area {
  flex: 1;
  min-width: 0;
}

.task-title-area h3 {
  margin: 0;

  color: var(--text);

  font-size: 14px;
  font-weight: 650;

  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.task-title-area h3.completed {
  color: #969cab;
  text-decoration: line-through;
}

.dark .task-title-area h3.completed {
  color: #a98795;
}

/* ========================================= */
/* EDIT INPUT */
/* ========================================= */

.edit-title-input {
  width: 100%;
  height: 32px;

  padding: 0 9px;

  border:
    1px solid rgba(244, 63, 127, 0.4);

  border-radius: 9px;

  outline: none;

  color: var(--text);

  background:
    rgba(255, 255, 255, 0.35);

  backdrop-filter: blur(10px);

  font-size: 13px;
}

.dark .edit-title-input {
  background:
    rgba(83, 27, 51, 0.45);

  border-color:
    rgba(255, 125, 173, 0.2);
}

/* ========================================= */
/* STATUS */
/* ========================================= */

.status {
  padding: 4px 10px;

  border-radius: 10px;

  font-size: 11px;

  border:
    1px solid rgba(255, 255, 255, 0.4);

  backdrop-filter: blur(8px);
}

.status.active {
  color: #416fc7;

  background:
    rgba(221, 235, 255, 0.55);
}

.status.done {
  color: #d83271;

  background:
    rgba(255, 221, 234, 0.55);
}

.dark .status.active {
  color: #ffc0d6;

  background:
    rgba(176, 35, 89, 0.25);
}

.dark .status.done {
  color: #ff9cbe;

  background:
    rgba(218, 45, 112, 0.2);
}

/* ========================================= */
/* META */
/* ========================================= */

.task-meta {
  display: flex;
  align-items: center;

  gap: 10px;

  margin-top: 6px;

  color: var(--muted);

  font-size: 11px;
}

.task-meta .date {
  color: #e63b7b;
  flex-shrink: 0;
}

.dark .task-meta .date {
  color: #ff85b1;
}

.note {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

/* ========================================= */
/* ACTION BUTTON */
/* ========================================= */

.task-actions {
  display: flex;
  align-items: center;

  gap: 4px;

  flex-shrink: 0;
}

.edit,
.delete {
  width: 30px;
  height: 30px;

  display: flex;
  justify-content: center;
  align-items: center;

  padding: 0;

  border:
    1px solid transparent;

  border-radius: 9px;

  background:
    rgba(255, 255, 255, 0.12);

  transition:
    transform 0.2s ease,
    background 0.2s ease,
    border-color 0.2s ease;
}

.edit {
  color: #d93674;
  font-size: 16px;
}

.delete {
  color: #8f96a6;
  font-size: 14px;
}

.edit:hover,
.delete:hover {
  transform: scale(1.1);

  background:
    rgba(255, 231, 240, 0.55);

  border-color:
    rgba(255, 255, 255, 0.55);
}

.dark .edit:hover,
.dark .delete:hover {
  background:
    rgba(167, 35, 85, 0.25);

  border-color:
    rgba(255, 133, 178, 0.12);
}

.save-edit {
  color: white;

  background:
    linear-gradient(
      135deg,
      #f84b89,
      #dc286b
    );
}

.cancel-edit {
  color: #806b76;

  background:
    rgba(255, 255, 255, 0.25);
}

/* ========================================= */
/* DETAIL PANEL */
/* ========================================= */

.detail-panel {
  padding: 16px;

  border-radius: 17px;

  background:
    linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.46),
      rgba(255, 255, 255, 0.22)
    );

  border:
    1px solid rgba(255, 255, 255, 0.65);

  backdrop-filter:
    blur(20px)
    saturate(150%);

  -webkit-backdrop-filter:
    blur(20px)
    saturate(150%);

  box-shadow:
    0 14px 32px rgba(161, 49, 91, 0.12),
    inset 0 1px 0 rgba(255, 255, 255, 0.7);
}

.dark .detail-panel {
  background:
    linear-gradient(
      135deg,
      rgba(99, 32, 61, 0.45),
      rgba(44, 14, 29, 0.4)
    );

  border-color:
    rgba(255, 136, 180, 0.15);

  box-shadow:
    0 14px 32px rgba(0, 0, 0, 0.25),
    inset 0 1px 0 rgba(255, 188, 212, 0.08);
}

.detail-header {
  display: flex;
  align-items: center;
  justify-content: space-between;

  margin-bottom: 14px;
}

.detail-title {
  display: flex;
  align-items: center;

  gap: 8px;
}

.detail-icon {
  width: 29px;
  height: 29px;

  display: flex;
  align-items: center;
  justify-content: center;

  border-radius: 9px;

  color: white;

  background:
    linear-gradient(
      135deg,
      #ff5591,
      #ee3478
    );

  border:
    1px solid rgba(255, 255, 255, 0.35);

  font-size: 13px;
}

.detail-title h2 {
  margin: 0;

  color: var(--text);

  font-size: 16px;
}

.close {
  border: none;

  color: var(--text);

  background: transparent;

  font-size: 21px;

  transition: transform 0.2s ease;
}

.close:hover {
  transform: rotate(90deg);
}

/* ========================================= */
/* DETAIL FORM */
/* ========================================= */

.form-group {
  margin-bottom: 12px;
}

.form-group label {
  display: block;

  margin-bottom: 6px;

  color: var(--text);

  font-size: 12px;
  font-weight: 650;
}

.form-group input,
.form-group textarea {
  width: 100%;

  border:
    1px solid rgba(255, 255, 255, 0.55);

  border-radius: 11px;

  outline: none;

  color: var(--text);

  background:
    rgba(255, 255, 255, 0.28);

  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);

  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.45);

  font-size: 12px;

  transition:
    border-color 0.3s ease,
    box-shadow 0.3s ease;
}

.form-group input:focus,
.form-group textarea:focus {
  border-color:
    rgba(244, 63, 127, 0.5);

  box-shadow:
    0 0 0 3px rgba(244, 63, 127, 0.07);
}

.form-group input {
  height: 40px;
  padding: 0 10px;
}

.form-group textarea {
  height: 76px;
  padding: 10px;
  resize: vertical;
}

/* ========================================= */
/* STATUS SWITCH */
/* ========================================= */

.status-switch {
  display: grid;

  grid-template-columns: 1fr 1fr;

  padding: 4px;

  border-radius: 13px;

  background:
    rgba(255, 255, 255, 0.25);

  border:
    1px solid rgba(255, 255, 255, 0.45);

  backdrop-filter: blur(12px);
}

.dark .status-switch {
  background:
    rgba(76, 24, 46, 0.35);

  border-color:
    rgba(255, 130, 176, 0.12);
}

.status-switch button {
  border: none;

  padding: 8px;

  border-radius: 10px;

  color: #76829b;

  background: transparent;

  font-size: 12px;

  transition:
    background 0.25s ease,
    color 0.25s ease;
}

.dark .status-switch button {
  color: #d4adbc;
}

.status-switch button.selected {
  color: white;

  background:
    linear-gradient(
      135deg,
      #ff4c88,
      #f02f76
    );

  box-shadow:
    0 5px 12px rgba(223, 45, 111, 0.18);
}

.dark .status-switch button.selected {
  background:
    linear-gradient(
      135deg,
      #c82a65,
      #8b1443
    );
}

/* ========================================= */
/* DETAIL ACTION */
/* ========================================= */

.detail-actions {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 8px;

  margin-top: 15px;
}

.detail-actions button {
  height: 38px;

  border-radius: 11px;

  font-size: 12px;
  font-weight: 650;

  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}

.detail-actions button:hover {
  transform: translateY(-1px);
}

.cancel {
  color: #66728b;

  background:
    rgba(255, 255, 255, 0.28);

  border:
    1px solid rgba(255, 255, 255, 0.6);

  backdrop-filter: blur(10px);
}

.dark .cancel {
  color: #dfbdca;

  background:
    rgba(83, 27, 51, 0.35);

  border-color:
    rgba(255, 137, 180, 0.15);
}

.save {
  border:
    1px solid rgba(255, 255, 255, 0.3);

  color: white;

  background:
    linear-gradient(
      135deg,
      #ff4b88,
      #ee2f76
    );

  box-shadow:
    0 7px 15px rgba(225, 47, 112, 0.2);
}

.dark .save {
  background:
    linear-gradient(
      135deg,
      #c82a65,
      #8b1443
    );
}

/* ========================================= */
/* EMPTY */
/* ========================================= */

.empty {
  padding: 30px;

  text-align: center;

  color: var(--muted);

  font-size: 13px;

  border-radius: 14px;

  background:
    rgba(255, 255, 255, 0.18);

  border:
    1px solid rgba(255, 255, 255, 0.4);

  backdrop-filter: blur(12px);
}

/* ========================================= */
/* ANIMASI TAMBAH */
/* ========================================= */

.task-animation-enter-active {
  animation:
    taskAdded 0.45s ease;
}

@keyframes taskAdded {
  0% {
    opacity: 0;

    transform:
      translateY(18px)
      scale(0.96);
  }

  60% {
    transform:
      translateY(-3px)
      scale(1.01);
  }

  100% {
    opacity: 1;

    transform:
      translateY(0)
      scale(1);
  }
}

/* ========================================= */
/* ANIMASI HAPUS */
/* ========================================= */

.task-animation-leave-active {
  animation:
    taskDeleted 0.38s ease forwards;
}

@keyframes taskDeleted {
  from {
    opacity: 1;

    transform:
      translateX(0)
      scale(1);
  }

  to {
    opacity: 0;

    transform:
      translateX(35px)
      scale(0.92);
  }
}

.task-animation-move {
  transition:
    transform 0.4s ease;
}

/* ========================================= */
/* EMPTY ANIMATION */
/* ========================================= */

.empty-animation-enter-active,
.empty-animation-leave-active {
  transition:
    opacity 0.3s ease,
    transform 0.3s ease;
}

.empty-animation-enter-from,
.empty-animation-leave-to {
  opacity: 0;

  transform:
    translateY(10px);
}

/* ========================================= */
/* DETAIL ANIMATION */
/* ========================================= */

.detail-animation-enter-active {
  animation:
    detailOpen 0.35s ease;
}

.detail-animation-leave-active {
  animation:
    detailClose 0.25s ease forwards;
}

@keyframes detailOpen {
  from {
    opacity: 0;

    transform:
      translateX(15px)
      scale(0.98);
  }

  to {
    opacity: 1;

    transform:
      translateX(0)
      scale(1);
  }
}

@keyframes detailClose {
  from {
    opacity: 1;

    transform:
      translateX(0)
      scale(1);
  }

  to {
    opacity: 0;

    transform:
      translateX(12px)
      scale(0.98);
  }
}

/* ========================================= */
/* RESPONSIVE */
/* ========================================= */

@media (max-width: 800px) {
  .app-container {
    max-width: 680px;
  }

  .add-form {
    grid-template-columns:
      1fr 1fr;
  }

  .content.detail-open {
    grid-template-columns: 1fr;
  }

  .detail-panel {
    order: -1;
  }
}

@media (max-width: 600px) {
  .page {
    padding: 0;
  }

  .app-container {
    min-height: 100vh;

    padding: 18px 14px;

    border-radius: 0;
  }

  .header {
    align-items: flex-start;
    flex-direction: column;
  }

  .mode-switch {
    align-self: flex-end;
  }

  .brand h1 {
    font-size: 25px;
  }

  .add-form {
    grid-template-columns: 1fr;
  }

  .filter-section {
    align-items: flex-start;
    flex-direction: column;
  }

  .counter {
    align-self: flex-end;
  }

  .task-meta {
    align-items: flex-start;
    flex-direction: column;

    gap: 5px;
  }

  .task-title-area h3 {
    white-space: normal;
  }
}
</style>