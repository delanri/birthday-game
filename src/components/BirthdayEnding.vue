<template>
  <div class="birthday-page" :class="'warmth-' + bgWarmth">

    <!-- 白闪 -->
    <div class="white-flash" :class="{ active: whiteFlash }"></div>

    <!-- 散景（庆祝后） -->
    <div v-if="showEffects" class="bokeh-layer">
      <span v-for="(b,i) in bokeh" :key="'bk'+i" class="bokeh-dot"
        :style="{
          left: b.left+'%', width: b.size+'px', height: b.size+'px',
          background: b.bg, animationDelay: b.delay+'s', animationDuration: b.dur+'s',
        }"></span>
    </div>

    <!-- 彩带（庆祝后） -->
    <div v-if="showEffects" class="confetti-layer">
      <span v-for="(c,i) in confetti" :key="i" class="confetti-piece"
        :style="{
          left: c.left+'%', fontSize: c.size+'px', color: c.color,
          animationDelay: c.delay+'s', animationDuration: c.dur+'s',
          '--c-op': c.opacity,
        }">{{ c.sym }}</span>
    </div>

    <div class="birthday-content">

      <!-- 铺子旁白（干净，黑字白底） -->
      <p v-for="(line,i) in lines" :key="i"
        class="b-line" :class="{ visible: i < visibleLines }">{{ line }}</p>

      <!-- 蛋糕区 -->
      <div class="cake-area">

        <div class="cake-backlight" :class="{ glow: showBacklight }"></div>

        <div class="cake" :class="{ highlight: cakeHighlight }">
          <div class="ck-candle" :class="{ 'pop-in': builtStage >= 5 }">
            <div class="ck-flame"></div>
            <div class="ck-stick"></div>
          </div>

          <div class="ck-tier ck-1" :class="{ 'pop-in': builtStage >= 4, lit: droppedRows >= 1 }"></div>
          <div class="frost-row fr-1">
            <span v-for="n in 5" :key="n" class="frost-dot"
              :class="{ dropped: droppedRows >= 1 }"
              :style="{ animationDelay: ((n-1)*0.06)+'s' }"></span>
          </div>

          <div class="ck-tier ck-2" :class="{ 'pop-in': builtStage >= 3, lit: droppedRows >= 2 }"></div>
          <div class="frost-row fr-2">
            <span v-for="n in 7" :key="n" class="frost-dot"
              :class="{ dropped: droppedRows >= 2 }"
              :style="{ animationDelay: ((n-1)*0.05)+'s' }"></span>
          </div>

          <div class="ck-tier ck-3" :class="{ 'pop-in': builtStage >= 2, lit: droppedRows >= 3 }"></div>
          <div class="frost-row fr-3">
            <span v-for="n in 9" :key="n" class="frost-dot"
              :class="{ dropped: droppedRows >= 3 }"
              :style="{ animationDelay: ((n-1)*0.04)+'s' }"></span>
          </div>

          <div class="ck-plate" :class="{ 'pop-in': builtStage >= 1 }"></div>
        </div>

        <div v-if="warmGlow" class="warm-ps">
          <span v-for="n in 4" :key="n" class="warm-p"
            :style="{ left: (25+n*12)+'%', animationDelay: (n*0.7)+'s' }"></span>
        </div>

        <!-- 生日快乐（盖章弹入） -->
        <div class="bd-block" :class="{ visible: showBirthday }">
          <div class="bd-wrap">
            <span class="bd-glow" aria-hidden="true">生日快乐，影子。</span>
            <span class="bd-text">生日快乐，影子</span>
            <span v-for="(s,i) in sparkles" :key="'s'+i" class="bd-sparkle"
              :style="{
                left:`calc(50% + ${s.x}px)`, top:`calc(50% + ${s.y}px)`,
                fontSize: s.size+'px', animationDelay: s.delay+'s', color: s.color,
              }">{{ s.sym }}</span>
            <span v-for="p in burstParticles" :key="'p'+p.id" class="bd-particle"
              :style="{ color: p.color, '--dx': p.dx+'px', '--dy': p.dy+'px' }">{{ p.sym }}</span>
          </div>
        </div>
      </div>

      <div class="heart-space">
        <Transition name="fade">
  <div v-if="showHeart" class="heart-wrap">
    <button class="heart-btn"
      :class="{ shake: heartShaking }"
      :style="{ filter: `hue-rotate(${heartHue}deg)` }"
      @click="emit('back')">♡</button>
    <span class="heart-hint">轻触带走这颗心</span>
  </div>
</Transition>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const emit = defineEmits(['back'])

const lines = [
  '货架还有空位，灯也开始亮了。',
  '每一样东西都等到了它的主人。',
  '放心，铺子不会关门的。',
  '以后还会有更多新的东西。',
  '欢迎下次光临。',
]

// ══ 状态 ══
const visibleLines   = ref(0)
const bgWarmth       = ref(0)
const builtStage     = ref(0)
const droppedRows    = ref(0)
const cakeHighlight  = ref(false)
const showEffects    = ref(false)     // 散景+彩带（庆祝才出现）
const showBacklight  = ref(false)
const showBirthday   = ref(false)
const showHeart      = ref(false)
const whiteFlash     = ref(false)
const warmGlow       = ref(false)
const burstParticles = ref([])

const heartShaking = ref(false)
const heartHue     = ref(0)
let heartTimer     = null

onBeforeUnmount(() => { if (heartTimer) clearInterval(heartTimer) })

onMounted(() => {
  // 正文（安静，什么装饰都没有）
  lines.forEach((_, i) => {
    setTimeout(() => { visibleLines.value = i + 1 }, 1000 + i * 1200)
  })

  const t = 1000 + lines.length * 1200   // 7000

  // 蛋糕安静搭建
  setTimeout(() => { builtStage.value = 1 }, t + 800)
  setTimeout(() => { builtStage.value = 2 }, t + 1100)
  setTimeout(() => { builtStage.value = 3 }, t + 1400)
  setTimeout(() => { builtStage.value = 4 }, t + 1700)
  setTimeout(() => { builtStage.value = 5 }, t + 2000)

  // 糖霜安静掉落
  setTimeout(() => { droppedRows.value = 1 }, t + 2800)
  setTimeout(() => { droppedRows.value = 2 }, t + 3700)
  setTimeout(() => { droppedRows.value = 3 }, t + 4800)

  // ——— 沉默两秒 ———
  // ——— 然后炸开 ———
  setTimeout(() => triggerCelebration(), t + 7000)

  // ♡
  setTimeout(() => {
    showHeart.value = true
    heartTimer = setInterval(() => {
      heartShaking.value = true
      heartHue.value = (heartHue.value + 45) % 360
      setTimeout(() => { heartShaking.value = false }, 500)
    }, 7500)
  }, t + 8800)
})

function triggerCelebration() {
  // 闪光
  whiteFlash.value = true

  // 闪光期间一切同时亮起
  setTimeout(() => {
    showEffects.value = true
    showBacklight.value = true
    cakeHighlight.value = true
    bgWarmth.value = 2
    warmGlow.value = true
    showBirthday.value = true

    // 粒子炸开
    const syms = ['✦','♡','✧','·','◇','❀','✦','♡','✧','·','◇','❀']
    const cols = ['#E890A8','#B098D8','#D0B898','#88B0D0','#D898B0','#A8C8A0']
    const ps = []
    for (let i = 0; i < 12; i++) {
      const a = (Math.PI * 2 * i) / 12 + (Math.random() - 0.5) * 0.3
      const r = 40 + Math.random() * 50
      ps.push({
        id: i, sym: syms[i], color: cols[i % cols.length],
        dx: (Math.cos(a) * r).toFixed(0),
        dy: (Math.sin(a) * r).toFixed(0),
      })
    }
    burstParticles.value = ps
    setTimeout(() => { burstParticles.value = [] }, 700)
  }, 80)
}

// ══ 散景（庆祝后才出现） ══
const bokeh = (() => {
  const arr = []
  const cols = [
    'rgba(230,210,190,0.08)','rgba(200,190,220,0.06)',
    'rgba(220,210,180,0.08)','rgba(210,200,195,0.07)',
    'rgba(235,215,195,0.08)',
  ]
  for (let i = 0; i < 12; i++) {
    arr.push({
      bg: cols[i % cols.length],
      size: (22 + Math.random() * 38).toFixed(0),
      left: (3 + Math.random() * 94).toFixed(0),
      delay: (Math.random() * 14).toFixed(1),
      dur: (10 + Math.random() * 8).toFixed(1),
    })
  }
  return arr
})()

// ══ 彩带（庆祝后才出现） ══
const confetti = (() => {
  const arr = []
  const syms = ['✦','❀','◇','✿','♡','▪','✦','❀','◆','✿','·','♡','◇','❀','✦','◆','♡','✿','❀','♡','✦','◇','✿','◆']
  const cols = ['#D8B8A0','#B8A8D0','#A8C8A0','#D8C890','#A0B8C8','#C8A8B0','#A8C0A0','#D0B898','#A0B0C8']
  for (let i = 0; i < 24; i++) {
    arr.push({
      sym: syms[i % syms.length], color: cols[i % cols.length],
      left: (2 + Math.random() * 96).toFixed(0),
      delay: (Math.random() * 4).toFixed(1),
      dur: (4 + Math.random() * 5).toFixed(1),
      size: (7 + Math.random() * 6).toFixed(0),
      opacity: (0.22 + Math.random() * 0.28).toFixed(2),
    })
  }
  return arr
})()

// ══ 生日装饰 ══
const sparkles = (() => {
  const syms = ['✦','♡','✧','·','✦','♡','✧','·']
  const cols = ['rgba(216,152,168,0.5)','rgba(176,152,216,0.45)']
  const arr = []
  for (let i = 0; i < 8; i++) {
    const a = (Math.PI * 2 * i) / 8 + (Math.random() - 0.5) * 0.5
    const r = 65 + Math.random() * 40
    arr.push({
      sym: syms[i], color: cols[i % 2],
      x: (Math.cos(a) * r).toFixed(0),
      y: (Math.sin(a) * r * 0.5).toFixed(0),
      delay: (i * 0.35).toFixed(1),
      size: 8 + Math.floor(Math.random() * 4),
    })
  }
  return arr
})()
</script>

<style scoped>
.birthday-page {
  display: flex; align-items: center; justify-content: center;
  min-height: 100vh; padding: 40px 24px;
  position: relative; overflow: hidden;
  transition: background 2.5s ease;
}
.warmth-0 { background: #FFFCF8; }
.warmth-1 { background: #FFF9F2; }
.warmth-2 { background: #FFF5EA; }

/* ━━━ 白闪 ━━━ */
.white-flash {
  position: fixed; inset: 0;
  background: radial-gradient(ellipse at 50% 45%, rgba(255,225,180,0.9), rgba(255,240,220,0.3) 70%);
  opacity: 0; pointer-events: none; z-index: 20;
}
.white-flash.active { animation: flashBang 0.45s ease-out forwards; }
@keyframes flashBang {
  0%  { opacity: 0; }
  15% { opacity: 0.5; }
  100%{ opacity: 0; }
}

/* ━━━ 散景 ━━━ */
.bokeh-layer { position: fixed; inset: 0; pointer-events: none; z-index: 0; overflow: hidden; }
.bokeh-dot {
  position: absolute; bottom: -10%; border-radius: 50%;
  animation: bokehFloat linear infinite; animation-fill-mode: both;
}
@keyframes bokehFloat {
  0%{transform:translateY(0);opacity:0}
  10%{opacity:1} 90%{opacity:1}
  100%{transform:translateY(-120vh);opacity:0}
}

/* ━━━ 彩带 ━━━ */
.confetti-layer { position: fixed; inset: 0; pointer-events: none; z-index: 0; overflow: hidden; }
.confetti-piece {
  position: absolute; top: -15px;
  animation: confettiFall linear infinite; animation-fill-mode: both;
}
@keyframes confettiFall {
  0%{transform:translateY(0) rotate(0deg);opacity:0}
  8%{opacity:var(--c-op,0.25)} 85%{opacity:var(--c-op,0.25)}
  100%{transform:translateY(108vh) rotate(540deg);opacity:0}
}

/* ━━━ 内容 ━━━ */
.birthday-content {
  display: flex; flex-direction: column; align-items: center;
  text-align: center; max-width: 320px; position: relative; z-index: 1;
}

/* ━━━ 旁白 ━━━ */
.b-line {
  font-size: 15px; line-height: 2; letter-spacing: 1.5px;
  color: var(--text, #4A3F35);
  margin: 0 0 4px; opacity: 0; transform: translateY(10px);
  transition: opacity 1s ease, transform 1s ease;
}
.b-line.visible { opacity: 1; transform: translateY(0); }

/* ━━━ 蛋糕区 ━━━ */
.cake-area {
  position: relative; display: flex; flex-direction: column;
  align-items: center; margin-top: 28px;
}

.cake-backlight {
  position: absolute; top: 40%; left: 50%;
  transform: translate(-50%, -50%);
  width: 220px; height: 200px; border-radius: 50%;
  background: radial-gradient(circle,
    rgba(255,225,195,0.22) 0%, rgba(255,215,185,0.1) 45%, transparent 70%
  );
  opacity: 0; transition: opacity 2s ease; pointer-events: none;
}
.cake-backlight.glow { opacity: 1; }

/* ━━━ 蛋糕 ━━━ */
.cake {
  display: flex; flex-direction: column; align-items: center;
  position: relative;
}

.ck-candle {
  display: flex; flex-direction: column; align-items: center;
  opacity: 0; transform: translateY(6px);
}
.ck-candle.pop-in { animation: candlePop 0.4s ease forwards; }
@keyframes candlePop { 0%{opacity:0;transform:translateY(6px)} 100%{opacity:1;transform:translateY(0)} }

.ck-flame {
  width: 6px; height: 8px; background: #F0D090;
  border-radius: 3px 3px 2px 2px;
  animation: flicker 1.5s ease-in-out infinite;
}
@keyframes flicker { 0%,100%{opacity:0.65;transform:scaleY(1)} 50%{opacity:1;transform:scaleY(1.15)} }

.ck-stick { width: 4px; height: 12px; background: #E8B8A8; margin-bottom: 1px; }

.ck-tier {
  margin-top: 1px; opacity: 0; transform: translateY(12px); border-radius: 2px;
  transition: background 1.2s ease;
}
.ck-tier.pop-in { animation: tierPop 0.45s cubic-bezier(0.34,1.56,0.64,1) forwards; }
@keyframes tierPop {
  0%{opacity:0;transform:translateY(12px) scaleY(0.8)}
  70%{opacity:1;transform:translateY(-2px) scaleY(1.02)}
  100%{opacity:1;transform:translateY(0) scaleY(1)}
}

.ck-1 { width: 56px;  height: 20px; background: rgba(220,212,198,0.12); }
.ck-2 { width: 88px;  height: 22px; background: rgba(218,208,195,0.12); }
.ck-3 { width: 120px; height: 24px; background: rgba(215,205,190,0.12); }

.ck-1.lit { background: rgba(220,212,198,0.22); }
.ck-2.lit { background: rgba(218,208,195,0.22); }
.ck-3.lit { background: rgba(215,205,190,0.22); }

.cake.highlight .ck-1 { background: rgba(220,212,198,0.35); }
.cake.highlight .ck-2 { background: rgba(218,208,195,0.35); }
.cake.highlight .ck-3 { background: rgba(215,205,190,0.35); }

.ck-plate {
  width: 140px; height: 5px; margin-top: 1px;
  opacity: 0; transform: translateY(8px); border-radius: 1px;
  background: rgba(210,200,185,0.12); transition: background 1.2s ease;
}
.ck-plate.pop-in { animation: tierPop 0.4s cubic-bezier(0.34,1.56,0.64,1) forwards; }
.cake.highlight .ck-plate { background: rgba(210,200,185,0.3); }

/* ━━━ 糖霜 ━━━ */
.frost-row {
  display: flex; justify-content: space-evenly; align-items: center;
  height: 6px; margin-top: -3px; margin-bottom: -3px;
  position: relative; z-index: 2;
}
.fr-1 { width: 88px; }
.fr-2 { width: 120px; }
.fr-3 { width: 140px; }

.frost-dot {
  width: 5px; height: 5px; border-radius: 50%;
  background: rgba(212,175,120,0.5);
  box-shadow: 0 0 3px rgba(212,175,120,0.18);
  opacity: 0; flex-shrink: 0;
}
.frost-dot.dropped {
  animation: dotDrop 0.45s cubic-bezier(0.34,1.56,0.64,1) forwards;
}
@keyframes dotDrop {
  0%{opacity:0;transform:translateY(-26px)}
  65%{opacity:1;transform:translateY(2px)}
  100%{opacity:1;transform:translateY(0)}
}

/* ━━━ 蜡烛热气 ━━━ */
.warm-ps {
  position: absolute; top: 0; left: 50%;
  transform: translateX(-50%); width: 60px; height: 0;
  pointer-events: none;
}
.warm-p {
  position: absolute; top: 0;
  width: 4px; height: 4px; border-radius: 50%;
  background: rgba(255,210,160,0.3);
  animation: warmFloat 3s ease-out infinite;
}
@keyframes warmFloat {
  0%{transform:translateY(0) scale(1);opacity:0.35}
  100%{transform:translateY(-55px) scale(0.4);opacity:0}
}

/* ━━━ 生日快乐（盖章弹入） ━━━ */
.bd-block {
  margin-top: 20px;
  opacity: 0; transform: scale(0.3);
}
.bd-block.visible {
  animation: stampIn 0.55s cubic-bezier(0.34,1.56,0.64,1) forwards;
}
@keyframes stampIn {
  0%  { opacity: 0; transform: scale(0.3); }
  45% { opacity: 1; transform: scale(1.1); }
  70% { transform: scale(0.96); }
  100%{ opacity: 1; transform: scale(1); }
}

.bd-wrap { position: relative; padding: 14px 24px; }

.bd-glow {
  position: absolute; inset: 0;
  display: flex; align-items: center; justify-content: center;
  font-size: 28px; font-weight: 700; letter-spacing: 5px;
  color: #D898A8; filter: blur(22px); opacity: 0.55;
  pointer-events: none;
}

.bd-text {
  position: relative; font-size: 28px; font-weight: 700; letter-spacing: 5px;
  background:
    linear-gradient(100deg,
      transparent 0%, rgba(255,255,255,0.4) 42%,
      rgba(255,255,255,0.65) 50%, rgba(255,255,255,0.4) 58%,
      transparent 100%
    ) no-repeat -100% 0 / 50% 100%,
    linear-gradient(90deg, #D898A8, #D898A8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: shimmerText 5s ease-in-out 1s infinite;
}
@keyframes shimmerText {
  0%,100%{background-position:-100% 0,0 0}
  50%{background-position:250% 0,0 0}
}

.bd-sparkle { position: absolute; pointer-events: none; animation: twinkle 2.8s ease-in-out infinite; }
@keyframes twinkle {
  0%,100%{opacity:0;transform:scale(0.4) translateY(0)}
  50%{opacity:0.75;transform:scale(1.1) translateY(-3px)}
}

.bd-particle {
  position: absolute; left: 50%; top: 50%;
  font-size: 11px; pointer-events: none;
  animation: pBurst 0.65s cubic-bezier(0.22,0.61,0.36,1) forwards;
}
@keyframes pBurst {
  0%{transform:translate(-50%,-50%) scale(1.3);opacity:1}
  100%{transform:translate(calc(-50% + var(--dx,0px)),calc(-50% + var(--dy,0px))) scale(0);opacity:0}
}

/* ━━━ ♡ ━━━ */
.heart-space {
  height: 56px; display: flex; align-items: center;
  justify-content: center; margin-top: 32px;
}
.heart-btn {
  background: none; border: none;
  font-size: 26px; color: #E890A8;
  cursor: pointer; padding: 8px;
  transition: filter 1.5s ease;
  -webkit-tap-highlight-color: transparent;
  font-family: inherit;
}
.heart-btn.shake { animation: heartJiggle 0.5s ease; }
@keyframes heartJiggle {
  0%,100%{transform:scale(1) rotate(0)}
  20%{transform:scale(1.25) rotate(-10deg)}
  40%{transform:scale(0.9) rotate(7deg)}
  60%{transform:scale(1.15) rotate(-5deg)}
  80%{transform:scale(0.95) rotate(3deg)}
}

.fade-enter-active { transition: opacity 1s ease; }
.fade-enter-from   { opacity: 0; }

.heart-wrap { display: flex; flex-direction: column; align-items: center; gap: 6px; }
.heart-hint { font-size: 11px; color: rgba(232,144,168,0.45); letter-spacing: 1px; animation: hintBreathe 2.5s ease-in-out infinite; }
@keyframes hintBreathe { 0%,100%{opacity:0.3} 50%{opacity:0.6} }
</style>