<script setup>
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue'
import GameStatus from './components/GameStatus.vue'
import GameConnection from './components/GameConnection.vue'
import GameUniverse from './components/GameUniverse.vue'
import GameHUD from './components/GameHUD.vue'

// Reactive WebSocket instance
const ws = ref(null)
// Connection status
const connected = ref(false)
// Player username
const username = ref('')
// Reactive object to hold all player status data
const status = reactive({
  life: 0,
  shield: 0,
  damage: 0,
  speed: 0,
  x: 0,
  y: 0,
  ms: 0, // Milliseconds for ping
  universeSize: 0,
  lastUpdate: 0 // Timestamp of the last update
})
// Reactive error message
const error = ref('')

// Função dedicada para tratar eventos do WebSocket
function handleWSEvent(event) {
  try {
    const data = JSON.parse(event.data)
    console.log('[WS EVENTO]', data)
    if (data.type === 'update' || data.type === 'status') {
      Object.assign(status, data)
      status.lastUpdate = Date.now()
    } else if (data.type === 'pong') {
      status.ms = data.ms
    } else if (data.error) {
      error.value = data.error
    }
    // Outros tipos de eventos podem ser tratados aqui
  } catch (e) {
    console.error('Erro ao processar mensagem do servidor:', e, event.data)
    error.value = 'Erro ao processar mensagem do servidor.'
  }
}

/**
 * Establishes a WebSocket connection.
 * @param {string} name - The username to connect with.
 */
function connectWS(name) {
  if (ws.value) ws.value.close()
  ws.value = new WebSocket('wss://wppapi-socket-mgkurw-0189c1-168-231-95-211.traefik.me')
  ws.value.onopen = () => {
    connected.value = true
    username.value = name
    error.value = ''
    console.log('[WS] Conectado ao servidor.')
    ws.value.send(JSON.stringify({ type: 'get_status', username: name }))
  }
  ws.value.addEventListener('message', handleWSEvent)
  ws.value.onerror = (err) => {
    console.error('[WS ERRO]', err)
    error.value = 'Erro de conexão com o servidor. Verifique o console para detalhes.'
    connected.value = false
  }
  ws.value.onclose = (event) => {
    console.log('[WS] Conexão fechada:', event.code, event.reason)
    connected.value = false
    ws.value = null
    username.value = ''
    if (!event.wasClean) {
      error.value = 'Conexão encerrada inesperadamente.'
    } else if (event.code !== 1000) {
      error.value = `Conexão fechada: ${event.code} - ${event.reason || 'Desconhecido'}`
    }
  }
}

/**
 * Disconnects from the WebSocket.
 */
function disconnectWS() {
  if (ws.value) {
    ws.value.removeEventListener('message', handleWSEvent)
    ws.value.close(1000, 'User initiated disconnect')
    console.log('[WS] Desconectando...')
  }
  connected.value = false
  username.value = ''
  error.value = ''
}

/**
 * Sends a move command to the server.
 * @param {number} dx - Change in X-coordinate.
 * @param {number} dy - Change in Y-coordinate.
 */
function move(dx, dy) {
  if (ws.value && connected.value) {
    ws.value.send(JSON.stringify({ type: 'move', dx, dy }))
  }
}

/**
 * Sends a ping command to the server to measure latency.
 */
function ping() {
  if (ws.value && connected.value) {
    ws.value.send(JSON.stringify({ type: 'ping' }))
  }
}

/**
 * Handles keyboard input for movement.
 * @param {KeyboardEvent} e - The keyboard event.
 */
function handleKey(e) {
  if (!connected.value) return // Only handle keys if connected

  // Prevent default scroll behavior for arrow keys if game handles movement
  if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.key)) {
    e.preventDefault() 
  }

  if (e.key === 'ArrowUp') move(0, -1)
  else if (e.key === 'ArrowDown') move(0, 1)
  else if (e.key === 'ArrowLeft') move(-1, 0)
  else if (e.key === 'ArrowRight') move(1, 0)
  // You can add more key bindings for other actions like 'attack', 'use_item', etc.
}

// Lifecycle hooks
onMounted(() => {
  window.addEventListener('keydown', handleKey)
  // Optionally, you can try to reconnect or prompt for username on mount
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKey)
  disconnectWS() // Ensure disconnection on component unmount
})
</script>

<template>
  <div class="container fullscreen">
    <GameHUD 
      :life="status.life" 
      :shield="status.shield" 
      :speed="status.speed" 
      :damage="status.damage" 
      :ms="status.ms" 
    />

    <GameConnection @connect="connectWS" @disconnect="disconnectWS" :isConnected="connected" :currentUsername="username" />
    
    <GameUniverse v-if="connected" :x="status.x" :y="status.y" :universeSize="status.universeSize" :onMove="move" />
    
    <div v-if="connected" class="hud-panel">
      <GameStatus v-bind="status" />
    </div>
    
    <div v-if="error" class="error">{{ error }}</div>
    
    <footer>
      <small>Frontend Vue 3 • WebSocket seguro • Modular</small>
    </footer>
  </div>
</template>

<style scoped>
.container.fullscreen {
  width: 100vw;
  height: 100vh;
  min-height: 100vh;
  max-width: 100vw;
  margin: 0;
  padding: 0;
  background: transparent;
  border-radius: 0;
  box-shadow: none;
  position: relative;
  overflow: hidden;
  display: flex; /* Use flexbox for better layout control */
  flex-direction: column; /* Stack elements vertically */
  align-items: center; /* Center content horizontally */
}
.hud-panel {
  position: fixed;
  right: 2vw;
  top: 6vh;
  z-index: 10;
  width: 320px;
  max-width: 90vw;
  display: flex; /* Enable flexbox for inner elements */
  flex-direction: column; /* Stack them vertically */
  gap: 1rem; /* Space between status panel and ping button */
}
.ping-btn {
  background: #6ee7b7;
  color: #181c24;
  border: none;
  border-radius: 6px;
  padding: 0.6rem 1.5rem;
  font-size: 1.1rem;
  margin-bottom: 1rem;
  cursor: pointer;
  transition: background 0.2s;
  width: 100%;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2); /* Added subtle shadow */
}
.ping-btn:hover {
  background: #34d399;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3); /* Enhanced shadow on hover */
}
.error {
  color: #f87171;
  background: rgba(255, 0, 0, 0.1); /* Light red background for errors */
  padding: 0.5rem 1rem;
  border-radius: 5px;
  margin-top: 1rem;
  text-align: center;
  z-index: 20;
  position: fixed;
  left: 0; right: 0;
  top: 0;
  font-weight: bold; /* Make error message more prominent */
}
footer {
  text-align: center;
  margin-top: 2rem;
  color: #aaa;
  z-index: 20;
  position: fixed;
  left: 0; right: 0;
  bottom: 0;
  padding: 1rem; /* Add padding for better visibility */
  background: rgba(24,28,36,0.5); /* Semi-transparent background */
  box-shadow: 0 -2px 8px rgba(0,0,0,0.3); /* Shadow for footer */
}
</style>