<template>
  <div class="status-panel">
    <h2>Status do Jogador</h2>
    <ul>
      <li><b>Vida:</b> <span class="status-value">{{ Math.round(life) }}</span></li>
      <li><b>Escudo:</b> <span class="status-value">{{ Math.round(shield) }}</span></li>
      <li><b>Dano:</b> <span class="status-value">{{ Math.round(damage) }}</span></li>
      <li><b>Velocidade:</b> <span class="status-value">{{ Math.round(speed) }}</span></li>
      <li><b>Posição:</b> <span class="status-value">({{ Math.round(x) }}, {{ Math.round(y) }})</span></li>
      <li><b>Ping:</b> <span class="status-value">{{ ms }} ms</span></li>
      <li><b>Tamanho do Universo:</b> <span class="status-value">{{ universeSize }}</span></li>
      <li><b>Última Atualização:</b> <span class="status-value">{{ formatTime(lastUpdate) }}</span></li> 
    </ul>
  </div>
</template>

<script setup>
import { computed } from 'vue';

// Define props received from the parent component (App.vue)
const props = defineProps({
  life: Number,
  shield: Number,
  damage: Number,
  speed: Number,
  x: Number,
  y: Number,
  ms: Number,
  universeSize: Number,
  lastUpdate: Number // Add lastUpdate as a prop
})

// Helper function to format timestamp for display
const formatTime = (timestamp) => {
    if (!timestamp) return 'N/A';
    const date = new Date(timestamp);
    return date.toLocaleTimeString('pt-BR'); // Adjust locale as needed
};
</script>

<style scoped>
.status-panel {
  background: #181c24;
  color: #fff;
  border-radius: 8px;
  padding: 1.5rem 2.5rem; /* Increased padding */
  margin-bottom: 1rem;
  box-shadow: 0 4px 16px rgba(0,0,0,0.5); /* Stronger shadow */
  border: 1px solid #333; /* Subtle border */
}
h2 {
  color: #6ee7b7; /* Highlight heading color */
  margin-bottom: 1rem;
  font-size: 1.4rem;
  text-align: center;
}
ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
li {
  margin-bottom: 0.6rem; /* More space between list items */
  display: flex; /* Use flexbox for label-value alignment */
  justify-content: space-between; /* Push value to the right */
  align-items: center;
  font-size: 1.1rem;
}
b {
  color: #a7a7a7; /* Lighter color for labels */
  font-weight: normal; /* Less bold for labels */
}
.status-value {
  color: #fff;
  font-weight: bold; /* Bold for values */
  text-shadow: 0 0 2px rgba(0,0,0,0.3); /* Subtle shadow for values */
}
</style>

---

### Key Points for Dynamic UI Updates

1.  **Reactive State (`status` object in `App.vue`):** This is your single source of truth for the player's dynamic data. When you update it using `Object.assign(status, data)`, Vue automatically detects these changes.
2.  **Props (`defineProps` in child components):** Your `GameHUD` and `GameStatus` components receive the `status` properties as props. When the `status` object in `App.vue` changes, Vue efficiently re-renders only the necessary parts of these child components.
3.  **WebSocket `onmessage` Listener:** This is the core handler. Every time your server sends data, this function parses it and updates the `status` object. This is what drives all UI changes.
4.  **Data Consistency:** Ensure that the data types and names sent from your WebSocket server (`data.type`, `data.life`, `data.x`, etc.) match what your client-side `status` object and props expect.

With these refined components, your application will smoothly update its interface in real-time as events stream from your WebSocket server!

Do you want to add any other features or visual elements to your game's UI?