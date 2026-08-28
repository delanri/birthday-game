<template>
  <Transition name="ach">
    <div v-if="visible" class="ach-backdrop" @click.self="emit('close')">
      <div class="ach-panel">
        <div class="ach-header">
          <span class="ach-icon">🏆</span>
          <div class="ach-title">老板的账本</div>
          <div class="ach-sub">铺子里发生过的事</div>
        </div>
        <div class="ach-list">
          <div v-for="a in achievements" :key="a.id" class="ach-item">
            <div class="ach-stamp">{{ a.stamp }}</div>
            <div class="ach-info">
              <span class="ach-name">{{ a.name }}</span>
              <span class="ach-desc">{{ a.desc }}</span>
            </div>
          </div>
        </div>
        <div class="ach-footer">恭喜全部达成！昼老板很满意。</div>
        <button class="ach-close" @click="emit('close')">✕</button>
      </div>
    </div>
  </Transition>
</template>

<script setup>
defineProps({ visible: { type: Boolean, default: false } })
const emit = defineEmits(['close'])

const achievements = [
  { id: 1, stamp: '📦', name: '货架全部整理完毕', desc: '第一轮通关' },
  { id: 2, stamp: '👻', name: '连捣蛋鬼都抓住了', desc: '第二轮通关' },
  { id: 3, stamp: '🎁', name: '压箱底的也翻出来了', desc: '解锁隐藏照片' },
  { id: 4, stamp: '🔑', name: '老顾客了', desc: '回到铺子，灯还亮着' },
]
</script>

<style scoped>
.ach-backdrop { position: fixed; inset: 0; background: rgba(30,25,20,0.8); display: flex; align-items: center; justify-content: center; z-index: 500; }
.ach-panel { background: #FFF8F0; border: 3px solid var(--border-dark,#5D4E3C); padding: 24px 20px; max-width: 320px; width: 90%; display: flex; flex-direction: column; align-items: center; gap: 14px; box-shadow: 0 6px 0 0 var(--shadow,rgba(0,0,0,0.08)); }

.ach-header { text-align: center; display: flex; flex-direction: column; align-items: center; gap: 4px; }
.ach-icon { font-size: 32px; }
.ach-title { font-size: 15px; font-weight: 700; color: var(--text,#4A3F35); letter-spacing: 2px; }
.ach-sub { font-size: 11px; color: var(--text-light,#8B7E6A); letter-spacing: 1px; }

.ach-list { display: flex; flex-direction: column; gap: 8px; width: 100%; }
.ach-item { display: flex; align-items: center; gap: 12px; padding: 12px; border: 2px solid var(--border,#C8C0B0); background: #fff; }
.ach-stamp { font-size: 28px; flex-shrink: 0; width: 40px; text-align: center; }
.ach-info { display: flex; flex-direction: column; gap: 2px; }
.ach-name { font-size: 13px; font-weight: 700; color: var(--text,#4A3F35); letter-spacing: 0.5px; }
.ach-desc { font-size: 11px; color: var(--text-light,#8B7E6A); letter-spacing: 0.5px; }

.ach-footer { font-size: 11px; color: var(--accent,#D4A574); letter-spacing: 1px; padding: 6px 0; }

.ach-close { font-size: 11px; font-weight: 600; color: var(--text-light,#8B7E6A); background: none; border: none; cursor: pointer; padding: 4px 12px; letter-spacing: 1px; font-family: inherit; opacity: 0.6; -webkit-tap-highlight-color: transparent; }

.ach-enter-active { transition: opacity 0.3s ease; }
.ach-leave-active { transition: opacity 0.25s ease; }
.ach-enter-from, .ach-leave-to { opacity: 0; }
.ach-enter-active .ach-panel { animation: achPop 0.35s cubic-bezier(0.34,1.56,0.64,1); }
@keyframes achPop { from{opacity:0;transform:scale(0.85)} to{opacity:1;transform:scale(1)} }
</style>