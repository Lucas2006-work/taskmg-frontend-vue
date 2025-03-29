<template>
  <div v-if="isOpen" class="modal-overlay">
    <div class="modal-content">
      <h2 class="modal-title">New Task</h2>
      
      <!-- Title Input -->
      <input 
        type="text" 
        v-model="title" 
        class="form-control modal-input" 
        placeholder="Enter task title" 
      />
      
      <!-- Description Input -->
      <textarea 
        v-model="description" 
        class="form-control modal-input" 
        placeholder="Enter task description" 
        rows="4"
      ></textarea>
      
      <div class="modal-actions">
        <button @click="confirm" class="btn btn-primary">OK</button>
        <button @click="cancel" class="btn btn-secondary">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script>
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

export default {
  props: ['isOpen'],
  emits: ['confirm', 'cancel'],
  data() {
    return {
      title: '',
      description: ''
    }
  },
  methods: {
    confirm() {
      if(this.title == '' || this.description == '') {
        toast.error("Please input contents.", {
          autoClose: 2000,
        });
        return
      }
      this.$emit('confirm', { title: this.title, description: this.description });
      this.clearInputs();
    },
    cancel() {
      this.$emit('cancel');
      this.clearInputs();
    },
    clearInputs() {
      this.title = '';
      this.description = '';
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.4); /* Semi-transparent background */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 2rem;
  border-radius: 8px;
  width: 400px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.modal-title {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 1.5rem;
  text-align: center;
}

.modal-input {
  width: 100%;
  padding: 0.8rem;
  margin-bottom: 1rem;
  font-size: 1rem;
  border-radius: 8px;
  border: 1px solid #ccc;
  outline: none;
  transition: border-color 0.2s ease;
}

.modal-input:focus {
  border-color: #007bff;
}

textarea.modal-input {
  resize: none; /* Disable textarea resizing */
}

.modal-actions {
  width: 100%;
  display: flex;
  justify-content: space-between;
}

.btn {
  padding: 0.8rem 1.2rem;
  font-size: 1rem;
  font-weight: 600;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  width: 48%;
}

.btn-primary {
  background-color: #007bff;
  color: white;
  border: none;
}

.btn-primary:hover {
  background-color: #0056b3;
}

.btn-secondary {
  background-color: #f0f0f0;
  color: #333;
  border: 1px solid #ccc;
}

.btn-secondary:hover {
  background-color: #e2e2e2;
}
</style>
