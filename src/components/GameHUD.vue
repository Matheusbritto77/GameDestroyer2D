<template>
  <div class="hud-bar">
    <div class="hud-item">
      <span class="icon">❤️</span>
      <div class="bar life" :style="{width: life + '%'}"></div>
      <span class="value">{{ Math.round(life) }}</span> </div>
    <div class="hud-item">
      <span class="icon">🛡️</span>
      <div class="bar shield" :style="{width: shield + '%'}"></div>
      <span class="value">{{ Math.round(shield) }}</span> </div>
    <div class="hud-item hud-text-item">
      <span class="icon">⚡</span>
      <span class="value">{{ Math.round(speed) }}</span>
    </div>
    <div class="hud-item hud-text-item">
      <span class="icon">💥</span>
      <span class="value">{{ Math.round(damage) }}</span>
    </div>
    <div class="hud-item hud-text-item">
      <span class="icon">📶</span>
      <span class="value">{{ ms }} ms</span>
    </div>
  </div>
</template>

<script setup>
// Define props received from the parent component (App.vue)
defineProps({
  life: Number,
  shield: Number,
  speed: Number,
  damage: Number,
  ms: Number
})
</script>

<style scoped>
.hud-bar {
  position: fixed;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  z-index: 30; /* Ensure it's above other elements */
  display: flex;
  gap: 1.5rem; /* Space between HUD items */
  background: rgba(24,28,36,0.9); /* Slightly more opaque background */
  border-radius: 0 0 16px 16px;
  padding: 0.5rem 2.5rem 0.5rem 2.5rem;
  box-shadow: 0 2px 16px rgba(0,0,0,0.5); /* Stronger shadow */
  min-width: 380px; /* Increased min-width for better spacing */
  max-width: 95vw; /* Allow it to be wider on larger screens */
  align-items: center;
  justify-content: center; /* Center items within the bar */
}
.hud-item {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  min-width: 70px; /* Increased min-width for consistency */
}
.hud-text-item {
  justify-content: flex-start; /* Align text-only items to the start */
}
.icon {
  font-size: 1.5rem; /* Slightly larger icons */
}
.bar {
  height: 14px; /* Slightly taller bars */
  border-radius: 7px; /* Adjust border-radius to match height */
  min-width: 40px;
  max-width: 80px;
  margin: 0 0.3rem;
  background: #23272f; /* Background for empty bar portion */
  box-shadow: inset 0 1px 3px rgba(0,0,0,0.4); /* Inner shadow for depth */
  transition: width 0.2s ease-out; /* Smoother transition */
  overflow: hidden; /* Ensure gradient stays within bounds */
}
.bar.life {
  background: linear-gradient(90deg, #f87171, #fbbf24); /* Vibrant gradient */
}
.bar.shield {
  background: linear-gradient(90deg, #60a5fa, #6ee7b7); /* Vibrant gradient */
}
.value {
  font-size: 1.2rem; /* Slightly larger value text */
  min-width: 28px; /* Ensure enough space for values */
  text-align: right;
  color: #fff;
  font-weight: bold; /* Make values stand out */
  text-shadow: 0 0 3px rgba(0,0,0,0.5); /* Subtle text shadow */
}
</style>