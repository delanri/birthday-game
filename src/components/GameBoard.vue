<script setup>
import { ref, watch, onBeforeUnmount } from 'vue'

const props = defineProps({
  colorStage: { type: Number, default: 2 }
})

const emit = defineEmits(['complete', 'roundChange'])

// ══════════════════════════════════════
//  游戏数据
// ══════════════════════════════════════
const GROUPS = [
  { id: 0, name: '光遇', color: '#C0D8F0', accent: '#89B4D4', icons: ['🕯️', '⭐'], label: '星空下的冒险' },
  { id: 1, name: '原神', color: '#C8E4A8', accent: '#8CB868', icons: ['🍃', '⚡'], label: '竹筏上的旅途' },
  { id: 2, name: '星露谷', color: '#F0C8D8', accent: '#D898B0', icons: ['🍓', '🌻'], label: '花舞节的回忆' },
  { id: 3, name: '胡闹厨房', color: '#F0DCA8', accent: '#D0B870', icons: ['🍳', '🧅'], label: '三星大厨！' },
]

function makeCards(ordered) {
  const cards = []
  GROUPS.forEach(g => {
    g.icons.forEach((icon, pi) => {
      for (let c = 0; c < 2; c++) {
        cards.push({
          id: `${g.id}-${pi}-${c}`,
          groupId: g.id,
          icon,
          matchKey: `${g.id}-${pi}`,
        })
      }
    })
  })
  if (!ordered) shuffleArr(cards)
  return cards
}

function shuffleArr(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

// ══════════════════════════════════════
//  游戏状态
// ══════════════════════════════════════
const phase = ref('preview') // preview | playing | roundDone
const round = ref(1)
const cards = ref(makeCards(true))
const flipped = ref([])         // 当前翻开的牌的索引
const matched = ref(new Set())  // 已配对的牌id
const doneGroups = ref(new Set())
const locked = ref(false)
const countdown = ref(5)
const cheatMode = ref(false)

const showPopup = ref(null)     // { type, ... }
const mischiefMsg = ref('')
const turns = ref(0)
const mcd = ref(0) // mischief cooldown
const albumUnlocked = ref(false)
const timers = []

function later(fn, ms) {
  const t = setTimeout(fn, ms)
  timers.push(t)
  return t
}

onBeforeUnmount(() => timers.forEach(clearTimeout))

// ── 倒计时 ──
let countdownTimer = null
watch(phase, (val) => {
  if (val === 'preview') {
    countdown.value = 5
    clearInterval(countdownTimer)
    countdownTimer = setInterval(() => {
      countdown.value--
      if (countdown.value <= 0) {
        clearInterval(countdownTimer)
        cards.value = shuffleArr([...cards.value])
        phase.value = 'playing'
      }
    }, 1000)
  }
}, { immediate: true })

onBeforeUnmount(() => clearInterval(countdownTimer))

// ══════════════════════════════════════
//  游戏逻辑
// ══════════════════════════════════════

function getGroup(id) {
  return GROUPS.find(g => g.id === id)
}

function isCardVisible(idx) {
  return phase.value === 'preview'
    || flipped.value.includes(idx)
    || matched.value.has(cards.value[idx].id)
}

function onCardClick(idx) {
  if (phase.value !== 'playing' || locked.value) return
  const card = cards.value[idx]
  if (flipped.value.includes(idx) || matched.value.has(card.id)) return

  const next = [...flipped.value, idx]
  flipped.value = next
  if (next.length < 2) return

  locked.value = true
  turns.value++
  mcd.value = Math.max(0, mcd.value - 1)

  const [a, b] = next
  const ca = cards.value[a]
  const cb = cards.value[b]

  if (ca.matchKey === cb.matchKey) {
    // ── 配对成功 ──

    // 第二轮捣蛋：偷走配对
    if (round.value === 2 && Math.random() < 0.25 && mcd.value <= 0) {
      mcd.value = 4
      mischiefMsg.value = '逃跑啦！🏃‍♀️'
      later(() => {
        const arr = [...cards.value]
        const avail = arr.map((_, i) => i).filter(i =>
          !matched.value.has(arr[i].id) && i !== a && i !== b
        )
        if (avail.length) {
          const si = avail[Math.floor(Math.random() * avail.length)]
          ;[arr[a], arr[si]] = [arr[si], arr[a]]
          cards.value = arr
        }
        flipped.value = []
        locked.value = false
        mischiefMsg.value = ''
      }, 900)
      return
    }

    // 正常配对
    later(() => {
      const nm = new Set(matched.value)
      nm.add(ca.id)
      nm.add(cb.id)
      matched.value = nm
      flipped.value = []

      // 检查一组是否完成
      const gid = ca.groupId
      const groupDone = cards.value
        .filter(c => c.groupId === gid)
        .every(c => nm.has(c.id))

      if (groupDone) {
        const nd = new Set(doneGroups.value)
        nd.add(gid)
        doneGroups.value = nd
        showPopup.value = { type: 'group', gid }

        later(() => {
          showPopup.value = null
          // 全部完成
          if (nd.size === 4) {
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
        }, 1400)
      }

      locked.value = false
    }, 500)
  } else {
    // ── 没有配对 ──

    // 第二轮捣蛋：偶尔偷偷换位
    const doSwap = round.value === 2 && turns.value % 5 === 0 && turns.value > 0

    later(() => {
      if (doSwap) {
        mischiefMsg.value = '嘻嘻 🔀'
        later(() => {
          const arr = [...cards.value]
          const avail = arr.map((_, i) => i).filter(i =>
            !matched.value.has(arr[i].id) && !next.includes(i)
          )
          if (avail.length >= 2) {
            const i1 = avail[Math.floor(Math.random() * avail.length)]
            let i2
            do { i2 = avail[Math.floor(Math.random() * avail.length)] } while (i2 === i1)
            ;[arr[i1], arr[i2]] = [arr[i2], arr[i1]]
            cards.value = arr
          }
          mischiefMsg.value = ''
        }, 700)
      }
      flipped.value = []
      locked.value = false
    }, 800)
  }
}

// ── 作弊模式：一键通关 ──
function cheatSkip() {
  matched.value = new Set(cards.value.map(c => c.id))
  doneGroups.value = new Set(GROUPS.map(g => g.id))
  phase.value = 'roundDone'
  if (round.value === 1) {
    albumUnlocked.value = true
    showPopup.value = { type: 'round1' }
  } else {
    showPopup.value = { type: 'round2' }
  }
}

// ── 开始第二轮 ──
function startRound2() {
  round.value = 2
  emit('roundChange', 2)
  cards.value = makeCards(true)
  flipped.value = []
  matched.value = new Set()
  doneGroups.value = new Set()
  locked.value = false
  turns.value = 0
  mcd.value = 0
  showPopup.value = null
  mischiefMsg.value = ''
  phase.value = 'preview'
}
</script>

<template>
  <div class="game-page">
    <div class="game-container">
      <!-- ── 顶部信息 ── -->
      <div class="game-header">
        <div class="header-text">
          <template v-if="phase === 'preview'">
            记住它们的位置！ {{ countdown }}s
          </template>
          <template v-else-if="phase === 'playing'">
            {{ round === 2 ? '第二轮 · 小心捣蛋鬼！' : '翻开卡片，找到配对 ♪' }}
          </template>
          <template v-else>
            完成！
          </template>
        </div>
        <div v-if="round === 2 && phase === 'playing'" class="header-sub">
          卡片可能会偷偷跑掉哦~
        </div>
      </div>

      <!-- ── 收集栏 ── -->
      <div class="collection-bar">
        <div
          v-for="g in GROUPS"
          :key="g.id"
          class="collection-slot"
          :class="{ done: doneGroups.has(g.id) }"
          :style="{
            borderColor: doneGroups.has(g.id) ? g.accent : 'var(--border)',
            background: doneGroups.has(g.id) ? g.color : 'var(--card-back)',
          }"
        >
          <template v-if="doneGroups.has(g.id)">
            <span class="slot-icon">{{ g.icons[0] }}</span>
            <span class="slot-label" :style="{ color: g.accent }">{{ g.name }}</span>
          </template>
          <template v-else>
            <span class="slot-empty">?</span>
          </template>
        </div>
      </div>

      <!-- ── 卡牌区 ── -->
      <div class="card-grid">
        <div
          v-for="(card, idx) in cards"
          :key="card.id + '-' + idx"
          class="card-cell"
          @click="onCardClick(idx)"
        >
          <div
            class="card-inner"
            :class="{ flipped: isCardVisible(idx), matched: matched.has(card.id) }"
          >
            <!-- 卡牌背面 -->
            <div class="card-back">
              <span class="card-back-icon">?</span>
            </div>
            <!-- 卡牌正面 -->
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
                class="card-group-label"
                :style="{ color: getGroup(card.groupId).accent }"
              >
                {{ getGroup(card.groupId).name }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <!-- ── 作弊按钮 ── -->
<div v-if="phase === 'playing'" class="cheat-area">
  <button class="cheat-btn" @click="cheatSkip">
    🚀 一键通关
  </button>
</div>

      <!-- ── 回合结束后的按钮 ── -->
      <div v-if="phase === 'roundDone' && !showPopup" class="done-actions">
        <button v-if="round === 1" class="pixel-btn" @click="startRound2">
          再来一次 ✨
        </button>
        <button v-else class="pixel-btn" @click="emit('complete')">
          查看最后的惊喜 ♡
        </button>
      </div>
    </div>

    <!-- ══════════ 弹窗层 ══════════ -->

    <!-- 捣蛋消息 -->
    <Transition name="pop">
      <div v-if="mischiefMsg" class="mischief-toast pop-in">
        {{ mischiefMsg }}
      </div>
    </Transition>

    <!-- 一组完成 -->
    <Transition name="pop">
      <div v-if="showPopup?.type === 'group'" class="overlay-popup pop-in">
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
        <div class="overlay-box pop-in">
          <div class="popup-emoji">🎉</div>
          <div class="popup-title">记忆全部找回来了！</div>
          <div class="popup-sub">相册已解锁</div>
          <div class="popup-hint">✨ 还有一张隐藏照片...再玩一次解锁它？</div>
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
        <div class="overlay-box pop-in">
          <div class="popup-emoji">🏆</div>
          <div class="popup-title">全成就解锁！</div>
          <div class="popup-sub">你找到了所有记忆碎片</div>
          <button class="pixel-btn" @click="showPopup = null; emit('complete')">
            查看隐藏照片 ♡
          </button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.game-page {
  display: flex;
  justify-content: center;
  min-height: 100vh;
  padding: 16px;
}

.game-container {
  width: 100%;
  max-width: 380px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding-top: 12px;
}

/* ── 顶部信息 ── */
.game-header {
  text-align: center;
}
.header-text {
  font-size: 14px;
  color: var(--text);
  font-weight: 600;
  letter-spacing: 1px;
}
.header-sub {
  font-size: 11px;
  color: var(--accent);
  margin-top: 2px;
}

/* ── 收集栏 ── */
.collection-bar {
  display: flex;
  gap: 6px;
  width: 100%;
  justify-content: center;
}

.collection-slot {
  width: 76px;
  height: 52px;
  border: 2px solid var(--border);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.4s ease;
  opacity: 0.5;
}

.collection-slot.done {
  opacity: 1;
  box-shadow: 0 3px 0 0 var(--shadow);
}

.slot-icon { font-size: 16px; }
.slot-label { font-size: 9px; font-weight: 600; margin-top: 1px; }
.slot-empty { font-size: 14px; color: var(--border); }

/* ── 卡牌网格 ── */
.card-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  width: 100%;
  max-width: 340px;
}

.card-cell {
  aspect-ratio: 1;
  perspective: 600px;
  cursor: pointer;
}

.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.45s ease;
}

.card-inner.flipped {
  transform: rotateY(180deg);
}

.card-inner.matched {
  opacity: 0.65;
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
  border: 2px solid var(--border);
}

.card-back {
  background:
    repeating-conic-gradient(var(--card-back) 0% 25%, #F5F0EB 0% 50%)
    50% / 12px 12px;
  box-shadow: 0 3px 0 0 var(--shadow);
}

.card-back-icon {
  font-size: 20px;
  color: var(--border);
}

.card-front {
  transform: rotateY(180deg);
  border-width: 2px;
  box-shadow: 0 3px 0 0 var(--shadow);
}

.card-icon {
  font-size: 28px;
}

.card-group-label {
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 0.5px;
  margin-top: 2px;
}

/* ── 作弊区 ── */
.cheat-area {
  display: flex;
  gap: 8px;
  margin-top: 8px;
}

.cheat-btn {
  padding: 6px 14px;
  border: 2px solid var(--border);
  background: var(--card-back);
  color: var(--text-light);
  font-size: 12px;
  cursor: pointer;
  font-family: inherit;
  transition: transform 0.1s;
}

.cheat-btn:active {
  transform: translateY(2px);
}

.cheat-peek {
  background: #FFF0E0;
  color: #D0A060;
  border-color: #D0A060;
}

/* ── 完成后按钮 ── */
.done-actions {
  margin-top: 16px;
}

/* ── 捣蛋消息 ── */
.mischief-toast {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #fff;
  border: 3px solid var(--accent);
  padding: 12px 24px;
  font-size: 16px;
  font-weight: 700;
  color: var(--accent);
  z-index: 100;
  box-shadow: 0 4px 0 0 var(--shadow);
}

/* ── 弹窗 ── */
.overlay-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #fff;
  border: 3px solid var(--border-dark);
  padding: 20px 28px;
  text-align: center;
  z-index: 100;
  box-shadow: 0 4px 0 0 var(--shadow);
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
  border: 3px solid var(--border-dark);
  padding: 28px 24px;
  text-align: center;
  max-width: 320px;
  width: 90%;
  box-shadow: 0 6px 0 0 var(--shadow);
}

.popup-emoji { font-size: 36px; margin-bottom: 12px; }
.popup-icons { font-size: 28px; margin-bottom: 8px; }
.popup-title { font-size: 16px; font-weight: 700; margin-bottom: 6px; }
.popup-sub { font-size: 12px; color: var(--text-light); margin-bottom: 4px; }

.popup-hint {
  font-size: 12px;
  color: var(--accent);
  padding: 8px;
  background: #FFF5F5;
  border: 1px solid #F0D0D0;
  margin: 12px 0;
}

.popup-btns {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 16px;
}

/* ── 过渡动画 ── */
.pop-enter-active { animation: popIn 0.3s ease; }
.pop-leave-active { transition: opacity 0.2s ease; }
.pop-leave-to { opacity: 0; }
</style>