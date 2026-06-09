<script setup>
import { reactive, ref } from 'vue'

const props = defineProps({
  record: {
    type: Object,
    required: true,
  },
  drinkMenu: {
    type: Array,
    required: true,
  },
})

const emit = defineEmits(['close', 'save'])
const error = ref('')
const form = reactive({
  id: props.record.id,
  date: props.record.date,
  drinkName: props.record.drinkName,
  price: props.record.price,
})

function save() {
  error.value = ''
  const parsedPrice = Number(form.price)

  if (!form.date || !form.drinkName || form.price === '') {
    error.value = '請填寫日期、飲品與價格。'
    return
  }

  if (!Number.isFinite(parsedPrice) || parsedPrice < 0) {
    error.value = '價格必須是 0 以上的數字。'
    return
  }

  emit('save', {
    id: form.id,
    date: form.date,
    drinkName: form.drinkName,
    price: parsedPrice,
  })
}
</script>

<template>
  <div class="modal-backdrop" role="presentation" @click.self="emit('close')">
    <section class="modal" role="dialog" aria-modal="true" aria-labelledby="edit-title">
      <h2 id="edit-title">編輯價格紀錄</h2>
      <div v-if="error" class="message error">{{ error }}</div>

      <form class="edit-form" @submit.prevent="save">
        <label>
          日期
          <input v-model="form.date" type="date" required />
        </label>

        <label>
          飲品
          <select v-model="form.drinkName" required>
            <option value="">請選擇飲品</option>
            <option v-for="drink in drinkMenu" :key="drink" :value="drink">
              {{ drink }}
            </option>
          </select>
        </label>

        <label>
          價格
          <input v-model="form.price" type="number" min="0" step="0.01" required />
        </label>

        <div class="modal-actions">
          <button type="button" class="secondary-button" @click="emit('close')">取消</button>
          <button type="submit" class="primary-button">儲存</button>
        </div>
      </form>
    </section>
  </div>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  background: rgba(18, 55, 42, 0.42);
  padding: 18px;
  z-index: 20;
}

.modal {
  width: min(440px, 100%);
  border-radius: 8px;
  background: #ffffff;
  padding: 22px;
  box-shadow: 0 24px 70px rgba(0, 0, 0, 0.24);
}

h2 {
  margin: 0 0 18px;
  color: #12372a;
  font-size: 22px;
}

.edit-form {
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

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 4px;
}

button {
  min-height: 42px;
  border-radius: 8px;
  padding: 0 16px;
  font-weight: 800;
}

.primary-button {
  border: 0;
  background: #00754a;
  color: white;
}

.secondary-button {
  border: 1px solid #b9ad99;
  background: #fffdf8;
  color: #3d463f;
}

.message {
  margin-bottom: 14px;
  border: 1px solid transparent;
  border-radius: 8px;
  padding: 11px 12px;
  font-size: 14px;
  font-weight: 700;
}

.message.error {
  border-color: #e5b6ad;
  background: #fff0ed;
  color: #a23120;
}
</style>
