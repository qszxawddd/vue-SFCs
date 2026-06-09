<script setup>
defineProps({
  summary: {
    type: Object,
    required: true,
  },
})

function formatPrice(value) {
  return `$${Number(value).toFixed(2)}`
}

function formatChange(amount, percent) {
  const sign = amount >= 0 ? '+' : ''
  return `${sign}${formatPrice(amount)} (${sign}${percent.toFixed(1)}%)`
}
</script>

<template>
  <section class="summary-card" aria-label="飲品價格統計">
    <div class="summary-title">
      <p>統計摘要</p>
      <h3>{{ summary.drinkName }}</h3>
    </div>

    <dl>
      <div>
        <dt>價格範圍</dt>
        <dd>{{ formatPrice(summary.minPrice) }} - {{ formatPrice(summary.maxPrice) }}</dd>
      </div>
      <div>
        <dt>總漲跌</dt>
        <dd :class="{ positive: summary.totalChange > 0, negative: summary.totalChange < 0 }">
          {{ formatChange(summary.totalChange, summary.totalChangePercent) }}
        </dd>
      </div>
      <div>
        <dt>紀錄期間</dt>
        <dd>{{ summary.firstDate }} 到 {{ summary.lastDate }}</dd>
      </div>
      <div>
        <dt>紀錄筆數</dt>
        <dd>{{ summary.recordCount }} 筆</dd>
      </div>
    </dl>
  </section>
</template>

<style scoped>
.summary-card {
  display: grid;
  grid-template-columns: minmax(180px, 0.8fr) minmax(0, 1.8fr);
  gap: 16px;
  margin: 14px 0;
  border: 1px solid #bbd8c9;
  border-radius: 8px;
  background: #f0faf5;
  padding: 16px;
}

.summary-title p {
  margin: 0 0 6px;
  color: #00754a;
  font-size: 13px;
  font-weight: 800;
}

.summary-title h3 {
  margin: 0;
  color: #12372a;
  font-size: 22px;
}

dl {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
  margin: 0;
}

dt {
  color: #617069;
  font-size: 12px;
  font-weight: 800;
}

dd {
  overflow-wrap: anywhere;
  margin: 4px 0 0;
  color: #1d2421;
  font-size: 15px;
  font-weight: 800;
}

.positive {
  color: #b63b28;
}

.negative {
  color: #0f7752;
}

@media (max-width: 720px) {
  .summary-card,
  dl {
    grid-template-columns: 1fr;
  }
}
</style>
