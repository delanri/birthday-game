<script setup>
import { ref, onMounted, nextTick } from 'vue'

const emit = defineEmits(['done'])

// ══ 故事脚本 ══
const sections = [
  {
    light: 0.06,
    decoration: null,
    lines: [
      '这间铺子很小。',
      '刚进门时灯也不太亮。',
      '但货架上的东西都还在。',
    ],
  },
  {
    light: 0.15,
    decoration: 'candle',
    lines: [
      '角落里有一盏没点亮的蜡烛——',
      '高高的影子站成一排，有人跑在最前面忘了说话。',
    ],
  },
  {
    light: 0.25,
    decoration: 'water',
    lines: [
      '旁边搁着一截旧竹竿——',
      '隔了好久又回到水面上，竹筏晃晃悠悠的，谁也没急着靠岸。',
    ],
  },
  {
    light: 0.38,
    decoration: 'petals',
    lines: [
      '第二层货架上放着一小束干花——',
      '那一天，方块草地上开满花，谁都没有缺席。',
    ],
  },
  {
    light: 0.5,
    decoration: 'receipt',
    lines: [
      '最底下压着一张沾着油污的收据——',
      '厨房里乱成一团，但盘子全端出去了，分数比谁都没想到的高。',
    ],
  },
  {
    light: 0.7,
    decoration: null,
    lines: [
      '灰扑扑的货架上，每一样都在等人来领。',
      '这次的「影子」，会是他们的主人吗？',
    ],
  },
]

// ══ 状态 ══
const overlayOpacity = ref(0)
const lightOpacity = ref(0)

const displayedLines = ref([])
const currentText = ref('')
const showCursor = ref(false)
const showButton = ref(false)
const typingLineRef = ref(null)
const activeDecoration = ref(null)

// 推门
const contentFading = ref(false)
const doorOpen = ref(false)

// ══ 花瓣数据 ══
const petals = (() => {
  const arr = []
  const syms = ['❀', '✿', '❁', '·', '✾', '❀', '✿', '·']
  for (let i = 0; i < 8; i++) {
    arr.push({
      sym: syms[i],
      left: (8 + Math.random() * 84).toFixed(0),
      delay: (Math.random() * 6).toFixed(1),
      dur: (5 + Math.random() * 5).toFixed(1),
      size: (8 + Math.random() * 6).toFixed(0),
    })
  }
  return arr
})()

// ══ 工具 ══
function sleep(ms) {
  return new Promise(r => setTimeout(r, ms))
}

function scrollToTyping() {
  nextTick(() => {
    typingLineRef.value?.scrollIntoView({ behavior: 'smooth', block: 'end' })
  })
}

// ══ 打字机主循环 ══
async function startTyping() {
  showCursor.value = true
  await sleep(800)

  for (let sIdx = 0; sIdx < sections.length; sIdx++) {
    const section = sections[sIdx]

    // 段间空行
    if (sIdx > 0) {
      displayedLines.value.push({ text: '', empty: true })
      await sleep(350)
    }

    // 灯光扩大 + 切换装饰
    lightOpacity.value = section.light
    activeDecoration.value = section.decoration
    await sleep(500)

    for (let lIdx = 0; lIdx < section.lines.length; lIdx++) {
      const line = section.lines[lIdx]
      currentText.value = ''
      scrollToTyping()

      for (let i = 0; i < line.length; i++) {
        currentText.value += line[i]
        scrollToTyping()

        const ch = line[i]
        if ('。！'.includes(ch)) await sleep(400)
        else if (ch === '，') await sleep(320)
        else if (ch === '—') await sleep(100)
        else if ('？…'.includes(ch)) await sleep(340)
        else await sleep(58)
      }

      const isIntro = section.decoration !== null && lIdx === 0
      displayedLines.value.push({ text: line, intro: isIntro })
      currentText.value = ''
      await sleep(580)
    }
  }

  activeDecoration.value = null
  showCursor.value = false
  await sleep(500)
  showButton.value = true
}

// ══ 推门 ══
function openDoor() {
  showButton.value = false
  contentFading.value = true
  setTimeout(() => { doorOpen.value = true }, 250)
  setTimeout(() => { emit('done') }, 950)
}

onMounted(() => {
  // 渐暗入场 → 开始打字
  requestAnimationFrame(() => {
    overlayOpacity.value = 0.93
    setTimeout(() => startTyping(), 1500)
  })
})
</script>

<template>
  <div class="story-page">

    <!-- 暗色遮罩（入场渐暗后固定） -->
    <div class="darkness-overlay" :style="{ opacity: overlayOpacity }"></div>

    <!-- 头顶灯光（逐渐扩大） -->
    <div class="light-glow" :style="{ opacity: lightOpacity }"></div>

    <!-- ══ 装饰层 ══ -->
    <div class="decoration-layer">
      <!-- 烛火 -->
      <Transition name="deco">
        <div v-if="activeDecoration === 'candle'" class="deco">
          <span class="glow glow-1"></span>
          <span class="glow glow-2"></span>
          <span class="glow glow-3"></span>
        </div>
      </Transition>

      <!-- 水波 -->
      <Transition name="deco">
        <div v-if="activeDecoration === 'water'" class="deco">
          <span class="wave wave-1"></span>
          <span class="wave wave-2"></span>
          <span class="wave wave-3"></span>
          <span class="wave wave-4"></span>
        </div>
      </Transition>

      <!-- 花瓣 -->
      <Transition name="deco">
        <div v-if="activeDecoration === 'petals'" class="deco">
          <span
            v-for="(p, i) in petals"
            :key="i"
            class="petal"
            :style="{
              left: p.left + '%',
              fontSize: p.size + 'px',
              animationDelay: p.delay + 's',
              animationDuration: p.dur + 's',
            }"
          >{{ p.sym }}</span>
        </div>
      </Transition>

      <!-- 收据 -->
      <Transition name="deco">
        <div v-if="activeDecoration === 'receipt'" class="deco">
          <div class="receipt-paper">
            <span class="rline"></span>
            <span class="rline short"></span>
            <span class="rline"></span>
            <span class="rline medium"></span>
            <span class="rline short"></span>
            <span class="rstain"></span>
            <span class="rstain stain-2"></span>
          </div>
        </div>
      </Transition>
    </div>

    <!-- ══ 内容 ══ -->
    <div class="story-content" :class="{ fading: contentFading }">
      <div class="story-lines">
        <p
          v-for="(line, i) in displayedLines"
          :key="i"
          class="story-line"
          :class="{ empty: line.empty, intro: line.intro }"
        >{{ line.text }}</p>

        <p
          v-if="currentText !== '' || showCursor"
          ref="typingLineRef"
          class="story-line typing-line"
        >
          {{ currentText }}<span v-if="showCursor" class="cursor">▊</span>
        </p>
      </div>

      <Transition name="btn">
        <button
          v-if="showButton"
          class="door-btn"
          @click="openDoor"
        >
          靠近看看
        </button>
      </Transition>
    </div>

    <!-- 推门光效 -->
    <div v-if="doorOpen" class="door-light"></div>
  </div>
</template>

<style scoped>
/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   页面
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.story-page {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 40px 24px;
  overflow-y: auto;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   暗色遮罩 —— 入场渐暗后固定不动
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.darkness-overlay {
  position: fixed;
  inset: 0;
  background: rgb(18, 18, 24);
  z-index: 1;
  transition: opacity 1.4s ease;
  pointer-events: none;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   头顶灯光 —— 从小光圈逐渐扩大
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.light-glow {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(
    ellipse at 50% 12%,
    rgba(255, 215, 150, 0.55) 0%,
    rgba(255, 200, 120, 0.3) 18%,
    rgba(255, 185, 95, 0.12) 40%,
    transparent 65%
  );
  z-index: 2;
  transition: opacity 1.8s ease;
  pointer-events: none;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   装饰层
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.decoration-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 3;
  overflow: hidden;
}

.deco {
  position: absolute;
  inset: 0;
}

.deco-enter-active,
.deco-leave-active {
  transition: opacity 1s ease;
}
.deco-enter-from,
.deco-leave-to {
  opacity: 0;
}

/* ── 烛火 ── */
.glow {
  position: absolute;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: rgba(255, 180, 80, 0.35);
  box-shadow:
    0 0 20px 10px rgba(255, 180, 80, 0.12),
    0 0 45px 20px rgba(255, 160, 60, 0.06);
}

.glow-1 {
  top: 18%;
  left: 15%;
  animation: candleBreathe 3s ease-in-out infinite;
}
.glow-2 {
  top: 42%;
  right: 12%;
  animation: candleBreathe 3.5s ease-in-out 0.8s infinite;
}
.glow-3 {
  bottom: 32%;
  left: 55%;
  animation: candleBreathe 4s ease-in-out 1.6s infinite;
}

@keyframes candleBreathe {
  0%, 100% { opacity: 0.35; transform: scale(1); }
  50%      { opacity: 0.7;  transform: scale(1.15); }
}

/* ── 水波 ── */
.wave {
  position: absolute;
  width: 55%;
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(100, 180, 170, 0.28),
    transparent
  );
  animation: waveFloat linear infinite;
}

.wave-1 { top: 22%; animation-duration: 7s; }
.wave-2 { top: 42%; animation-duration: 9s;  animation-delay: 1.2s; height: 1.5px; }
.wave-3 { top: 62%; animation-duration: 8s;  animation-delay: 2.8s; }
.wave-4 { top: 78%; animation-duration: 10s; animation-delay: 0.5s; width: 40%; }

@keyframes waveFloat {
  0%   { transform: translateX(-60%) translateY(0);   opacity: 0; }
  15%  { opacity: 1; }
  85%  { opacity: 1; }
  100% { transform: translateX(160%) translateY(4px); opacity: 0; }
}

/* ── 花瓣 ── */
.petal {
  position: absolute;
  top: -15px;
  color: rgba(210, 160, 175, 0.4);
  animation: petalFall linear infinite;
  animation-fill-mode: both;
}

@keyframes petalFall {
  0%   { transform: translateY(0)     translateX(0)    rotate(0deg);   opacity: 0; }
  8%   { opacity: 0.6; }
  92%  { opacity: 0.6; }
  100% { transform: translateY(105vh) translateX(20px) rotate(200deg); opacity: 0; }
}

/* ── 收据 ── */
.receipt-paper {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(-3deg);
  width: 110px;
  height: 150px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  padding: 20px 14px;
  opacity: 0.09;
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 2px;
}

.rline {
  display: block;
  height: 1.5px;
  background: rgba(255, 255, 255, 0.5);
  width: 100%;
  border-radius: 1px;
}
.rline.short  { width: 45%; }
.rline.medium { width: 70%; }

.rstain {
  position: absolute;
  width: 18px;
  height: 16px;
  border-radius: 50%;
  background: rgba(180, 150, 80, 0.35);
  top: 22%;
  right: 14%;
}
.rstain.stain-2 {
  width: 12px;
  height: 12px;
  top: 68%;
  left: 18%;
  background: rgba(160, 130, 70, 0.25);
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   内容
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.story-content {
  position: relative;
  z-index: 4;
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 320px;
  width: 100%;
  gap: 40px;
  transition: opacity 0.35s ease;
}

.story-content.fading {
  opacity: 0;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   文字行 —— 固定白色
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.story-lines {
  display: flex;
  flex-direction: column;
  gap: 4px;
  width: 100%;
}

.story-line {
  font-size: 14px;
  line-height: 2;
  letter-spacing: 1.5px;
  color: rgba(255, 255, 255, 0.9);
  text-align: left;
  margin: 0;
  min-height: 1.2em;
}

.story-line.empty {
  min-height: 18px;
}

.story-line.intro {
  font-size: 12px;
  opacity: 0.5;
  letter-spacing: 1px;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   光标
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.cursor {
  display: inline-block;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
  margin-left: 2px;
  animation: cursorBlink 0.7s step-end infinite;
  vertical-align: baseline;
}

@keyframes cursorBlink {
  0%, 100% { opacity: 1; }
  50%      { opacity: 0; }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   按钮
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.door-btn {
  font-size: 14px;
  letter-spacing: 1.5px;
  line-height: 2;
  color: rgba(255, 210, 140, 0.8);
  border: none;
  background: transparent;
  cursor: pointer;
  padding: 0;
  text-shadow: 0 0 16px rgba(255, 210, 140, 0.3);
  animation: breathe 3s ease-in-out infinite;
}

.door-btn:active {
  text-shadow: 0 0 24px rgba(255, 210, 140, 0.6);
}

@keyframes breathe {
  0%, 100% { opacity: 0.5; }
  50%      { opacity: 1; }
}
.btn-enter-active {
  animation: btnFadeUp 0.5s ease;
}
.btn-leave-active {
  transition: opacity 0.2s ease;
}
.btn-leave-to {
  opacity: 0;
}

@keyframes btnFadeUp {
  from { opacity: 0; transform: translateY(14px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   推门光效
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.door-light {
  position: fixed;
  inset: 0;
  background: rgba(255, 255, 255, 0.95);
  z-index: 100;
  animation: doorExpand 0.65s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

@keyframes doorExpand {
  0%  { clip-path: inset(0 50% 0 50%);     opacity: 0.85; }
  15% { clip-path: inset(0 49.7% 0 49.7%); opacity: 1; }
  100%{ clip-path: inset(0 0 0 0);          opacity: 1; }
}
</style>