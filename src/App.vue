<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import StartPage from './components/StartPage.vue'
import StoryPage from './components/StoryPage.vue'
import GameBoard from './components/GameBoard.vue'
import BirthdayEnding from './components/BirthdayEnding.vue'


// ── 页面状态 ──
const currentPage = ref('start')
const gameComplete = ref(false)

// ── 颜色阶段 ──
// 0=纯白(开始) 1=微暖(故事) 2=浅色(第一轮) 3=彩色(第二轮) 4=五彩蛋糕(完成)
const colorStage = ref(0)
const pageFading = ref(false)

watch(colorStage, (stage) => {
  const icon = stage >= 4 ? '💝' : '🤍'
  const link = document.querySelector("link[rel='icon']")
    || document.createElement('link')
  link.rel = 'icon'
  link.href = `data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text x='50' y='55' text-anchor='middle' dominant-baseline='central' font-size='80'>${icon}</text></svg>`
  document.head.appendChild(link)
}, { immediate: true })

const colorClass = computed(() => `color-stage-${colorStage.value}`)

// ── 页面跳转 ──
function goToStory() {
  colorStage.value = 1
  currentPage.value = 'story'
}

function goToGame() {
  colorStage.value = 2
  currentPage.value = 'game'
  preloadPhotos()
}

function goToBirthday() {
  pageFading.value = true
  setTimeout(() => {
    currentPage.value = 'birthday'
    setTimeout(() => {
      pageFading.value = false
    }, 100)
  }, 500)
}

function backToStart() {
  gameComplete.value = true
  colorStage.value = 4
  currentPage.value = 'start'
  saveProgress()
}

// ── 缓存 ──
onMounted(() => {
  const saved = localStorage.getItem('birthday-game')
  if (saved) {
    const data = JSON.parse(saved)
    gameComplete.value = data.gameComplete || false
    colorStage.value = data.colorStage || 0
  }
  preloadCards()
})

function saveProgress() {
  localStorage.setItem('birthday-game', JSON.stringify({
    gameComplete: gameComplete.value,
    colorStage: colorStage.value,
  }))
}

function preloadCards() {
  const cards = import.meta.glob('./assets/cards/*.png', { eager: true })
  Object.values(cards).forEach(mod => {
    const img = new Image()
    img.src = mod.default
  })
}

function preloadPhotos() {
  const photos = import.meta.glob('./assets/photos/*.jpg', { eager: true })
  Object.values(photos).forEach(mod => {
    const img = new Image()
    img.src = mod.default
  })
}

</script>

<template>
  <div id="app" :class="colorClass" :style="{
    background: colorStage === 4
      ? 'linear-gradient(135deg, #FFE8E8 0%, #FFF0D0 25%, #E8FFE8 50%, #E0F0FF 75%, #F0E8FF 100%)'
      : 'var(--bg)',
    minHeight: '100vh',
    transition: 'background 0.8s ease',
  }">
    <StartPage
      v-if="currentPage === 'start'"
      :colorful="gameComplete"
      @start="goToStory"
    />
    <StoryPage
      v-if="currentPage === 'story'"
      @done="goToGame"
    />
    <GameBoard
      v-if="currentPage === 'game'"
      :color-stage="colorStage"
      @round-change="colorStage = 3"
      @complete="goToBirthday"
    />
    <BirthdayEnding
      v-if="currentPage === 'birthday'"
      @back="backToStart"
    />
   <div class="page-fade" :class="{ active: pageFading }"></div>
  </div>
</template>

<style>
.page-fade {
  position: fixed;
  inset: 0;
  background: #FFFCF8;
  opacity: 0;
  pointer-events: none;
  z-index: 999;
  transition: opacity 0.5s ease;
}
.page-fade.active {
  opacity: 1;
}
</style>