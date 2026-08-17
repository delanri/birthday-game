<script setup>
import { ref, onMounted } from 'vue'

const emit = defineEmits(['done'])

const lines = [
  { text: '在一个小小的像素世界里——', delay: 0 },
  { text: '散落着属于你们的冒险记忆', delay: 800 },
  { text: '', delay: 1400 },
  { text: '🕯️ 光遇的星空', delay: 1800, color: '#89B4D4' },
  { text: '🍃 原神的旅途', delay: 2400, color: '#8CB868' },
  { text: '🌻 星露谷的田园', delay: 3000, color: '#D898B0' },
  { text: '🍳 胡闹厨房的欢笑', delay: 3600, color: '#D0B870' },
  { text: '', delay: 4200 },
  { text: '翻开卡片，找回这些珍贵的瞬间吧！', delay: 4600 },
]

const visibleCount = ref(0)
const showButton = ref(false)

onMounted(() => {
  lines.forEach((line, i) => {
    setTimeout(() => {
      visibleCount.value = i + 1
    }, line.delay)
  })
  // 最后一行出现后显示按钮
  setTimeout(() => {
    showButton.value = true
  }, lines[lines.length - 1].delay + 600)
})
</script>

<template>
  <div class="story-page">
    <div class="story-content">
      <div class="story-lines">
        <div
          v-for="(line, i) in lines"
          :key="i"
          class="story-line"
          :class="{ 'visible': i < visibleCount }"
          :style="{
            color: line.color || 'var(--text)',
            transitionDelay: '0s',
            minHeight: line.text ? 'auto' : '16px',
          }"
        >
          {{ line.text }}
        </div>
      </div>

      <Transition name="btn">
        <button
          v-if="showButton"
          class="pixel-btn"
          @click="emit('done')"
        >
          准备好了！
        </button>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.story-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 24px 16px;
}

.story-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 340px;
  width: 100%;
  gap: 32px;
}

.story-lines {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

.story-line {
  font-size: 15px;
  line-height: 1.8;
  letter-spacing: 1px;
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.6s ease, transform 0.6s ease;
  text-align: center;
}

.story-line.visible {
  opacity: 1;
  transform: translateY(0);
}

/* ── 按钮动画 ── */
.btn-enter-active { animation: popIn 0.4s ease; }
</style>