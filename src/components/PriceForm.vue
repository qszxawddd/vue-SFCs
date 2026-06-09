<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  drinkMenu: {
    type: Array,
    required: true,
  },
  message: {
    type: Object,
    default: null,
  },
})

const emit = defineEmits(['add-record'])

const today = new Date().toISOString().slice(0, 10)
const date = ref(today)
const drinkName = ref('')
const price = ref('')
const localError = ref('')

const canSubmit = computed(() => date.value && drinkName.value && price.value !== '')

function submitForm() {
  localError.value = ''
  const parsedPrice = Number(price.value)

  if (!date.value || !drinkName.value || price.value === '') {
    localError.value = '請填寫日期、飲品與價格。'
    return
  }

  if (!Number.isFinite(parsedPrice) || parsedPrice < 0) {
    localError.value = '價格必須是 0 以上的數字。'
    return
  }

  emit('add-record', {
    date: date.value,
    drinkName: drinkName.value,
    price: parsedPrice,
  })

  drinkName.value = ''
  price.value = ''
  date.value = today
}
</script>

<template>
  <section class="panel">
    <div class="panel-heading">
      <p>新增紀錄</p>
      <h2>記下這次購買價格</h2>
    </div>

    <div v-if="localError" class="message error">{{ localError }}</div>
    <div v-else-if="props.message" class="message" :class="props.message.type">
      {{ props.message.text }}
    </div>

    <form class="price-form" @submit.prevent="submitForm">
      <label>
        日期
        <input v-model="date" type="date" required />
      </label>

      <label>
        飲品
        <select v-model="drinkName" required>
          <option value="">請選擇飲品</option>
          <option v-for="drink in drinkMenu" :key="drink" :value="drink">
            {{ drink }}
          </option>
        </select>
      </label>

      <label>
        價格
        <input v-model="price" type="number" min="0" step="0.01" placeholder="例如 150" required />
      </label>

      <button class="primary-button" type="submit" :disabled="!canSubmit">新增</button>
    </form>
  </section>
</template>

<style scoped>
.panel {
  border: 1px solid #d9d0bf;
  border-radius: 8px;
  background: #ffffff;
  padding: 20px;
  box-shadow: 0 18px 42px rgba(29, 36, 33, 0.08);
}

.panel-heading p {
  margin: 0 0 6px;
  color: #00754a;
  font-size: 13px;
  font-weight: 800;
}

.panel-heading h2 {
  margin: 0 0 18px;
  color: #12372a;
  font-size: 22px;
}

.price-form {
  display: grid;
  gap: 14px;
}

label {
  display: grid;
  gap: 7px;
  color: #2d3833;
  font-size: 14px;
  font-weight: 700;
}

input,
select {
  width: 100%;
  border: 1px solid #cfc7b8;
  border-radius: 8px;
  background: #fffdf8;
  padding: 11px 12px;
  color: #1d2421;
}

input:focus,
select:focus {
  border-color: #00754a;
  outline: 3px solid rgba(0, 117, 74, 0.14);
}

.primary-button {
  min-height: 44px;
  border: 0;
  border-radius: 8px;
  background: #00754a;
  color: white;
  font-weight: 800;
}

.message {
  margin-bottom: 14px;
  border: 1px solid transparent;
  border-radius: 8px;
  padding: 11px 12px;
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
</style>
