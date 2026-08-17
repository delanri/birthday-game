<script setup>
import { ref } from 'vue'

const props = defineProps({
  colorful: { type: Boolean, default: false }
})

const emit = defineEmits(['start'])

// ── 退出按钮 ──
const exitShaking = ref(false)
const exitMsg = ref('')

const exitMessages = [
  '店门已经锁了🔒',
  '老板说不买不准走！',
  '逃跑失败 ❌ 货架挡住了去路',
  '路过也要进来坐坐嘛~',
  '勇者不可以临阵脱逃！',
]

function onExit() {
  if (exitShaking.value) return
  exitShaking.value = true
  exitMsg.value = exitMessages[Math.floor(Math.random() * exitMessages.length)]
  setTimeout(() => {
    exitShaking.value = false
  }, 500)
  // 消息停留久一点再消失
  setTimeout(() => {
    exitMsg.value = ''
  }, 2000)
}
</script>

<template>
  <div class="start-page">
    <div class="start-content fade-in">
      <!-- 像素装饰：小蛋糕 -->
      <div class="pixel-cake" :class="{ 'bounce': !colorful, 'float': colorful, 'white': !colorful }">
        <div class="cake-top">{{ colorful ? '🎂' : '🎁' }}</div>
      </div>

      <!-- 标题框 -->
      <div class="title-frame pixel-box">
        <h1 class="title">回忆杂货铺₍ᐢ..ᐢ₎</h1>
        <p class="subtitle">这里收藏着勇者们曾经的冒险故事</p>
        <div class="pixel-divider"></div>
        <p class="desc">
          {{ colorful ? '欢迎勇者大人回家！蛋糕已经准备好了~' : '店门还开着，要进来看看吗？૮₍ ◜ᵕ◝ ₎ა' }}
        </p>
      </div>

      <!-- 按钮区 -->
      <div class="btn-group">
        <button class="pixel-btn" @click="emit('start')">
          {{ colorful ? '再次进入' : '推门进入' }}
        </button>

        <div class="exit-wrapper">
          <button
            class="pixel-btn pixel-btn--ghost"
            :class="{ shake: exitShaking }"
            @click="onExit"
          >
            只是路过
          </button>
          <Transition name="msg">
            <div v-if="exitMsg" class="exit-msg pixel-box">
              {{ exitMsg }}
            </div>
          </Transition>
        </div>
      </div>

      <!-- 底部小字 -->
      <p class="footer-text">
        {{ colorful ? '货架上的宝藏都找到啦 ♡' : '每一份物品都在货架上等你୧₍ᐢ·͈༝·͈ᐢ₎୨' }}
      </p>
    </div>
  </div>
</template>

<style scoped>
.start-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 24px 16px;
}

.start-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 360px;
  width: 100%;
  gap: 24px;
}

/* ── 像素小蛋糕 ── */
.pixel-cake {
  font-size: 48px;
  margin-bottom: 8px;
  transition: filter 0.8s ease;
}

.pixel-cake.white {
  filter: saturate(0.4) brightness(1.15);
}
/* ── 标题框 ── */
.title-frame {
  width: 100%;
  padding: 28px 24px;
  text-align: center;
}

.title {
  font-size: 24px;
  font-weight: 700;
  color: var(--text);
  letter-spacing: 4px;
  margin-bottom: 8px;
}

.subtitle {
  font-size: 13px;
  color: var(--text-light);
  letter-spacing: 2px;
}

.pixel-divider {
  width: 60px;
  height: 3px;
  background: var(--border);
  margin: 16px auto;
}

.desc {
  font-size: 14px;
  color: var(--text-light);
  line-height: 1.6;
}

/* ── 按钮区 ── */
.btn-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.exit-wrapper {
  position: relative;
}

.exit-msg {
  position: absolute;
  top: -44px;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  padding: 6px 14px;
  font-size: 12px;
  font-weight: 600;
  color: var(--accent);
  background: #fff;
  border-color: var(--accent);
  z-index: 10;
}

/* ── 消息动画 ── */
.msg-enter-active {
  animation: msgPop 0.25s ease;
}
.msg-leave-active { transition: opacity 0.3s ease; }
.msg-leave-to { opacity: 0; }

@keyframes msgPop {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* ── 底部小字 ── */
.footer-text {
  font-size: 12px;
  color: var(--text-light);
  letter-spacing: 1px;
  opacity: 0.6;
}
</style>