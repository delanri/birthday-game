<script setup>
import { ref, onMounted } from 'vue'

const emit = defineEmits(['back'])

const lines = [
  { text: '🎂', size: '48px', weight: 700, spacing: '0', mb: '16px' },
  { text: '生日快乐！', size: '28px', weight: 700, spacing: '4px', mb: '24px' },
  { text: '在这个特别的日子里', size: '15px', weight: 400, spacing: '1px', mb: '8px' },
  { text: '希望你永远被美好的回忆包围', size: '15px', weight: 400, spacing: '1px', mb: '8px' },
  { text: '每一段冒险都有人陪在身边', size: '15px', weight: 400, spacing: '1px', mb: '24px' },
  { text: '—— 来自你的朋友 ♡', size: '14px', weight: 600, spacing: '2px', mb: '0' },
]

const visibleCount = ref(0)
const showBackBtn = ref(false)

onMounted(() => {
  lines.forEach((_, i) => {
    setTimeout(() => {
      visibleCount.value = i + 1
    }, 800 + i * 700)
  })
  setTimeout(() => {
    showBackBtn.value = true
  }, 800 + lines.length * 700 + 500)
})
</script>

<template>
  <div class="birthday-page">
    <div class="birthday-content">
      <div
        v-for="(line, i) in lines"
        :key="i"
        class="birthday-line"
        :class="{ visible: i < visibleCount }"
        :style="{
          fontSize: line.size,
          fontWeight: line.weight,
          letterSpacing: line.spacing,
          marginBottom: line.mb,
          color: i === 0 ? '#F0A0B0' : '#ffffff',
        }"
      >
        {{ line.text }}
      </div>

      <Transition name="fade">
        <button
          v-if="showBackBtn"
          class="back-btn"
          @click="emit('back')"
        >
          回到起点 🎂
        </button>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.birthday-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: #000;
  padding: 40px 24px;
}

.birthday-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.birthday-line {
  opacity: 0;
  transform: translateY(12px);
  transition: opacity 0.8s ease, transform 0.8s ease;
  line-height: 1.6;
}

.birthday-line.visible {
  opacity: 1;
  transform: translateY(0);
}

.back-btn {
  margin-top: 40px;
  padding: 10px 24px;
  border: 2px solid #F0A0B0;
  background: transparent;
  color: #F0A0B0;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  font-family: inherit;
  letter-spacing: 2px;
  transition: background 0.3s, color 0.3s;
}

.back-btn:hover {
  background: #F0A0B0;
  color: #000;
}

.fade-enter-active { transition: opacity 0.8s ease; }
.fade-enter-from { opacity: 0; }
</style>