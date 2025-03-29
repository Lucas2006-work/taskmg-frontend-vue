<!-- ModalDialog.vue -->
<script setup>
  const emit = defineEmits(['confirm', 'cancel']);
</script>

<template>
  <div class="modal-overlay">
    <div class="modal-container">
      <div class="modal-header">
        <h2 class="modal-title">Are you sure?</h2>
      </div>

      <div class="modal-body">
        <p class="modal-description">Please confirm your action.</p>
      </div>

      <div class="modal-footer">
        <button class="btn btn-cancel" @click="emit('cancel')">Cancel</button>
        <button class="btn btn-confirm" @click="emit('confirm')">Confirm</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

/* Modal Container */
.modal-container {
  background-color: #fff;
  border-radius: 8px;
  width: 400px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

/* Modal Header */
.modal-header {
  text-align: center;
}

.modal-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

/* Modal Body */
.modal-body {
  text-align: center;
  font-size: 1rem;
  color: #555;
  margin-bottom: 20px;
}

/* Modal Footer */
.modal-footer {
  display: flex;
  justify-content: space-between;
  gap: 10px;
}

button {
  width: 45%;
  padding: 12px 20px;
  font-size: 1rem;
  font-weight: 500;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

button:hover {
  transform: scale(1.05);
}

.btn-cancel {
  background-color: #f44336; /* Red for cancel */
  color: white;
}

.btn-confirm {
  background-color: #4caf50; /* Green for confirm */
  color: white;
}

/* Add focus styles for accessibility */
button:focus {
  outline: 3px solid #1976d2;
  outline-offset: 2px;
}
</style>
