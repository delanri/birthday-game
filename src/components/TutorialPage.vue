<template>
  <div class="tutorial-overlay">
    <div class="tutorial-box">

      <p class="tutorial-label">
        {{ round === 1 ? '「老板留的便签」' : '「老板又贴了一张」' }}
      </p>

      <div class="steps">

        <Transition name="step">
          <div v-if="visibleSteps >= 1" class="step-card">
            <div class="step-demo">
              <div class="demo-flip-row">
                <div class="mini-card flip-anim">
                  <div class="mini-inner">
                    <div class="mini-back">?</div>
                    <div class="mini-front mini-front--sky"></div>
                  </div>
                </div>
                <div class="mini-card flip-anim delay-1">
                  <div class="mini-inner">
                    <div class="mini-back">?</div>
                    <div class="mini-front mini-front--green"></div>
                  </div>
                </div>
              </div>
            </div>
            <p class="step-text">翻开卡片，看看货架上藏了什么~</p>
          </div>
        </Transition>

        <Transition name="step">
          <div v-if="visibleSteps >= 2" class="step-card">
            <div class="step-demo">
              <div class="demo-match">
                <div class="match-card match-slide-l mini-front--sky"></div>
                <div class="match-spark">✦</div>
                <div class="match-card match-slide-r mini-front--sky"></div>
              </div>
            </div>
            <p class="step-text">颜色相同的是一组，找到两张就收回来啦</p>
          </div>
        </Transition>

        <Transition name="step">
          <div v-if="visibleSteps >= 3" class="step-card">

            <template v-if="round === 1">
              <div class="step-demo">
                <div class="demo-collect">
                  <span
                    v-for="i in 4" :key="i"
                    class="collect-pip"
                    :class="'pip-pop-' + i"
                  >●</span>
                </div>
              </div>
              <p class="step-text">集齐四份，货架就满了</p>
            </template>

            <template v-else>
              <div class="step-demo">
                <div class="demo-runaway">
                  <div class="run-card-demo">
                    <div class="run-legs-demo">
                      <span class="run-leg-demo"></span>
                      <span class="run-leg-demo"></span>
                    </div>
                  </div>
                </div>
              </div>
              <p class="step-text">不过……这次货架上的东西有点不听话</p>
            </template>

          </div>
        </Transition>
      </div>

      <Transition name="fade-up">
        <button
          v-if="showBtn"
          class="pixel-btn tut-start-btn"
          @click="emit('done')"
        >
          {{ round === 1 ? '开始寻宝' : '再次挑战' }}
        </button>
      </Transition>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  round: { type: Number, default: 1 },
})

const emit = defineEmits(['done'])

const visibleSteps = ref(0)
const showBtn = ref(false)

onMounted(() => {
  setTimeout(() => { visibleSteps.value = 1 }, 300)
  setTimeout(() => { visibleSteps.value = 2 }, 1200)
  setTimeout(() => { visibleSteps.value = 3 }, 2100)
  setTimeout(() => { showBtn.value = true }, 3000)
})
</script>

<style scoped>
.tutorial-overlay {
  position: fixed; inset: 0;
  display: flex; align-items: center; justify-content: center;
  background: rgba(0, 0, 0, 0.5);
  z-index: 50; padding: 24px;
}

.tutorial-box {
  background: var(--bg, #FFFCF7);
  border: 3px solid var(--border-dark, #5D4E3C);
  padding: 28px 20px; max-width: 320px; width: 100%;
  display: flex; flex-direction: column;
  align-items: center; gap: 18px;
  box-shadow: 0 6px 0 0 var(--shadow, rgba(0,0,0,0.08));
}

.tutorial-label {
  font-size: 14px; color: var(--text-light, #8B7E6A);
  letter-spacing: 2px; text-align: center; margin: 0;
}

.steps {
  display: flex; flex-direction: column;
  gap: 14px; width: 100%;
}

.step-card {
  border: 2px solid var(--border, #C8C0B0);
  padding: 14px 12px 10px; text-align: center;
  background: var(--card-back, #FAF6F0);
}

.step-demo {
  height: 56px; display: flex;
  align-items: center; justify-content: center;
  margin-bottom: 6px;
}

.step-text {
  font-size: 13px; color: var(--text, #4A3F35);
  letter-spacing: 1px; margin: 0;
}

/* ── Demo 1 翻牌 ── */
.demo-flip-row { display: flex; gap: 14px; }

.mini-card { width: 36px; height: 44px; perspective: 200px; }
.mini-inner {
  width: 100%; height: 100%;
  position: relative; transform-style: preserve-3d;
}
.flip-anim .mini-inner { animation: miniFlip 2.8s ease-in-out infinite; }
.flip-anim.delay-1 .mini-inner { animation-delay: 0.5s; }

.mini-back, .mini-front {
  position: absolute; inset: 0; backface-visibility: hidden;
  display: flex; align-items: center; justify-content: center;
  border: 2px solid var(--border, #C8C0B0);
}
.mini-back { background: var(--card-back, #FAF6F0); color: var(--border, #C8C0B0); font-size: 12px; }
.mini-front { transform: rotateY(180deg); font-size: 18px; }
.mini-front--sky   { background: #C0D8F0; border-color: #89B4D4; }
.mini-front--green { background: #C8E4A8; border-color: #8CB868; }

@keyframes miniFlip {
  0%, 25%  { transform: rotateY(0deg); }
  35%, 75% { transform: rotateY(180deg); }
  85%, 100%{ transform: rotateY(0deg); }
}

/* ── Demo 2 配对 ── */
.demo-match { display: flex; align-items: center; position: relative; }
.match-card {
  width: 36px; height: 44px;
  border: 2px solid #89B4D4;
  display: flex; align-items: center; justify-content: center;
  font-size: 18px;
}
.match-slide-l { animation: matchSlideL 2.8s ease-in-out infinite; }
.match-slide-r { animation: matchSlideR 2.8s ease-in-out infinite; }
.match-spark {
  font-size: 14px; color: var(--accent, #D4A574);
  width: 16px; text-align: center;
  animation: sparkBurst 2.8s ease-in-out infinite;
}

@keyframes matchSlideL {
  0%, 20% { transform: translateX(0); opacity: 1; }
  40%     { transform: translateX(6px); opacity: 1; }
  55%     { transform: translateX(6px); opacity: 0.3; }
  70%, 100% { transform: translateX(0); opacity: 1; }
}
@keyframes matchSlideR {
  0%, 20% { transform: translateX(0); opacity: 1; }
  40%     { transform: translateX(-6px); opacity: 1; }
  55%     { transform: translateX(-6px); opacity: 0.3; }
  70%, 100% { transform: translateX(0); opacity: 1; }
}
@keyframes sparkBurst {
  0%, 35%  { opacity: 0; transform: scale(0.4); }
  45%      { opacity: 1; transform: scale(1.5); }
  58%      { opacity: 0; transform: scale(0.4); }
  100%     { opacity: 0; }
}

/* ── Demo 3a 收集 ── */
.demo-collect { display: flex; gap: 14px; }
.collect-pip { font-size: 18px; color: var(--border, #C8C0B0); opacity: 0.3; }
.pip-pop-1 { animation: pipGlow 3s ease 0s    infinite; }
.pip-pop-2 { animation: pipGlow 3s ease 0.35s infinite; }
.pip-pop-3 { animation: pipGlow 3s ease 0.7s  infinite; }
.pip-pop-4 { animation: pipGlow 3s ease 1.05s infinite; }
@keyframes pipGlow {
  0%, 12%  { opacity: 0.3; color: var(--border, #C8C0B0); transform: scale(1); }
  22%      { opacity: 1;   color: var(--accent, #D4A574); transform: scale(1.35); }
  36%, 100%{ opacity: 1;   color: var(--accent, #D4A574); transform: scale(1); }
}

/* ── Demo 3b 跑路 ── */
.demo-runaway { position: relative; }
.run-card-demo {
  position: relative; width: 36px; height: 44px;
  border: 2px solid #8CB868; background: #C8E4A8;
  display: flex; align-items: center; justify-content: center;
  animation: demoRun 2.4s ease-in-out infinite;
}
.run-icon-demo { font-size: 18px; }
.run-legs-demo {
  position: absolute; bottom: -8px; left: 50%;
  transform: translateX(-50%); display: flex; gap: 6px;
}
.run-leg-demo {
  display: block; width: 3px; height: 6px;
  background: var(--text-light, #8B7E6A); border-radius: 0 0 1px 1px;
}
.run-leg-demo:first-child  { animation: legStep 0.3s step-end infinite; }
.run-leg-demo:last-child   { animation: legStep 0.3s step-end 0.15s infinite; }
@keyframes legStep { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-4px); } }
@keyframes demoRun {
  0%, 8%    { transform: translateX(0); }
  45%       { transform: translateX(36px); }
  55%       { transform: translateX(36px) rotate(5deg); }
  90%, 100% { transform: translateX(0); }
}

/* ── 过渡 & 按钮 ── */
.step-enter-active { animation: stepIn 0.4s ease; }
@keyframes stepIn {
  from { opacity: 0; transform: translateY(14px); }
  to   { opacity: 1; transform: translateY(0); }
}

.fade-up-enter-active { animation: fadeUp 0.4s ease; }
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.tut-start-btn { margin-top: 4px; }
</style>