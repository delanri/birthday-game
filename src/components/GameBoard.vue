<script setup>
import { ref, computed, nextTick, onMounted, onBeforeUnmount } from 'vue'
import TutorialPage from './TutorialPage.vue'

const props = defineProps({
  colorStage: { type: Number, default: 2 },
})

const emit = defineEmits(['complete', 'roundChange'])

// ══════════════════════════════════════════════
//  游戏数据
// ══════════════════════════════════════════════
const GROUPS = [
  { id: 0, name: '光遇',     icons: ['🕯️', '⭐', '🌙', '☁️'], color: '#C0D8F0', accent: '#89B4D4', label: '星空下的冒险' },
  { id: 1, name: '原神',     icons: ['🍃', '💨', '⚡', '🪨'],  color: '#C8E4A8', accent: '#8CB868', label: '竹筏上的旅途' },
  { id: 2, name: '星露谷',   icons: ['🥬', '🥩', '🧅', '🍅'],  color: '#F0C8D8', accent: '#D898B0', label: '花舞节的回忆' },
  { id: 3, name: '胡闹厨房', icons: ['🍳', '🍖', '🥘', '🧈'],  color: '#F0DCA8', accent: '#D0B870', label: '三星大厨！' },
]

const SHAKE_MSGS = ['才不给你看！😝', '哼，想偷看？🙈', '看清了吗～才怪！']
const RUN_MSGS   = ['逃跑啦！🏃', '抓不到我～💨', '溜了溜了！']

// ══════════════════════════════════════════════
//  工具
// ══════════════════════════════════════════════
function shuffleArr(arr) {
  const a = [...arr]
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

function pick(arr) { return arr[Math.floor(Math.random() * arr.length)] }

function gridPos(idx) { return { row: Math.floor(idx / 4), col: idx % 4 } }

function calcOffset(from, to) {
  const f = gridPos(from), t = gridPos(to)
  return {
    x: `calc(${t.col - f.col} * (100% + 8px))`,
    y: `calc(${t.row - f.row} * (100% + 8px))`,
  }
}

function makeCards() {
  const cards = []
  GROUPS.forEach(g => {
    g.icons.forEach((icon, i) => {
      cards.push({ id: `${g.id}-${i}`, groupId: g.id, icon })
    })
  })
  return shuffleArr(cards)
}

// 计时器管理
const timers = []
function later(fn, ms) { const t = setTimeout(fn, ms); timers.push(t); return t }
let cdInterval = null
onBeforeUnmount(() => { timers.forEach(clearTimeout); clearInterval(cdInterval) })

// ══════════════════════════════════════════════
//  状态
// ══════════════════════════════════════════════
const phase     = ref('tutorial')   // tutorial | preview | playing | roundDone
const round     = ref(1)
const cards     = ref(makeCards())
const flipped   = ref([])           // 当前翻开的索引 (最多2)
const matched   = ref(new Set())    // 已配对的牌id
const groupProg = ref({ 0: 0, 1: 0, 2: 0, 3: 0 }) // 每组配对次数 0/1/2
const locked    = ref(false)
const countdown = ref(2)

// 捣蛋
const flipCount      = ref(0)
const mischiefMsg    = ref('')
const shakingIdx     = ref(-1)
const cardAnims      = ref({})          // { idx: { x, y } } 跑路位移
const legSet         = ref(new Set())   // 显示小腿的牌
const crawlAnims     = ref({})          // { idx: { x, y } } 蜗牛爬
const skipTransition = ref(false)

// UI
const showPopup      = ref(null)
const albumUnlocked  = ref(false)

// ══════════════════════════════════════════════
//  计算
// ══════════════════════════════════════════════
const remaining = computed(() =>
  cards.value.filter(c => !matched.value.has(c.id)).length
)

const mischiefChance = computed(() => {
  if (remaining.value <= 6) return 1.0
  return Math.min(1.0, 0.4 + Math.floor(flipCount.value / 2) * 0.1)
})

// ══════════════════════════════════════════════
//  辅助查询
// ══════════════════════════════════════════════
function getGroup(gid)     { return GROUPS.find(g => g.id === gid) }
function isFlipped(idx)    { return phase.value === 'preview' || flipped.value.includes(idx) }
function isMatched(idx)    { return matched.value.has(cards.value[idx].id) }
function unmatchedIdxs()   { return cards.value.map((_, i) => i).filter(i => !isMatched(i)) }
function emptyIdxs()       { return cards.value.map((_, i) => i).filter(i => isMatched(i)) }

// ══════════════════════════════════════════════
//  教程结束 → 预览
// ══════════════════════════════════════════════
function onTutorialDone() {
  phase.value = 'preview'
  countdown.value = 2
  clearInterval(cdInterval)
  cdInterval = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) {
      clearInterval(cdInterval)
      phase.value = 'playing'
    }
  }, 1000)
}

// ══════════════════════════════════════════════
//  卡牌点击
// ══════════════════════════════════════════════
function onCardClick(idx) {
  if (phase.value !== 'playing' || locked.value) return
  const card = cards.value[idx]
  if (matched.value.has(card.id) || flipped.value.includes(idx)) return

  // 点击正在蜗牛爬的牌 → 打断它
  if (crawlAnims.value[idx]) {
    const ca = { ...crawlAnims.value }
    delete ca[idx]
    crawlAnims.value = ca
    const ls = new Set(legSet.value); ls.delete(idx); legSet.value = ls
  }

  const isSecond = flipped.value.length === 1

  // 第二轮计数
  if (round.value === 2) flipCount.value++

  // ── 第二轮：翻第二张时可能「抖回去」 ──
  if (round.value === 2 && isSecond && Math.random() < mischiefChance.value) {
    doShakeBack(idx)
    return
  }

  flipped.value = [...flipped.value, idx]
  if (!isSecond) return   // 第一张牌，等第二张

  // ── 两张都翻了，判定 ──
  locked.value = true
  const [a, b] = flipped.value
  const ca = cards.value[a], cb = cards.value[b]

  if (ca.groupId === cb.groupId) {
    // ✦ 配对成功 ✦
    later(() => {
      const nm = new Set(matched.value)
      nm.add(ca.id); nm.add(cb.id)
      matched.value = nm
      flipped.value = []

      const gp = { ...groupProg.value }
      gp[ca.groupId]++
      groupProg.value = gp

      if (gp[ca.groupId] === 2) {
        showPopup.value = { type: 'group', gid: ca.groupId }
        later(() => {
          showPopup.value = null
          if (GROUPS.every(g => gp[g.id] === 2)) {
            finishRound()
          } else {
            locked.value = false
            if (round.value === 2) afterAction()
          }
        }, 1400)
      } else {
        locked.value = false
        if (round.value === 2) afterAction()
      }
    }, 600)
  } else {
    // ✗ 没配上
    later(() => {
      flipped.value = []
      locked.value = false
      if (round.value === 2) afterAction()
    }, 800)
  }
}

// ══════════════════════════════════════════════
//  回合结束
// ══════════════════════════════════════════════
function finishRound() {
  later(() => {
    phase.value = 'roundDone'
    if (round.value === 1) {
      albumUnlocked.value = true
      showPopup.value = { type: 'round1' }
    } else {
      showPopup.value = { type: 'round2' }
    }
  }, 500)
}

function startRound2() {
  round.value = 2
  emit('roundChange', 2)
  cards.value = makeCards()
  flipped.value = []
  matched.value = new Set()
  groupProg.value = { 0: 0, 1: 0, 2: 0, 3: 0 }
  locked.value = false
  flipCount.value = 0
  showPopup.value = null
  mischiefMsg.value = ''
  shakingIdx.value = -1
  cardAnims.value = {}
  legSet.value = new Set()
  crawlAnims.value = {}
  phase.value = 'tutorial'
}

// ══════════════════════════════════════════════
//  捣蛋：抖回去
// ══════════════════════════════════════════════
function doShakeBack(idx) {
  locked.value = true
  // 先翻开让她看到
  flipped.value = [...flipped.value, idx]

  later(() => {
    shakingIdx.value = idx
    later(() => {
      shakingIdx.value = -1
      flipped.value = flipped.value.filter(i => i !== idx)
      locked.value = false
      mischiefMsg.value = pick(SHAKE_MSGS)
      later(() => { mischiefMsg.value = '' }, 1500)
    }, 600)
  }, 500)
}

// ══════════════════════════════════════════════
//  捣蛋：跑路
// ══════════════════════════════════════════════
function afterAction() {
  later(() => maybeRunaway(), 350)
  later(() => maybeCrawl(), 900)
}

function maybeRunaway() {
  if (locked.value) return
  if (Math.random() >= mischiefChance.value) return

  const um = unmatchedIdxs()
  if (um.length < 2) return
  const em = emptyIdxs()

  const fromIdx = pick(um)
  let toIdx

  if (em.length > 0) {
    // 跑到空位
    toIdx = pick(em)
  } else {
    // 互换
    const others = um.filter(i => i !== fromIdx)
    if (!others.length) return
    toIdx = pick(others)
  }

  locked.value = true
  mischiefMsg.value = pick(RUN_MSGS)

  // 长腿
  const ls = new Set(legSet.value)
  ls.add(fromIdx)
  const toIsCard = !matched.value.has(cards.value[toIdx].id)
  if (toIsCard) ls.add(toIdx)
  legSet.value = ls

  // 延迟后开始移动
  later(() => {
    const anims = { [fromIdx]: calcOffset(fromIdx, toIdx) }
    if (toIsCard) anims[toIdx] = calcOffset(toIdx, fromIdx)
    cardAnims.value = anims
  }, 250)

  // 动画结束 → swap
  later(() => {
    skipTransition.value = true

    const arr = [...cards.value]
    ;[arr[fromIdx], arr[toIdx]] = [arr[toIdx], arr[fromIdx]]
    cards.value = arr
    cardAnims.value = {}
    legSet.value = new Set()
    mischiefMsg.value = ''

    nextTick(() => {
      skipTransition.value = false
      locked.value = false
    })
  }, 950)
}

// ══════════════════════════════════════════════
//  捣蛋：蜗牛爬
// ══════════════════════════════════════════════
function maybeCrawl() {
  const em = emptyIdxs()
  if (em.length < 6) return

  const um = unmatchedIdxs().filter(i => !crawlAnims.value[i])
  if (!um.length) return

  // 最后阶段允许多只同时爬
  const count = remaining.value <= 6 ? Math.min(2, um.length) : 1

  for (let n = 0; n < count; n++) {
    const pool = um.filter(i => !crawlAnims.value[i])
    if (!pool.length) break
    const idx   = pick(pool)
    const toIdx = pick(em)
    const off   = calcOffset(idx, toIdx)

    const ls = new Set(legSet.value); ls.add(idx); legSet.value = ls
    crawlAnims.value = { ...crawlAnims.value, [idx]: off }

    later(() => {
      const ca = { ...crawlAnims.value }; delete ca[idx]; crawlAnims.value = ca
      const ls2 = new Set(legSet.value); ls2.delete(idx); legSet.value = ls2
    }, 3000 + n * 500)
  }
}

// ══════════════════════════════════════════════
//  作弊
// ══════════════════════════════════════════════
function cheatSkip() {
  matched.value = new Set(cards.value.map(c => c.id))
  groupProg.value = { 0: 2, 1: 2, 2: 2, 3: 2 }
  finishRound()
}

// ══════════════════════════════════════════════
//  卡片样式计算
// ══════════════════════════════════════════════
function wrapperStyle(idx) {
  if (skipTransition.value) return { transition: 'none' }

  const anim = cardAnims.value[idx]
  if (anim) {
    return {
      transform: `translate(${anim.x}, ${anim.y})`,
      transition: 'transform 0.7s cubic-bezier(0.34, 1.56, 0.64, 1)',
      zIndex: 10,
    }
  }

  const crawl = crawlAnims.value[idx]
  if (crawl) {
    return {
      '--crawl-x': crawl.x,
      '--crawl-y': crawl.y,
      zIndex: 5,
    }
  }

  return {}
}
</script>

<template>
  <div class="game-page">

    <!-- ══ 教程 ══ -->
    <TutorialPage
      v-if="phase === 'tutorial'"
      :round="round"
      @done="onTutorialDone"
    />

    <div class="game-container">

      <!-- ── 顶部提示 ── -->
      <div class="game-header">
        <span v-if="phase === 'preview'" class="header-text">
          记住它们！ {{ countdown }}s
        </span>
        <span v-else-if="phase === 'playing'" class="header-text">
          {{ round === 2 ? '第二轮 · 小心捣蛋鬼！' : '翻开卡片，找到配对 ♪' }}
        </span>
        <span v-else class="header-text">完成！</span>
      </div>

      <!-- ── 收集栏 ── -->
      <div class="collection-bar">
        <div
          v-for="g in GROUPS"
          :key="g.id"
          class="collect-icon"
          :class="{
            half: groupProg[g.id] === 1,
            done: groupProg[g.id] === 2,
          }"
        >
          <span class="collect-emoji">{{ g.icons[0] }}</span>
          <span
            v-if="groupProg[g.id] === 2"
            class="collect-label"
            :style="{ color: g.accent }"
          >{{ g.name }}</span>
        </div>
      </div>

      <!-- ── 卡牌网格 ── -->
      <div class="card-grid">
        <div
          v-for="(card, idx) in cards"
          :key="idx"
          class="card-cell"
        >
          <!-- 已配对 → 空位 -->
          <div v-if="isMatched(idx)" class="card-slot-empty"></div>

          <!-- 活牌 -->
          <div
            v-else
            class="card-wrapper"
            :class="{
              'is-shaking':  shakingIdx === idx,
              'is-crawling': !!crawlAnims[idx],
              'has-legs':    legSet.has(idx),
            }"
            :style="wrapperStyle(idx)"
            @click="onCardClick(idx)"
          >
            <div class="card-inner" :class="{ 'is-flipped': isFlipped(idx) }">
              <!-- 背面 -->
              <div class="card-back">
                <span class="back-q">?</span>
              </div>
              <!-- 正面 -->
              <div
                class="card-front"
                :style="{
                  background: getGroup(card.groupId).color,
                  borderColor: getGroup(card.groupId).accent,
                }"
              >
                <span class="card-icon">{{ card.icon }}</span>
                <span
                  v-if="phase === 'preview'"
                  class="card-group-tag"
                  :style="{ color: getGroup(card.groupId).accent }"
                >{{ getGroup(card.groupId).name }}</span>
              </div>
            </div>

            <!-- 像素小腿 -->
            <div v-if="legSet.has(idx)" class="pixel-legs">
              <span class="p-leg"></span>
              <span class="p-leg"></span>
            </div>
          </div>
        </div>
      </div>

      <!-- ── 底部栏 ── -->
      <div class="game-footer">
        <!-- 相册入口（假） -->
        <button class="album-btn" :class="{ unlocked: albumUnlocked }" disabled>
          {{ albumUnlocked ? '📷' : '🔒' }}
          <span class="album-label">相册</span>
        </button>

        <!-- 作弊 -->
        <button v-if="phase === 'playing'" class="cheat-btn" @click="cheatSkip">
          🚀 跳过
        </button>
      </div>

      <!-- ── 回合结束按钮 ── -->
      <div v-if="phase === 'roundDone' && !showPopup" class="done-actions">
        <button v-if="round === 1" class="pixel-btn" @click="startRound2">
          再来一次 ✨
        </button>
        <button v-else class="pixel-btn" @click="emit('complete')">
          查看最后的惊喜 ♡
        </button>
      </div>
    </div>

    <!-- ══════════════════════════════════════════════
         弹窗 & 提示
         ══════════════════════════════════════════════ -->

    <!-- 捣蛋消息 -->
    <Transition name="pop">
      <div v-if="mischiefMsg" class="mischief-toast">
        {{ mischiefMsg }}
      </div>
    </Transition>

    <!-- 一组收集完成 -->
    <Transition name="pop">
      <div v-if="showPopup?.type === 'group'" class="overlay-popup">
        <div class="popup-icons">
          {{ getGroup(showPopup.gid).icons[0] }}
          {{ getGroup(showPopup.gid).icons[1] }}
        </div>
        <div class="popup-title" :style="{ color: getGroup(showPopup.gid).accent }">
          {{ getGroup(showPopup.gid).name }} · 收集完成！
        </div>
        <div class="popup-sub">{{ getGroup(showPopup.gid).label }}</div>
      </div>
    </Transition>

    <!-- 第一轮完成 -->
    <Transition name="pop">
      <div v-if="showPopup?.type === 'round1'" class="overlay">
        <div class="overlay-box">
          <div class="popup-emoji">🎉</div>
          <div class="popup-title">记忆全部找回来了！</div>
          <div class="popup-sub">相册已解锁</div>
          <div class="popup-hint">✨ 还有一张隐藏照片…再玩一次解锁它？</div>
          <div class="popup-btns">
            <button class="pixel-btn pixel-btn--ghost" @click="showPopup = null">
              先歇歇
            </button>
            <button class="pixel-btn" @click="showPopup = null; startRound2()">
              再来一次！
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- 第二轮完成 -->
    <Transition name="pop">
      <div v-if="showPopup?.type === 'round2'" class="overlay">
        <div class="overlay-box">
          <div class="popup-emoji">🏆</div>
          <div class="popup-title">全成就解锁！</div>
          <div class="popup-sub">你找到了所有记忆碎片</div>
          <button
            class="pixel-btn"
            @click="showPopup = null; emit('complete')"
          >
            查看隐藏照片 ♡
          </button>
        </div>
      </div>
    </Transition>

  </div>
</template>

<style scoped>
/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   页面
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.game-page {
  display: flex;
  justify-content: center;
  min-height: 100vh;
  padding: 12px;
}

.game-container {
  width: 100%;
  max-width: 380px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  padding-top: 10px;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   顶部提示
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.game-header { text-align: center; }
.header-text {
  font-size: 13px;
  color: var(--text);
  font-weight: 600;
  letter-spacing: 1px;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   收集栏
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.collection-bar {
  display: flex;
  gap: 10px;
  justify-content: center;
  width: 100%;
}

.collect-icon {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  filter: grayscale(1) brightness(0.15);
  transition: filter 0.8s ease;
}

.collect-icon.half {
  filter: grayscale(0.4) brightness(0.55);
}

.collect-icon.done {
  filter: grayscale(0) brightness(1);
}

.collect-emoji { font-size: 24px; }

.collect-label {
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 0.5px;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   卡牌网格
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.card-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  width: 100%;
  max-width: 340px;
}

/* ── 空位 ── */
.card-cell { aspect-ratio: 1; }

.card-slot-empty {
  width: 100%;
  height: 100%;
  border: 2px dashed var(--border, #C8C0B0);
  opacity: 0.25;
  border-radius: 2px;
}

/* ── 卡片容器 ── */
.card-wrapper {
  width: 100%;
  height: 100%;
  position: relative;
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
}

/* ── 翻转 ── */
.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.4s ease;
}

.card-inner.is-flipped {
  transform: rotateY(180deg);
}

.card-back,
.card-front {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px solid var(--border, #C8C0B0);
}

.card-back {
  background:
    repeating-conic-gradient(
      var(--card-back, #FAF6F0) 0% 25%,
      #F0EBE4 0% 50%
    ) 50% / 12px 12px;
  box-shadow: 0 3px 0 0 var(--shadow, rgba(0,0,0,0.06));
}

.back-q {
  font-size: 18px;
  color: var(--border, #C8C0B0);
  opacity: 0.6;
}

.card-front {
  transform: rotateY(180deg);
  box-shadow: 0 3px 0 0 var(--shadow, rgba(0,0,0,0.06));
}

.card-icon { font-size: 26px; }

.card-group-tag {
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 0.5px;
  margin-top: 2px;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   捣蛋：抖回去
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.card-wrapper.is-shaking {
  animation: cardShake 0.55s ease;
}

@keyframes cardShake {
  0%, 100% { transform: translateX(0) rotate(0); }
  15%      { transform: translateX(-6px) rotate(-3deg); }
  35%      { transform: translateX(6px) rotate(3deg); }
  55%      { transform: translateX(-4px) rotate(-2deg); }
  75%      { transform: translateX(4px) rotate(2deg); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   捣蛋：像素小腿
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.pixel-legs {
  position: absolute;
  bottom: -9px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
  z-index: 11;
}

.p-leg {
  display: block;
  width: 4px;
  height: 7px;
  background: var(--text-light, #8B7E6A);
  border-radius: 0 0 1px 1px;
}

.has-legs .p-leg:first-child {
  animation: legWalk 0.28s step-end infinite;
}
.has-legs .p-leg:last-child {
  animation: legWalk 0.28s step-end 0.14s infinite;
}

@keyframes legWalk {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-5px); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   捣蛋：蜗牛爬
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.card-wrapper.is-crawling {
  animation: crawlTrip 3s ease-in-out;
}

@keyframes crawlTrip {
  0%, 100% { transform: translate(0, 0); }
  35%, 65% { transform: translate(var(--crawl-x, 0px), var(--crawl-y, 0px)); }
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   底部栏
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.game-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  max-width: 340px;
  margin-top: 4px;
}

.album-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  border: 2px solid var(--border, #C8C0B0);
  background: var(--card-back, #FAF6F0);
  font-size: 14px;
  cursor: default;
  font-family: inherit;
  opacity: 0.4;
  transition: opacity 0.4s;
}

.album-btn.unlocked { opacity: 0.7; }

.album-label {
  font-size: 10px;
  color: var(--text-light, #8B7E6A);
}

.cheat-btn {
  padding: 4px 12px;
  border: 2px solid var(--border, #C8C0B0);
  background: var(--card-back, #FAF6F0);
  color: var(--text-light, #8B7E6A);
  font-size: 11px;
  cursor: pointer;
  font-family: inherit;
  transition: transform 0.1s;
}
.cheat-btn:active { transform: translateY(2px); }

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   完成按钮
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.done-actions { margin-top: 12px; }

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   捣蛋消息 toast
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.mischief-toast {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #fff;
  border: 3px solid var(--accent, #D4A574);
  padding: 10px 22px;
  font-size: 15px;
  font-weight: 700;
  color: var(--accent, #D4A574);
  z-index: 100;
  box-shadow: 0 4px 0 0 var(--shadow, rgba(0,0,0,0.08));
  white-space: nowrap;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   弹窗
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.overlay-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #fff;
  border: 3px solid var(--border-dark, #5D4E3C);
  padding: 18px 26px;
  text-align: center;
  z-index: 100;
  box-shadow: 0 4px 0 0 var(--shadow, rgba(0,0,0,0.08));
}

.overlay {
  position: fixed;
  inset: 0;
  background: rgba(93, 78, 60, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
}

.overlay-box {
  background: #fff;
  border: 3px solid var(--border-dark, #5D4E3C);
  padding: 28px 22px;
  text-align: center;
  max-width: 310px;
  width: 90%;
  box-shadow: 0 6px 0 0 var(--shadow, rgba(0,0,0,0.08));
}

.popup-emoji  { font-size: 36px; margin-bottom: 10px; }
.popup-icons  { font-size: 26px; margin-bottom: 6px; }
.popup-title  { font-size: 15px; font-weight: 700; margin-bottom: 4px; }
.popup-sub    { font-size: 12px; color: var(--text-light, #8B7E6A); margin-bottom: 4px; }

.popup-hint {
  font-size: 12px;
  color: var(--accent, #D4A574);
  padding: 8px;
  background: #FFF5F0;
  border: 1px solid #F0D8C8;
  margin: 12px 0;
}

.popup-btns {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 14px;
}

/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   过渡
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */
.pop-enter-active { animation: popIn 0.3s ease; }
.pop-leave-active { transition: opacity 0.2s ease; }
.pop-leave-to     { opacity: 0; }

@keyframes popIn {
  from { opacity: 0; transform: translate(-50%, -50%) scale(0.85); }
  to   { opacity: 1; transform: translate(-50%, -50%) scale(1); }
}
</style>