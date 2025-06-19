<template>
  <div class="connection-panel">
    <form @submit.prevent="connect">
      <input v-model="username" placeholder="Seu nome ou nick" required />
      <button type="submit" :disabled="connected">Conectar</button>
      <button v-if="connected" @click.prevent="disconnect">Desconectar</button>
    </form>
    <div v-if="connected" class="connected">Conectado como <b>{{ username }}</b></div>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const emit = defineEmits(['connect', 'disconnect'])
const username = ref('')
const connected = ref(false)
const error = ref('')

function connect() {
  if (!username.value) return
  connected.value = true
  error.value = ''
  emit('connect', username.value)
}
function disconnect() {
  connected.value = false
  emit('disconnect')
}

// Permite resetar conexão pelo pai
watch(connected, (val) => {
  if (!val) username.value = ''
})
</script>

<style scoped>
.connection-panel {
  background: #23272f;
  color: #fff;
  border-radius: 8px;
  padding: 1rem 2rem;
  margin-bottom: 1rem;
  box-shadow: 0 2px 8px #0003;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
input {
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  margin-right: 1rem;
  font-size: 1rem;
}
button {
  background: #3a4052;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1.2rem;
  margin-right: 0.5rem;
  cursor: pointer;
  transition: background 0.2s;
}
button:disabled {
  background: #555a6a;
  cursor: not-allowed;
}
.connected {
  margin-top: 0.5rem;
  color: #6ee7b7;
}
.error {
  margin-top: 0.5rem;
  color: #f87171;
}
</style> 