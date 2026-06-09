<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  drinkMenu: {
    type: Array,
    required: true,
  },
  selectedDrink: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['search', 'reset'])
const localDrink = ref(props.selectedDrink)

watch(
  () => props.selectedDrink,
  (nextDrink) => {
    localDrink.value = nextDrink
  },
)

function search() {
  emit('search', localDrink.value)
}

function reset() {
  localDrink.value = ''
  emit('reset')
}
</script>

<template>
  <section class="search-panel">
    <div>
      <p>查詢價格</p>
      <h2>選擇飲品查看漲跌</h2>
    </div>

    <div class="search-controls">
      <label>
        飲品
        <select v-model="localDrink">
          <option value="">請選擇飲品</option>
          <option v-for="drink in drinkMenu" :key="drink" :value="drink">
            {{ drink }}
          </option>
        </select>
      </label>

      <div class="button-row">
        <button type="button" class="primary-button" @click="search">查詢</button>
        <button type="button" class="secondary-button" @click="reset">清除</button>
      </div>
    </div>
  </section>
</template>

<style scoped>
.search-panel {
  display: grid;
  grid-template-columns: minmax(220px, 1fr) minmax(260px, 1.2fr);
  gap: 16px;
  align-items: end;
  border: 1px solid #d9d0bf;
  border-radius: 8px;
  background: #ffffff;
  padding: 18px;
}

p {
  margin: 0 0 6px;
  color: #00754a;
  font-size: 13px;
  font-weight: 800;
}

h2 {
  margin: 0;
  color: #12372a;
  font-size: 22px;
}

.search-controls {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  gap: 12px;
  align-items: end;
}

label {
  display: grid;
  gap: 7px;
  color: #2d3833;
  font-size: 14px;
  font-weight: 700;
}

select {
  width: 100%;
  border: 1px solid #cfc7b8;
  border-radius: 8px;
  background: #fffdf8;
  padding: 11px 12px;
  color: #1d2421;
}

select:focus {
  border-color: #00754a;
  outline: 3px solid rgba(0, 117, 74, 0.14);
}

.button-row {
  display: flex;
  gap: 8px;
}

button {
  min-height: 44px;
  border: 0;
  border-radius: 8px;
  padding: 0 16px;
  font-weight: 800;
}

.primary-button {
  background: #00754a;
  color: white;
}

.secondary-button {
  border: 1px solid #b9ad99;
  background: #fffdf8;
  color: #3d463f;
}

@media (max-width: 720px) {
  .search-panel,
  .search-controls {
    grid-template-columns: 1fr;
  }

  .button-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
</style>
