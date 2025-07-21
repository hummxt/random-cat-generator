<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const API_KEY = 'live_Q59dTsQbMf7xeiHaEEsfKluZKqKbVbIRHshzPw0n0TpeFxFTWi4d10999QvcpafE'
const catUrl = ref('')
const isLoading = ref(false)

async function fetchCat() {
  isLoading.value = true
  const res = await fetch('https://api.thecatapi.com/v1/images/search', {
    headers: { 'x-api-key': API_KEY },
  })
  const data = await res.json()
  catUrl.value = data[0]?.url || ''
  isLoading.value = false
}

onMounted(() => {
  const card = document.querySelector('.cat') as HTMLElement

  const baseHoverScale = 1.2
  let scale = baseHoverScale
  const minScale = baseHoverScale
  const maxScale = 2.5

  let isHovered = false
  let lastRotateX = 0
  let lastRotateY = 0

  function updateTransform(rotateX: number, rotateY: number, scaleVal: number) {
    card.style.transform = `rotateX(${rotateX}deg) rotateY(${rotateY}deg) scale(${scaleVal})`
  }

  function handleMouseEnter() {
    isHovered = true
    scale = baseHoverScale
    updateTransform(lastRotateX, lastRotateY, scale)
  }

  function handleMouseLeave() {
    isHovered = false
    scale = 1
    lastRotateX = 0
    lastRotateY = 0
    updateTransform(0, 0, scale)
  }

  function handleMouseMove(e: MouseEvent) {
    if (!isHovered) return

    const rect = card.getBoundingClientRect()
    const x = e.clientX - rect.left
    const y = e.clientY - rect.top

    const centerX = rect.width / 2
    const centerY = rect.height / 2

    lastRotateX = ((y - centerY) / centerY) * 25
    lastRotateY = ((x - centerX) / centerX) * -25

    updateTransform(lastRotateX, lastRotateY, scale)
  }

  function handleWheel(e: WheelEvent) {
    if (!isHovered) return

    e.preventDefault()
    if (e.deltaY < 0) {
      scale += 0.1
    } else {
      scale -= 0.1
    }
    scale = Math.min(maxScale, Math.max(minScale, scale))
    updateTransform(lastRotateX, lastRotateY, scale)
  }

  card.addEventListener('mouseenter', handleMouseEnter)
  card.addEventListener('mouseleave', handleMouseLeave)
  card.addEventListener('mousemove', handleMouseMove)
  card.addEventListener('wheel', handleWheel, { passive: false })

  onUnmounted(() => {
    card.removeEventListener('mouseenter', handleMouseEnter)
    card.removeEventListener('mouseleave', handleMouseLeave)
    card.removeEventListener('mousemove', handleMouseMove)
    card.removeEventListener('wheel', handleWheel)
  })
})

</script>

<template>
  <main>
    <div class="cat">
      <img v-if="catUrl" :src="catUrl" alt="Cat image" />
      <p v-else-if="isLoading">Loading cat...</p>
      <p v-else>Click the button to see a cat</p>
    </div>
    <button class="btn" @click="fetchCat">
      Show me a cat!
    </button>
  </main>
</template>

<style scoped>
main {
  height: 86vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  perspective: 1000px;
}

.cat {
  width: 400px;
  height: 400px;
  background: rgba(255, 255, 255, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  margin-bottom: 24px;
  text-align: center;
  font-size: 18px;
  color: #333;
  transition: transform 0.15s ease;
  transform-style: preserve-3d;
  z-index: 20;
}

.cat img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  pointer-events: none;
}

.btn {
  cursor: pointer;
  font-size: 20px;
  font-weight: 600;
  padding: 20px 24px;
  background: rgba(255, 255, 255, 0.3);
  border: none;
  border-radius: 12px;
  backdrop-filter: blur(6px);
  transition: all 0.3s ease;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  z-index: 10;
}

.btn:hover {
  background: rgba(255, 255, 255, 0.6);
  transform: scale(1.05);
}
</style>
