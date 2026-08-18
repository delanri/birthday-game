<script setup>
import { ref, reactive, onMounted } from 'vue'

const props = defineProps({
  colorful: { type: Boolean, default: false }
})

const emit = defineEmits(['start'])

// ══ stagger 入场动画 ══
const show = reactive({
  bg: false,
  cake: false,
  title: false,
  buttons: false,
  footer: false,
})

onMounted(() => {
  requestAnimationFrame(() => {
    show.bg = true
    setTimeout(() => (show.cake = true), 300)
    setTimeout(() => (show.title = true), 700)
    setTimeout(() => (show.buttons = true), 1200)
    setTimeout(() => (show.footer = true), 1600)
  })
})

// ══ 退出按钮 ══
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
  setTimeout(() => (exitShaking.value = false), 500)
  setTimeout(() => (exitMsg.value = ''), 2000)
}

// ══ 按钮点击粒子 ══
const clickParticles = ref([])
let pid = 0

function onStart(e) {
  const rect = e.currentTarget.getBoundingClientRect()
  const symbols = ['✦', '◆', '★', '♡', '◇', '▪', '✿', '❋']
  const batch = []
  for (let i = 0; i < 14; i++) {
    const angle = (Math.PI * 2 * i) / 14 + (Math.random() - 0.5) * 0.4
    const dist = 35 + Math.random() * 55
    batch.push({
      id: pid++,
      symbol: symbols[i % symbols.length],
      x: rect.width / 2,
      y: rect.height / 2,
      dx: Math.cos(angle) * dist,
      dy: Math.sin(angle) * dist,
    })
  }
  clickParticles.value = batch
  setTimeout(() => {
    clickParticles.value = []
    emit('start')
  }, 550)
}

// ══ 浮动背景粒子 ══
const floaters = (() => {
  const arr = []
  const syms = ['✦', '◇', '✿', '▪', '★', '♡', '◆', '❋', '•', '⬥']
  const hues = [340, 30, 120, 200, 270, 50, 160, 300, 0, 210]
  for (let i = 0; i < 22; i++) {
    arr.push({
      sym: syms[i % syms.length],
      left: (Math.random() * 90 + 5).toFixed(1),
      delay: (Math.random() * 12).toFixed(1),
      dur: (10 + Math.random() * 10).toFixed(1),
      size: (8 + Math.random() * 10).toFixed(0),
      baseOpacity: (0.12 + Math.random() * 0.18).toFixed(2),
      hue: hues[i % hues.length],
    })
  }
  return arr
})()
</script>

<template>
  <div class="start-page">

    <!-- ══ 浮动背景粒子 ══ -->
    <div class="floating-layer" :class="{ visible: show.bg }">
      <span
        v-for="(f, i) in floaters"
        :key="i"
        class="floater"
        :class="{ 'floater--colorful': colorful }"
        :style="{
          left: f.left + '%',
          fontSize: f.size + 'px',
          animationDelay: f.delay + 's',
          animationDuration: f.dur + 's',
          '--base-opacity': f.baseOpacity,
          '--hue': f.hue,
        }"
      >{{ f.sym }}</span>
    </div>

    <div class="start-content">

      <!-- ══ 像素小蛋糕（掉落弹跳入场）══ -->
      <div
        class="pixel-cake"
        :class="{
          'drop-in': show.cake,
          'white': !colorful,
          'cake-float': show.cake && colorful,
        }"
      >
        <div class="cake-top">{{ colorful ? '🎂' : '🎁' }}</div>
      </div>

      <!-- ══ 标题框（展开入场 + 四角装饰 + shimmer）══ -->
      <div
        class="title-frame pixel-box"
        :class="{ 'expand-in': show.title }"
      >
        <!-- 四角像素装饰 -->
        <span class="corner corner-tl"></span>
        <span class="corner corner-tr"></span>
        <span class="corner corner-bl"></span>
        <span class="corner corner-br"></span>

        <!-- 标题 + shimmer 光扫 -->
        <div class="title-shimmer-wrap">
          <h1 class="title">回忆杂货铺₍ᐢ..ᐢ₎</h1>
        </div>

        <p class="subtitle">这里收藏着勇者们曾经的冒险故事</p>
        <div class="pixel-divider"></div>
        <p class="desc">
          {{ colorful
            ? '欢迎勇者大人回家！蛋糕已经准备好了~'
            : '店门还开着，要进来看看吗？૮₍ ◜ᵕ◝ ₎ა'
          }}
        </p>
      </div>

      <!-- ══ 按钮区（弹上来入场）══ -->
      <div class="btn-group" :class="{ 'pop-up': show.buttons }">

        <!-- 进入按钮 + 点击粒子容器 -->
        <div class="start-btn-wrap">
          <button class="pixel-btn" @click="onStart">
            {{ colorful ? '再次进入' : '推门进入' }}
          </button>
          <span
            v-for="p in clickParticles"
            :key="p.id"
            class="click-particle"
            :style="{
              left: p.x + 'px',
              top: p.y + 'px',
              '--dx': p.dx + 'px',
              '--dy': p.dy + 'px',
            }"
          >{{ p.symbol }}</span>
        </div>

        <!-- 退出按钮 -->
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

      <!-- ══ 底部小字 ══ -->
      <p class="footer-text" :class="{ 'fade-in-up': show.footer }">
        {{ colorful
          ? '货架上的宝藏都找到啦 ♡'
          : '每一份物品都在货架上等你୧₍ᐢ·͈༝·͈ᐢ₎୨'
        }}
      </p>
    </div>
  </div>
</template>

<style scoped>
/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   页面容器
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.start-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 24px 16px;
  position: relative;
  overflow: hidden;
}

.start-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 360px;
  width: 100%;
  gap: 24px;
  position: relative;
  z-index: 1;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   1. 浮动背景粒子
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.floating-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  opacity: 0;
  transition: opacity 1.5s ease;
}

.floating-layer.visible {
  opacity: 1;
}

.floater {
  position: absolute;
  bottom: -20px;
  opacity: 0;
  color: var(--border, #ccc);
  animation: floatUp linear infinite;
  animation-fill-mode: both;
  filter: blur(0.3px);
  transition: color 1s ease, filter 1s ease;
}

.floater--colorful {
  color: hsl(var(--hue, 0), 70%, 72%);
  filter: blur(0px);
}

@keyframes floatUp {
  0% {
    transform: translateY(0) translateX(0) rotate(0deg);
    opacity: 0;
  }
  8% {
    opacity: var(--base-opacity, 0.15);
  }
  92% {
    opacity: var(--base-opacity, 0.15);
  }
  100% {
    transform: translateY(-110vh) translateX(15px) rotate(180deg);
    opacity: 0;
  }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   2. 像素小蛋糕 —— 掉落弹跳入场
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.pixel-cake {
  font-size: 48px;
  margin-bottom: 8px;
  transition: filter 0.8s ease;
  opacity: 0;
  transform: translateY(-80px);
}

.pixel-cake.white {
  filter: saturate(0.4) brightness(1.15);
}

.pixel-cake.drop-in {
  animation: dropBounce 0.7s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

.pixel-cake.cake-float {
  animation: dropBounce 0.7s cubic-bezier(0.34, 1.56, 0.64, 1) forwards,
             gentleFloat 3s ease-in-out 0.8s infinite;
}

@keyframes dropBounce {
  0%   { opacity: 0; transform: translateY(-80px) scale(0.5); }
  55%  { opacity: 1; transform: translateY(10px) scale(1.1); }
  75%  { transform: translateY(-5px) scale(0.97); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
}

@keyframes gentleFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-6px); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   3. 标题框 —— 展开入场
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.title-frame {
  width: 100%;
  padding: 28px 24px;
  text-align: center;
  position: relative;
  opacity: 0;
  transform: scale(0.3, 0.5);
}

.title-frame.expand-in {
  animation: expandIn 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

@keyframes expandIn {
  0%   { opacity: 0; transform: scale(0.3, 0.5); }
  50%  { opacity: 1; transform: scale(1.03, 1.02); }
  100% { opacity: 1; transform: scale(1, 1); }
}

/* ── 四角像素装饰 ── */
.corner {
  position: absolute;
  width: 10px;
  height: 10px;
  pointer-events: none;
  opacity: 0.5;
  transition: border-color 0.5s ease, opacity 0.5s ease;
}

.corner-tl {
  top: -1px;
  left: -1px;
  border-top: 3px solid var(--accent, #aaa);
  border-left: 3px solid var(--accent, #aaa);
}

.corner-tr {
  top: -1px;
  right: -1px;
  border-top: 3px solid var(--accent, #aaa);
  border-right: 3px solid var(--accent, #aaa);
}

.corner-bl {
  bottom: -1px;
  left: -1px;
  border-bottom: 3px solid var(--accent, #aaa);
  border-left: 3px solid var(--accent, #aaa);
}

.corner-br {
  bottom: -1px;
  right: -1px;
  border-bottom: 3px solid var(--accent, #aaa);
  border-right: 3px solid var(--accent, #aaa);
}

/* ── 标题 shimmer 光扫 ── */
.title-shimmer-wrap {
  position: relative;
  display: inline-block;
  overflow: hidden;
}

.title-shimmer-wrap::after {
  content: '';
  position: absolute;
  top: 0;
  left: -80%;
  width: 50%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(255, 255, 255, 0.35) 40%,
    rgba(255, 255, 255, 0.55) 50%,
    rgba(255, 255, 255, 0.35) 60%,
    transparent 100%
  );
  transform: skewX(-15deg);
  animation: shimmerSweep 4s ease-in-out 1.5s infinite;
}

@keyframes shimmerSweep {
  0%, 100% { left: -80%; }
  50%      { left: 130%; }
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

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   4. 按钮区 —— 弹上来入场 + 点击粒子
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.btn-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  opacity: 0;
  transform: translateY(30px);
}

.btn-group.pop-up {
  animation: popUp 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

@keyframes popUp {
  0%   { opacity: 0; transform: translateY(30px); }
  70%  { opacity: 1; transform: translateY(-4px); }
  100% { opacity: 1; transform: translateY(0); }
}

/* ── 粒子容器 ── */
.start-btn-wrap {
  position: relative;
  overflow: visible;
}

.click-particle {
  position: absolute;
  font-size: 11px;
  pointer-events: none;
  color: var(--accent, #e88);
  animation: particleBurst 0.5s cubic-bezier(0.22, 0.61, 0.36, 1) forwards;
  z-index: 10;
}

@keyframes particleBurst {
  0% {
    transform: translate(-50%, -50%) scale(1.3);
    opacity: 1;
  }
  100% {
    transform: translate(
      calc(-50% + var(--dx, 0px)),
      calc(-50% + var(--dy, 0px))
    ) scale(0);
    opacity: 0;
  }
}

/* ── 退出按钮 ── */
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

.msg-enter-active {
  animation: msgPop 0.25s ease;
}

.msg-leave-active {
  transition: opacity 0.3s ease;
}

.msg-leave-to {
  opacity: 0;
}

@keyframes msgPop {
  from { opacity: 0; transform: translateX(-50%) translateY(4px); }
  to   { opacity: 1; transform: translateX(-50%) translateY(0); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   5. 底部文字淡入
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.footer-text {
  font-size: 12px;
  color: var(--text-light);
  letter-spacing: 1px;
  opacity: 0;
  transform: translateY(10px);
}

.footer-text.fade-in-up {
  animation: fadeInUp 0.6s ease forwards;
}

@keyframes fadeInUp {
  0%   { opacity: 0; transform: translateY(10px); }
  100% { opacity: 0.6; transform: translateY(0); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   通用：shake
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.shake {
  animation: shakeIt 0.4s ease;
}

@keyframes shakeIt {
  0%, 100% { transform: translateX(0); }
  20%      { transform: translateX(-4px); }
  40%      { transform: translateX(4px); }
  60%      { transform: translateX(-3px); }
  80%      { transform: translateX(3px); }
}
</style>