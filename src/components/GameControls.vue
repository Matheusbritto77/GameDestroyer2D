<template>
  <div class="controls">
    <button @click="$emit('move', 0, -1)" title="Cima">⬆️</button>
    <div>
      <button @click="$emit('move', -1, 0)" title="Esquerda">⬅️</button>
      <button @click="$emit('move', 1, 0)" title="Direita">➡️</button>
    </div>
    <button @click="$emit('move', 0, 1)" title="Baixo">⬇️</button>
  </div>
</template>

<script setup>
defineEmits(['move'])
</script>

<style scoped>
.controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 1rem;
}
.controls > div {
  display: flex;
  gap: 1rem;
}
button {
  background: #23272f;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 0.7rem 1.2rem;
  font-size: 1.5rem;
  margin: 0.2rem;
  cursor: pointer;
  transition: background 0.2s;
}
button:hover {
  background: #3a4052;
}
</style> 