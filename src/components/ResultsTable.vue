<script setup>
defineProps({
  records: {
    type: Array,
    required: true,
  },
  selectedDrink: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['edit', 'delete'])

function formatPrice(value) {
  return `$${Number(value).toFixed(2)}`
}

function formatDelta(amount, percent) {
  if (amount === null || amount === undefined) return '-'
  const sign = amount >= 0 ? '+' : ''
  return `${sign}${formatPrice(amount)} (${sign}${Number(percent).toFixed(1)}%)`
}
</script>

<template>
  <section class="table-panel">
    <div v-if="!selectedDrink" class="empty-state">
      <strong>請先選擇飲品</strong>
      <p>選擇飲品後，這裡會列出每次購買價格與漲跌幅。</p>
    </div>

    <div v-else-if="records.length === 0" class="empty-state">
      <strong>沒有價格紀錄</strong>
      <p>可以從左側新增 {{ selectedDrink }} 的第一筆資料。</p>
    </div>

    <div v-else class="table-scroll">
      <table>
        <thead>
          <tr>
            <th>日期</th>
            <th>價格</th>
            <th>單次漲跌</th>
            <th>累積漲跌</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="record in records" :key="record.id">
            <td>{{ record.date }}</td>
            <td class="price-cell">{{ formatPrice(record.price) }}</td>
            <td
              :class="{
                positive: record.changeFromPrevious > 0,
                negative: record.changeFromPrevious < 0,
              }"
            >
              {{ formatDelta(record.changeFromPrevious, record.changeFromPreviousPercent) }}
            </td>
            <td
              :class="{
                positive: record.changeFromFirst > 0,
                negative: record.changeFromFirst < 0,
              }"
            >
              {{ formatDelta(record.changeFromFirst, record.changeFromFirstPercent) }}
            </td>
            <td>
              <div class="action-row">
                <button type="button" class="edit-button" @click="emit('edit', record)">編輯</button>
                <button type="button" class="delete-button" @click="emit('delete', record)">刪除</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>

<style scoped>
.table-panel {
  overflow: hidden;
  border: 1px solid #d9d0bf;
  border-radius: 8px;
  background: #ffffff;
}

.table-scroll {
  overflow-x: auto;
}

table {
  width: 100%;
  min-width: 690px;
  border-collapse: collapse;
  font-size: 14px;
}

thead {
  background: #f5f1e8;
}

th,
td {
  border-bottom: 1px solid #ece3d5;
  padding: 13px 14px;
  text-align: left;
  vertical-align: middle;
}

th {
  color: #56645d;
  font-size: 12px;
  font-weight: 900;
}

tbody tr:hover {
  background: #fffaf0;
}

.price-cell {
  color: #00754a;
  font-weight: 900;
}

.positive {
  color: #b63b28;
  font-weight: 800;
}

.negative {
  color: #0f7752;
  font-weight: 800;
}

.action-row {
  display: flex;
  gap: 8px;
}

button {
  min-height: 34px;
  border: 0;
  border-radius: 8px;
  padding: 0 12px;
  color: white;
  font-size: 13px;
  font-weight: 800;
}

.edit-button {
  background: #5c6f68;
}

.delete-button {
  background: #b63b28;
}

.empty-state {
  padding: 34px 20px;
  text-align: center;
}

.empty-state strong {
  display: block;
  color: #12372a;
  font-size: 20px;
}

.empty-state p {
  margin: 10px auto 0;
  max-width: 420px;
  color: #68736e;
  line-height: 1.7;
}
</style>
