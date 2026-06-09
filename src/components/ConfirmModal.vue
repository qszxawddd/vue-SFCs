<script setup>
defineProps({
  title: {
    type: String,
    required: true,
  },
  message: {
    type: String,
    required: true,
  },
  confirmLabel: {
    type: String,
    default: '確認',
  },
})

const emit = defineEmits(['cancel', 'confirm'])
</script>

<template>
  <div class="modal-backdrop" role="presentation" @click.self="emit('cancel')">
    <section class="modal" role="dialog" aria-modal="true" aria-labelledby="confirm-title">
      <h2 id="confirm-title">{{ title }}</h2>
      <p>{{ message }}</p>
      <div class="modal-actions">
        <button type="button" class="secondary-button" @click="emit('cancel')">取消</button>
        <button type="button" class="danger-button" @click="emit('confirm')">
          {{ confirmLabel }}
        </button>
      </div>
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
  width: min(420px, 100%);
  border-radius: 8px;
  background: #ffffff;
  padding: 22px;
  box-shadow: 0 24px 70px rgba(0, 0, 0, 0.24);
}

h2 {
  margin: 0 0 12px;
  color: #12372a;
  font-size: 22px;
}

p {
  margin: 0;
  color: #56645d;
  line-height: 1.7;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 22px;
}

button {
  min-height: 42px;
  border-radius: 8px;
  padding: 0 16px;
  font-weight: 800;
}

.secondary-button {
  border: 1px solid #b9ad99;
  background: #fffdf8;
  color: #3d463f;
}

.danger-button {
  border: 0;
  background: #b63b28;
  color: white;
}
</style>
