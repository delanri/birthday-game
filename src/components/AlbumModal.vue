<template>
  <Transition name="album">
    <div v-if="visible" class="album-backdrop" @click.self="emit('close')">
      <div class="album-panel">

        <div class="album-header">
          <span class="album-deco">✦</span>
          <div class="album-title">回忆相册</div>
          <div class="album-sub">铺子里收藏的照片</div>
          <span class="album-deco">✦</span>
        </div>

        <div class="album-grid">
          <div v-for="p in photos" :key="p.id"
            class="album-card"
            :class="{ unlocked: unlockedGroups.includes(p.id) }"
            :style="{ '--card-accent': p.accent }"
            @click="unlockedGroups.includes(p.id) && openPhoto(p)">
            <div class="card-img-wrap">
              <img v-if="unlockedGroups.includes(p.id)" class="card-img" :src="getPhoto(p.img)" :alt="p.name">
              <span v-else class="card-lock">🔒</span>
            </div>
            <div class="card-bottom">
              <span class="card-name">{{ unlockedGroups.includes(p.id) ? p.name : '???' }}</span>
              <span v-if="unlockedGroups.includes(p.id)" class="card-caption">{{ p.caption }}</span>
            </div>
          </div>
        </div>

        <div class="album-divider">
          <span class="divider-dot">·</span>
          <span class="divider-dot">·</span>
          <span class="divider-dot">·</span>
        </div>

        <div class="hidden-card"
          :class="{ unlocked: hiddenUnlocked }"
          @click="hiddenUnlocked && openPhoto(hiddenPhoto)">
          <div class="card-img-wrap hidden-img-wrap">
            <img v-if="hiddenUnlocked" class="card-img" :src="getPhoto(hiddenPhoto.img)" alt="隐藏照片">
            <span v-else class="hidden-q">？</span>
          </div>
          <div class="card-bottom">
            <span class="card-name">{{ hiddenUnlocked ? '压箱底的宝贝' : '？？？' }}</span>
            <span v-if="hiddenUnlocked" class="card-caption">{{ hiddenPhoto.caption }}</span>
          </div>
        </div>

        <button class="album-close" @click="emit('close')">✕</button>
      </div>

      <Transition name="photo-pop">
        <div v-if="viewing" class="photo-viewer" @click.self="viewing = null">
          <div class="photo-frame">
            <img class="photo-full" :src="getPhoto(viewing.img)" :alt="viewing.caption">
            <span class="photo-caption">{{ viewing.caption }}</span>
          </div>
        </div>
      </Transition>
    </div>
  </Transition>
</template>

<script setup>
import { ref } from 'vue'
defineProps({
  visible: { type: Boolean, default: false },
  unlockedGroups: { type: Array, default: () => [] },
  hiddenUnlocked: { type: Boolean, default: false },
})
const emit = defineEmits(['close'])

function getPhoto(name) {
  return new URL(`../assets/photos/${name}.jpg`, import.meta.url).href
}

const viewing = ref(null)

const photos = [
  { id: 0, img: 'photo-sky', name: '光遇', caption: '星空下的冒险', accent: '#89B4D4' },
  { id: 1, img: 'photo-genshin', name: '原神', caption: '竹筏上的旅途', accent: '#8CB868' },
  { id: 2, img: 'photo-stardew', name: '星露谷', caption: '花舞节的回忆', accent: '#D898B0' },
  { id: 3, img: 'photo-overcooked', name: '胡闹厨房', caption: '三星大厨！', accent: '#D0B870' },
]
const hiddenPhoto = { id: 'hidden', img: 'photo-hidden', caption: '压箱底的宝贝' }

function openPhoto(p) { viewing.value = p }
</script>

<style scoped>
.album-backdrop { position: fixed; inset: 0; background: rgba(25,20,16,0.85); display: flex; align-items: center; justify-content: center; z-index: 500; }
.album-panel { background: #FFF8F0; border: 3px solid var(--border-dark,#5D4E3C); padding: 22px 18px; max-width: 340px; width: 92%; display: flex; flex-direction: column; align-items: center; gap: 14px; box-shadow: 0 6px 0 0 var(--shadow,rgba(0,0,0,0.08)); max-height: 88vh; overflow-y: auto; }

.album-header { text-align: center; display: flex; flex-direction: column; align-items: center; gap: 4px; }
.album-deco { font-size: 10px; color: var(--accent,#D4A574); letter-spacing: 8px; }
.album-title { font-size: 16px; font-weight: 700; color: var(--text,#4A3F35); letter-spacing: 3px; }
.album-sub { font-size: 11px; color: var(--text-light,#8B7E6A); letter-spacing: 1px; }

.album-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; width: 100%; }

.album-card { display: flex; flex-direction: column; border: 2px solid var(--border,#C8C0B0); background: #fff; cursor: default; overflow: hidden; transition: border-color 0.3s, transform 0.15s; }
.album-card.unlocked { border-color: var(--card-accent, #C8C0B0); cursor: pointer; }
.album-card.unlocked:active { transform: scale(0.97); }

.card-img-wrap { width: 100%; aspect-ratio: 4/3; overflow: hidden; display: flex; align-items: center; justify-content: center; background: #F5F0EA; }
.card-img { width: 100%; height: 100%; object-fit: cover; }
.card-lock { font-size: 24px; opacity: 0.2; }

.card-bottom { padding: 8px 8px 10px; display: flex; flex-direction: column; gap: 2px; }
.card-name { font-size: 12px; font-weight: 700; color: var(--text,#4A3F35); letter-spacing: 0.5px; }
.card-caption { font-size: 10px; color: var(--text-light,#8B7E6A); letter-spacing: 0.5px; }

.album-divider { display: flex; gap: 8px; justify-content: center; padding: 2px 0; }
.divider-dot { font-size: 14px; color: var(--border,#C8C0B0); }

.hidden-card { width: 100%; display: flex; align-items: center; gap: 14px; padding: 10px; border: 2px dashed var(--border,#C8C0B0); background: #fff; cursor: default; transition: border-color 0.3s; }
.hidden-card.unlocked { border-style: solid; border-color: var(--accent,#D4A574); cursor: pointer; }
.hidden-card.unlocked:active { transform: scale(0.98); }
.hidden-img-wrap { width: 72px; height: 54px; flex-shrink: 0; aspect-ratio: auto; }
.hidden-q { font-size: 22px; font-weight: 700; color: var(--border,#C8C0B0); opacity: 0.3; }

.album-close { font-size: 11px; font-weight: 600; color: var(--text-light,#8B7E6A); background: none; border: none; cursor: pointer; padding: 4px 12px; letter-spacing: 1px; font-family: inherit; opacity: 0.6; -webkit-tap-highlight-color: transparent; }

/* ── 大图查看 ── */
.photo-viewer { position: fixed; inset: 0; background: rgba(20,16,12,0.92); display: flex; align-items: center; justify-content: center; z-index: 510; cursor: pointer; }
.photo-frame { background: #fff; border: 4px solid var(--border,#C8C0B0); padding: 10px 10px 14px; max-width: 88vw; display: flex; flex-direction: column; align-items: center; gap: 10px; box-shadow: 0 6px 0 0 var(--shadow,rgba(0,0,0,0.08)); cursor: default; }
.photo-full { max-width: 88vw; width: 100%; border-radius: 1px; }
.photo-caption { font-size: 13px; font-weight: 600; color: var(--text-light,#8B7E6A); letter-spacing: 1px; }

/* ── 过渡 ── */
.album-enter-active { transition: opacity 0.3s ease; }
.album-leave-active { transition: opacity 0.25s ease; }
.album-enter-from, .album-leave-to { opacity: 0; }
.album-enter-active .album-panel { animation: panelPop 0.35s cubic-bezier(0.34,1.56,0.64,1); }
@keyframes panelPop { from{opacity:0;transform:scale(0.85)} to{opacity:1;transform:scale(1)} }
.photo-pop-enter-active { animation: photoPop 0.3s cubic-bezier(0.34,1.56,0.64,1); }
.photo-pop-leave-active { transition: opacity 0.2s ease; }
.photo-pop-leave-to { opacity: 0; }
@keyframes photoPop { from{opacity:0;transform:scale(0.8)} to{opacity:1;transform:scale(1)} }
</style>