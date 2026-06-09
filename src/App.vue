<script setup>
import { computed, onMounted, ref, watch } from 'vue'
import ConfirmModal from './components/ConfirmModal.vue'
import EditModal from './components/EditModal.vue'
import PriceForm from './components/PriceForm.vue'
import ResultsTable from './components/ResultsTable.vue'
import SearchPanel from './components/SearchPanel.vue'
import SummaryCard from './components/SummaryCard.vue'

const STORAGE_KEY = 'starbucks-price-records-vue'

const drinkMenu = [
  'Tall Americano',
  'Grande Americano',
  'Venti Americano',
  'Tall Latte',
  'Grande Latte',
  'Venti Latte',
  'Tall Cappuccino',
  'Grande Cappuccino',
  'Venti Cappuccino',
  'Tall Mocha',
  'Grande Mocha',
  'Tall Macchiato',
  'Grande Macchiato',
  'Caramel Frappuccino',
  'Matcha Tea Latte',
]

const starterRecords = [
  { id: 1, date: '2024-05-14', drinkName: 'Grande Latte', price: 145 },
  { id: 2, date: '2024-08-10', drinkName: 'Grande Latte', price: 150 },
  { id: 3, date: '2025-03-08', drinkName: 'Grande Latte', price: 160 },
  { id: 4, date: '2024-06-02', drinkName: 'Caramel Frappuccino', price: 170 },
  { id: 5, date: '2025-01-18', drinkName: 'Caramel Frappuccino', price: 180 },
  { id: 6, date: '2024-09-21', drinkName: 'Tall Americano', price: 95 },
]

const records = ref([])
const selectedDrink = ref('Grande Latte')
const addMessage = ref(null)
const resultsMessage = ref(null)
const editingRecord = ref(null)
const deletingRecord = ref(null)
const hasLoaded = ref(false)

const filteredRecords = computed(() => {
  if (!selectedDrink.value) return []

  return records.value
    .filter((record) => record.drinkName === selectedDrink.value)
    .slice()
    .sort((a, b) => {
      const dateResult = new Date(a.date) - new Date(b.date)
      return dateResult || a.id - b.id
    })
    .map((record, index, list) => {
      const firstRecord = list[0]
      const previousRecord = list[index - 1]

      return {
        ...record,
        changeFromPrevious:
          previousRecord === undefined ? null : roundMoney(record.price - previousRecord.price),
        changeFromPreviousPercent:
          previousRecord === undefined
            ? null
            : calculatePercent(record.price - previousRecord.price, previousRecord.price),
        changeFromFirst: index === 0 ? null : roundMoney(record.price - firstRecord.price),
        changeFromFirstPercent:
          index === 0 ? null : calculatePercent(record.price - firstRecord.price, firstRecord.price),
      }
    })
})

const summary = computed(() => {
  if (!selectedDrink.value || filteredRecords.value.length === 0) return null

  const prices = filteredRecords.value.map((record) => record.price)
  const firstRecord = filteredRecords.value[0]
  const lastRecord = filteredRecords.value[filteredRecords.value.length - 1]
  const totalChange = roundMoney(lastRecord.price - firstRecord.price)

  return {
    drinkName: selectedDrink.value,
    minPrice: Math.min(...prices),
    maxPrice: Math.max(...prices),
    totalChange,
    totalChangePercent: calculatePercent(totalChange, firstRecord.price),
    recordCount: filteredRecords.value.length,
    firstDate: firstRecord.date,
    lastDate: lastRecord.date,
  }
})

const totalRecords = computed(() => records.value.length)

onMounted(() => {
  records.value = loadRecords()
  hasLoaded.value = true
})

watch(
  records,
  (nextRecords) => {
    if (!hasLoaded.value) return
    localStorage.setItem(STORAGE_KEY, JSON.stringify(nextRecords))
  },
  { deep: true },
)

function loadRecords() {
  const rawRecords = localStorage.getItem(STORAGE_KEY)
  if (!rawRecords) return starterRecords

  try {
    const parsedRecords = JSON.parse(rawRecords)
    if (!Array.isArray(parsedRecords)) return starterRecords

    return parsedRecords
      .filter(
        (record) =>
          Number.isFinite(Number(record.id)) &&
          typeof record.date === 'string' &&
          typeof record.drinkName === 'string' &&
          Number.isFinite(Number(record.price)),
      )
      .map((record) => ({
        id: Number(record.id),
        date: record.date,
        drinkName: record.drinkName,
        price: Number(record.price),
      }))
  } catch {
    return starterRecords
  }
}

function addRecord(payload) {
  records.value.push({
    id: createNextId(),
    date: payload.date,
    drinkName: payload.drinkName,
    price: normalizePrice(payload.price),
  })

  selectedDrink.value = payload.drinkName
  addMessage.value = { type: 'success', text: `已新增 ${payload.drinkName} 的價格紀錄。` }
  resultsMessage.value = null
}

function searchDrink(drinkName) {
  selectedDrink.value = drinkName

  if (!drinkName) {
    resultsMessage.value = { type: 'error', text: '請先選擇要查詢的飲品。' }
    return
  }

  resultsMessage.value =
    filteredRecords.value.length === 0
      ? { type: 'error', text: `目前沒有 ${drinkName} 的價格紀錄。` }
      : null
}

function resetSearch() {
  selectedDrink.value = ''
  resultsMessage.value = null
}

function openEdit(record) {
  editingRecord.value = { ...record }
}

function updateRecord(payload) {
  const index = records.value.findIndex((record) => record.id === payload.id)
  if (index === -1) return

  records.value[index] = {
    id: payload.id,
    date: payload.date,
    drinkName: payload.drinkName,
    price: normalizePrice(payload.price),
  }

  selectedDrink.value = payload.drinkName
  editingRecord.value = null
  resultsMessage.value = { type: 'success', text: '紀錄已更新。' }
}

function openDelete(record) {
  deletingRecord.value = record
}

function deleteRecord() {
  if (!deletingRecord.value) return

  records.value = records.value.filter((record) => record.id !== deletingRecord.value.id)
  resultsMessage.value = { type: 'success', text: '紀錄已刪除。' }
  deletingRecord.value = null
}

function createNextId() {
  return records.value.length === 0 ? 1 : Math.max(...records.value.map((record) => record.id)) + 1
}

function normalizePrice(price) {
  return roundMoney(Number(price))
}

function roundMoney(value) {
  return Math.round((Number(value) + Number.EPSILON) * 100) / 100
}

function calculatePercent(change, base) {
  if (!base) return 0
  return Number(((change / base) * 100).toFixed(1))
}
</script>

<template>
  <main class="app-shell">
    <section class="hero">
      <div class="brand-mark" aria-hidden="true">
        <span>SB</span>
      </div>
      <div>
        <p class="eyebrow">Vue SFC Assignment 5</p>
        <h1>Starbucks 飲品價格追蹤器</h1>

      </div>
    </section>

    <section class="dashboard-stats" aria-label="目前資料摘要">
      <div>
        <span>飲品選單</span>
        <strong>{{ drinkMenu.length }}</strong>
      </div>
      <div>
        <span>全部紀錄</span>
        <strong>{{ totalRecords }}</strong>
      </div>
      <div>
        <span>目前查詢</span>
        <strong>{{ selectedDrink || '未選擇' }}</strong>
      </div>
    </section>

    <section class="workspace">
      <PriceForm :drink-menu="drinkMenu" :message="addMessage" @add-record="addRecord" />

      <div class="results-panel">
        <SearchPanel
          :drink-menu="drinkMenu"
          :selected-drink="selectedDrink"
          @search="searchDrink"
          @reset="resetSearch"
        />
        <div v-if="resultsMessage" class="message" :class="resultsMessage.type">
          {{ resultsMessage.text }}
        </div>
        <SummaryCard v-if="summary" :summary="summary" />
        <ResultsTable
          :records="filteredRecords"
          :selected-drink="selectedDrink"
          @edit="openEdit"
          @delete="openDelete"
        />
      </div>
    </section>

    <EditModal
      v-if="editingRecord"
      :record="editingRecord"
      :drink-menu="drinkMenu"
      @close="editingRecord = null"
      @save="updateRecord"
    />

    <ConfirmModal
      v-if="deletingRecord"
      title="刪除價格紀錄"
      :message="`確定要刪除 ${deletingRecord.drinkName} 在 ${deletingRecord.date} 的紀錄嗎？`"
      confirm-label="刪除"
      @cancel="deletingRecord = null"
      @confirm="deleteRecord"
    />
  </main>
</template>

<style>
:root {
  color: #1d2421;
  background: #f5f1e8;
  font-family:
    Inter, 'Noto Sans TC', 'Microsoft JhengHei', system-ui, -apple-system, BlinkMacSystemFont,
    'Segoe UI', sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button,
input,
select {
  font: inherit;
}

button {
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

#app {
  min-height: 100vh;
}
</style>

<style scoped>
.app-shell {
  width: min(1120px, calc(100% - 32px));
  margin: 0 auto;
  padding: 32px 0;
}

.hero {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
  margin-bottom: 22px;
}

.brand-mark {
  display: grid;
  width: 86px;
  height: 86px;
  place-items: center;
  border: 8px solid #00754a;
  border-radius: 50%;
  background: #ffffff;
  color: #00754a;
  font-size: 24px;
  font-weight: 800;
  letter-spacing: 0;
  box-shadow: 0 12px 32px rgba(29, 36, 33, 0.14);
}

.eyebrow {
  margin: 0 0 8px;
  color: #7a4f2a;
  font-size: 13px;
  font-weight: 700;
  letter-spacing: 0;
  text-transform: uppercase;
}

h1 {
  margin: 0;
  color: #12372a;
  font-size: clamp(32px, 6vw, 54px);
  line-height: 1.05;
}

.subtitle {
  max-width: 680px;
  margin: 12px 0 0;
  color: #56645d;
  font-size: 16px;
  line-height: 1.7;
}

.dashboard-stats {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  margin-bottom: 18px;
}

.dashboard-stats div {
  min-width: 0;
  border: 1px solid #d9d0bf;
  border-radius: 8px;
  background: #fffaf0;
  padding: 14px 16px;
}

.dashboard-stats span {
  display: block;
  color: #6b746f;
  font-size: 13px;
  font-weight: 700;
}

.dashboard-stats strong {
  display: block;
  overflow-wrap: anywhere;
  margin-top: 6px;
  color: #12372a;
  font-size: 24px;
}

.workspace {
  display: grid;
  grid-template-columns: minmax(280px, 360px) minmax(0, 1fr);
  gap: 18px;
  align-items: start;
}

.results-panel {
  min-width: 0;
}

.message {
  margin: 14px 0;
  border: 1px solid transparent;
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 14px;
  font-weight: 700;
}

.message.success {
  border-color: #b6dbc8;
  background: #e6f4ec;
  color: #0f5f3d;
}

.message.error {
  border-color: #e5b6ad;
  background: #fff0ed;
  color: #a23120;
}

@media (max-width: 820px) {
  .app-shell {
    width: min(100% - 24px, 680px);
    padding: 24px 0;
  }

  .hero,
  .workspace {
    grid-template-columns: 1fr;
  }

  .brand-mark {
    width: 72px;
    height: 72px;
  }

  .dashboard-stats {
    grid-template-columns: 1fr;
  }
}
</style>
