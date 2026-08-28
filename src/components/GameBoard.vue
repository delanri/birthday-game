<template>
  <div class="game-page" :class="{ 'round2-deco': round === 2 }">

    <TutorialPage v-if="phase === 'tutorial'" :round="round" @done="onTutorialDone" />

    <!-- ══ 背景装饰层 ══ -->
    <div class="game-deco-layer">
      <Transition name="deco">
        <div v-if="activeGameDeco === 'candle'" class="deco" key="candle">
          <span class="g-glow g-glow-1"></span><span class="g-glow g-glow-2"></span>
          <span class="g-glow g-glow-3"></span><span class="g-glow g-glow-4"></span><span class="g-glow g-glow-5"></span>
        </div>
      </Transition>
      <Transition name="deco">
        <div v-if="activeGameDeco === 'water'" class="deco" key="water">
          <span class="g-wave g-wave-1"></span><span class="g-wave g-wave-2"></span><span class="g-wave g-wave-3"></span>
        </div>
      </Transition>
      <Transition name="deco">
        <div v-if="activeGameDeco === 'petals'" class="deco" key="petals">
          <span v-for="(p,i) in gamePetals" :key="i" class="g-petal"
            :style="{ left:p.left+'%', fontSize:p.size+'px', animationDelay:p.delay+'s', animationDuration:p.dur+'s' }">{{ p.sym }}</span>
        </div>
      </Transition>
      <Transition name="deco">
        <div v-if="activeGameDeco === 'receipt'" class="deco" key="receipt">
          <div class="g-receipt g-receipt-1"><span class="g-rline"></span><span class="g-rline short"></span><span class="g-rline"></span><span class="g-rline medium"></span><span class="g-rline short"></span><span class="g-rstain"></span><span class="g-rstain stain-2"></span></div>
          <div class="g-receipt g-receipt-2"><span class="g-rline"></span><span class="g-rline medium"></span><span class="g-rline"></span><span class="g-rline short"></span><span class="g-rstain stain-3"></span></div>
        </div>
      </Transition>
    </div>

    <div class="game-container">
      <div class="game-header">
        <span v-if="phase === 'preview'" class="header-text">记住它们！ {{ countdown }}s</span>
        <span v-else-if="phase === 'playing'" class="header-text">{{ round === 2 ? '第二轮 · 小心捣蛋鬼！' : '翻开卡片，找到配对 ♪' }}</span>
        <span v-else-if="phase === 'roundDone'" class="header-text">完成！</span>
      </div>

      <div class="collection-bar" :class="{ celebrating }">
        <div v-for="g in GROUPS" :key="g.id" class="collect-icon" :class="{ half: groupProg[g.id]===1, done: groupProg[g.id]===2 }">
          <span class="collect-emoji">{{ g.emoji }}</span>
          <span class="collect-label" :class="{ visible: groupProg[g.id]===2 }" :style="{ color: g.accent }">{{ g.name }}</span>
        </div>
      </div>

      <div class="card-grid" :class="{ 'round-2': round === 2 }">
        <div v-for="(card, idx) in cards" :key="idx" class="card-cell">
          <div v-if="isMatched(idx)" class="card-slot-empty"></div>
          <div v-else class="card-wrapper"
            :class="{
              'is-shaking': shakingIdx===idx,
              'is-crawling': !!crawlAnims[idx],
              'has-legs': legSet.has(idx),
              'is-vanishing': vanishing.has(cards[idx].id),
            }"
            :style="wrapperStyle(idx)" @click="onCardClick(idx)">
            <div class="card-inner" :class="{ 'is-flipped': isFlipped(idx) }">
              <div class="card-back"><span class="back-q">{{ round===2?'!':'?' }}</span></div>
              <div class="card-front" :style="{ background: getGroup(card.groupId).color, borderColor: getGroup(card.groupId).accent }">
                <img class="card-icon-img" :src="getCardImg(card.img)" :alt="card.img">
                <span v-if="phase==='preview'" class="card-group-tag" :style="{ color: getGroup(card.groupId).accent }">{{ getGroup(card.groupId).name }}</span>
              </div>
            </div>
            <div v-if="legSet.has(idx)" class="pixel-legs"><span class="p-leg"></span><span class="p-leg"></span></div>
          </div>
        </div>
      </div>

      <div class="game-footer">
        
      </div>
      <div v-if="phase === 'roundDone' && !showPopup" class="done-actions">
        <button v-if="round === 1" class="pixel-btn" @click="startRound2">再来一次 ✨</button>
        <button v-else class="pixel-btn" @click="emit('complete')">查看最后的惊喜 ♡</button>
      </div>
    </div>

    <!-- ══ 浮层 ══ -->
    <div class="overlay-root">
      <button class="album-fixed" :class="{ unlocked: albumUnlocked }" :disabled="!albumUnlocked"
        @click="albumUnlocked && (showAlbum = true)">{{ albumUnlocked ? '📷' : '🔒' }}</button>

      <Transition name="mischief">
        <div v-if="mischiefMsg" class="mischief-toast">{{ mischiefMsg }}</div>
      </Transition>

      <Transition name="toast">
        <div v-if="groupToast" class="group-toast">
          <span class="toast-icon">{{ getGroup(groupToast.gid).emoji }}</span>
          <span class="toast-text">{{ getGroup(groupToast.gid).label }}</span>
          <span class="toast-sub" :style="{ color: getGroup(groupToast.gid).accent }">· {{ getGroup(groupToast.gid).name }}</span>
        </div>
      </Transition>

      <Transition name="overlay-fade"><div v-if="showPopup" class="overlay-bg"></div></Transition>

      <Transition name="cele-fade">
        <div v-if="showCelebration" class="cele-layer">
          <div class="cele-spread" :class="{ intense: round===2 }"></div>
          <div class="cele-ambient" :class="{ active: glowActive, intense: round===2 }"></div>
          <span v-for="p in burstParticles" :key="'bp'+p.id" class="burst-particle"
            :style="{ fontSize:p.size+'px', color:p.color, animationDelay:p.delay+'s', '--bp-dx':p.dx+'px', '--bp-dy':p.dy+'px' }">{{ p.sym }}</span>
          <span v-for="p in burstParticles2" :key="'bp2'+p.id" class="burst-particle wave2"
            :style="{ fontSize:p.size+'px', color:p.color, animationDelay:p.delay+'s', '--bp-dx':p.dx+'px', '--bp-dy':p.dy+'px' }">{{ p.sym }}</span>
          <span v-for="(s,i) in starDust" :key="'sd'+i" class="star-dust"
            :style="{ left:s.left+'%', fontSize:s.size+'px', color:s.color, animationDelay:s.delay+'s', animationDuration:s.dur+'s', '--sd-op':s.opacity }">{{ s.sym }}</span>
        </div>
      </Transition>

      <Transition name="popup-pop">
        <div v-if="showPopup" class="popup-layer">
          <div v-if="showPopup.type === 'round1'" class="overlay-box">
            <div class="popup-emoji">🎉</div>
            <div class="popup-title">货架上的东西都找到主人了</div>
            <div class="popup-sub">柜台底下好像还压着什么……</div>
            <div class="popup-hint">✨ 再翻一次货架？</div>
            <div class="popup-btns">
              <button class="pixel-btn pixel-btn--ghost" @click="dismissPopup()">先歇歇</button>
              <button class="pixel-btn" @click="dismissPopup(); startRound2()">再来一次！</button>
            </div>
          </div>
          <div v-if="showPopup.type === 'round2'" class="overlay-box">
            <div class="popup-emoji">🏆</div>
            <div class="popup-title">铺子里的东西，一件不少。</div>
            <div class="popup-sub">连压箱底的都翻出来了。</div>
            <button class="pixel-btn" @click="dismissPopup(); hiddenReveal = true">查看隐藏照片 ♡</button>
          </div>
        </div>
      </Transition>
    </div>

    <!-- ══ 隐藏照片揭晓 ══ -->
    <Transition name="reveal-fade">
      <div v-if="hiddenReveal" class="reveal-overlay" @click.self="onRevealClose">
       <div class="reveal-frame">
  <img class="reveal-photo" :src="getHiddenPhoto()" alt="隐藏照片">
  <span class="reveal-text">压箱底的宝贝</span>
</div>
        <span class="reveal-hint">轻触空白处继续</span>
      </div>
    </Transition>

    <AlbumModal :visible="showAlbum" :unlocked-groups="albumGroups" :hidden-unlocked="false" @close="showAlbum = false" />
  </div>
</template>

<script setup>
import { ref, computed, nextTick, onBeforeUnmount } from 'vue'
import TutorialPage from './TutorialPage.vue'
import AlbumModal from './AlbumModal.vue'

const props = defineProps({ colorStage: { type: Number, default: 2 } })
const emit = defineEmits(['complete', 'roundChange'])

const GROUPS = [
  { id: 0, name: '光遇',     imgs: ['sky-1','sky-2','sky-3','sky-4'],                          emoji: '🕯️', color: '#C0D8F0', accent: '#89B4D4', label: '蜡烛亮了一盏。' },
  { id: 1, name: '原神',     imgs: ['genshin-1','genshin-2','genshin-3','genshin-4'],           emoji: '🍃', color: '#C8E4A8', accent: '#8CB868', label: '竹竿上还有水渍。' },
  { id: 2, name: '星露谷',   imgs: ['stardew-1','stardew-2','stardew-3','stardew-4'],           emoji: '🥬', color: '#F0C8D8', accent: '#D898B0', label: '干花好像还有香味。' },
  { id: 3, name: '胡闹厨房', imgs: ['overcooked-1','overcooked-2','overcooked-3','overcooked-4'], emoji: '🍳', color: '#F0DCA8', accent: '#D0B870', label: '收据上的墨水还没干。' },
]

function getCardImg(name) {
  return new URL(`../assets/cards/${name}.png`, import.meta.url).href
}

function getHiddenPhoto() {
  return new URL('../assets/photos/photo-hidden.jpg', import.meta.url).href
}

const SHAKE_MSGS = ['这个记忆害羞了', '还没准备好被看到…', '再翻一次试试？']
const RUN_MSGS   = ['记忆从货架上溜走了', '有东西换了位置…', '货架好像动了一下']
const CATCH_MSGS = ['抓到一个想逃的', '乖乖回来了', '差点跑掉']

function shuffleArr(arr) { const a=[...arr]; for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]]} return a }
function pick(arr) { return arr[Math.floor(Math.random() * arr.length)] }
function gridPos(idx) { return { row: Math.floor(idx/4), col: idx%4 } }
function calcOffset(from, to) {
  const f=gridPos(from), t=gridPos(to)
  return { x:`calc(${t.col-f.col} * (100% + 8px))`, y:`calc(${t.row-f.row} * (100% + 8px))` }
}

// ★ 短距离偏移（crawl 用，只走 35%）
function calcPartialOffset(from, to) {
  const f = gridPos(from), t = gridPos(to), frac = 0.35
  return {
    x: `calc(${((t.col-f.col)*frac).toFixed(2)} * (100% + 8px))`,
    y: `calc(${((t.row-f.row)*frac).toFixed(2)} * (100% + 8px))`,
  }
}

function makeCards() {
  const c=[]; GROUPS.forEach(g=>{g.imgs.forEach((img,i)=>{c.push({id:`${g.id}-${i}`,groupId:g.id,img})})}); return shuffleArr(c)
}
function nearestEmpty(fromIdx, emptyList) {
  if(!emptyList.length) return -1; const f=gridPos(fromIdx); let best=-1,bd=Infinity
  for(const e of emptyList){const t=gridPos(e),d=Math.abs(t.row-f.row)+Math.abs(t.col-f.col); if(d<bd){bd=d;best=e}}
  return best
}

const timers=[]; function later(fn,ms){const t=setTimeout(fn,ms);timers.push(t);return t}
let cdInterval=null; onBeforeUnmount(()=>{timers.forEach(clearTimeout);clearInterval(cdInterval)})

const phase=ref('tutorial'), round=ref(1), cards=ref(makeCards())
const flipped=ref([]), matched=ref(new Set()), groupProg=ref({0:0,1:0,2:0,3:0})
const locked=ref(false), countdown=ref(2)
const flipCount=ref(0), mischiefMsg=ref(''), shakingIdx=ref(-1)
const cardAnims=ref({}), legSet=ref(new Set()), crawlAnims=ref({})
const skipTransition=ref(false), shakeGuard=ref(false)
const showPopup=ref(null), groupToast=ref(null), albumUnlocked=ref(false), activeGameDeco=ref(null)
const showCelebration=ref(false), glowActive=ref(false)
const burstParticles=ref([]), burstParticles2=ref([]), celebrating=ref(false)
const showAlbum=ref(false), hiddenReveal=ref(false)
const vanishing = ref(new Set())  // ★ 消失动画中的卡牌 ID

const albumGroups = computed(() => albumUnlocked.value ? [0,1,2,3] : [])

const gamePetals=(()=>{const a=[],s=['❀','✿','❁','·','✾','❀'];for(let i=0;i<6;i++)a.push({sym:s[i],left:(10+Math.random()*80).toFixed(0),delay:(Math.random()*6).toFixed(1),dur:(6+Math.random()*5).toFixed(1),size:(10+Math.random()*8).toFixed(0)});return a})()
const starDust=(()=>{const a=[],s=['✦','·','✧','·','✦','✧','·','✦','·','✧','✦','·','✧','·','✦','✧','·','✦','✧','·','✦','·','✧','✦'],c=['#FFB8C8','#90C8E8','#A0D898','#FFD488','#E8A0C0','#C0A8E8','#A8D8A8','#FFD0A0'];for(let i=0;i<24;i++)a.push({sym:s[i],left:(2+Math.random()*96).toFixed(0),delay:(Math.random()*5).toFixed(1),dur:(3.5+Math.random()*4).toFixed(1),size:(7+Math.random()*9).toFixed(0),color:c[i%c.length],opacity:(0.55+Math.random()*0.35).toFixed(2)});return a})()

const remaining=computed(()=>cards.value.filter(c=>!matched.value.has(c.id)).length)
const mischiefChance=computed(()=>remaining.value<=6?1.0:Math.min(1.0,0.4+Math.floor(flipCount.value/2)*0.1))

function getGroup(gid){return GROUPS.find(g=>g.id===gid)}
function isFlipped(idx){return phase.value==='preview'||flipped.value.includes(cards.value[idx].id)}
function isMatched(idx){return matched.value.has(cards.value[idx].id)}
function unmatchedIdxs(){return cards.value.map((_,i)=>i).filter(i=>!isMatched(i))}
function emptyIdxs(){return cards.value.map((_,i)=>i).filter(i=>isMatched(i))}

function onTutorialDone(){phase.value='preview';countdown.value=2;clearInterval(cdInterval);cdInterval=setInterval(()=>{countdown.value--;if(countdown.value<=0){clearInterval(cdInterval);phase.value='playing'}},1000)}

// ══ 翻牌核心 ══
function onCardClick(idx, catchRedirect = false) {
  if (phase.value !== 'playing' || locked.value) return
  const card = cards.value[idx]
  if (matched.value.has(card.id) || flipped.value.includes(card.id)) return

  // ★ 点击爬行中的卡：停下来，原地翻（短距离不需要走到别的格子）
  if (crawlAnims.value[idx]) {
    const ca = { ...crawlAnims.value }; delete ca[idx]; crawlAnims.value = ca
    const ls = new Set(legSet.value); ls.delete(idx); legSet.value = ls
    mischiefMsg.value = pick(CATCH_MSGS)
    later(() => { mischiefMsg.value = '' }, 1500)
    // 继续往下走正常翻牌流程
  }

  const isSecond = flipped.value.length === 1
  if (round.value === 2 && !catchRedirect) flipCount.value++
  if (round.value === 2 && isSecond && !catchRedirect) {
    if (!shakeGuard.value && Math.random() < mischiefChance.value) { shakeGuard.value = true; doShakeBack(idx); return }
    shakeGuard.value = false
  }
  flipped.value = [...flipped.value, card.id]
  if (!isSecond) return

  locked.value = true
  const [idA, idB] = flipped.value
  const a = cards.value.findIndex(c => c.id === idA), b = cards.value.findIndex(c => c.id === idB)
  const cardA = cards.value[a], cardB = cards.value[b]

  if (cardA.groupId === cardB.groupId) {
    // ★ 配对成功：先展示 → 消失动画 → 真正移除
    later(() => {
      vanishing.value = new Set([cardA.id, cardB.id])

      later(() => {
        vanishing.value = new Set()
        const nm = new Set(matched.value); nm.add(cardA.id); nm.add(cardB.id); matched.value = nm
        flipped.value = []
        const gp = { ...groupProg.value }; gp[cardA.groupId]++; groupProg.value = gp

        if (gp[cardA.groupId] === 2) {
          groupToast.value = { gid: cardA.groupId }
          activeGameDeco.value = ['candle','water','petals','receipt'][cardA.groupId]
          later(() => { groupToast.value = null }, 3000)
          if (GROUPS.every(g => gp[g.id] === 2)) { activeGameDeco.value = null; finishRound() }
          else { locked.value = false; if (round.value === 2) afterAction() }
        } else { locked.value = false; if (round.value === 2) afterAction() }
      }, 420)
    }, 550)
  } else {
    later(() => { flipped.value = []; locked.value = false; if (round.value === 2) afterAction() }, 800)
  }
}

function finishRound() {
  later(() => {
    phase.value = 'roundDone'; triggerCelebration(round.value)
    if (round.value === 1) { albumUnlocked.value = true; showPopup.value = { type: 'round1' } }
    else { showPopup.value = { type: 'round2' } }
  }, 500)
}

function startRound2() {
  round.value=2; emit('roundChange',2); cards.value=makeCards()
  flipped.value=[]; matched.value=new Set(); groupProg.value={0:0,1:0,2:0,3:0}
  locked.value=false; flipCount.value=0; shakeGuard.value=false
  showPopup.value=null; mischiefMsg.value=''; shakingIdx.value=-1
  cardAnims.value={}; legSet.value=new Set(); crawlAnims.value={}
  activeGameDeco.value=null; showCelebration.value=false; glowActive.value=false
  burstParticles.value=[]; burstParticles2.value=[]; celebrating.value=false
  showAlbum.value=false; hiddenReveal.value=false; vanishing.value=new Set()
  phase.value='tutorial'
}

function dismissPopup(){showPopup.value=null;showCelebration.value=false;glowActive.value=false;burstParticles.value=[];burstParticles2.value=[];celebrating.value=false}
function onRevealClose(){hiddenReveal.value=false;emit('complete')}

function doShakeBack(idx) {
  const card=cards.value[idx]; locked.value=true
  flipped.value=[...flipped.value,card.id]
  later(()=>{shakingIdx.value=idx
    later(()=>{shakingIdx.value=-1;flipped.value=flipped.value.filter(id=>id!==card.id)
      locked.value=false;mischiefMsg.value=pick(SHAKE_MSGS);later(()=>{mischiefMsg.value=''},1500)
    },600)
  },500)
}

function afterAction(){later(()=>maybeRunaway(),350);later(()=>maybeCrawl(),900)}

function maybeRunaway() {
  if(locked.value) return; if(Math.random()>=mischiefChance.value) return
  const um=unmatchedIdxs().filter(i=>!crawlAnims.value[i]); if(um.length<2) return
  const em=emptyIdxs(); const fromIdx=pick(um); let toIdx
  if(em.length>0){toIdx=pick(em)}else{const others=um.filter(i=>i!==fromIdx);if(!others.length)return;toIdx=pick(others)}
  locked.value=true; mischiefMsg.value=pick(RUN_MSGS)
  const ls=new Set(legSet.value); ls.add(fromIdx)
  const toIsCard=!matched.value.has(cards.value[toIdx].id); if(toIsCard)ls.add(toIdx); legSet.value=ls
  later(()=>{
    const anims={[fromIdx]:{...calcOffset(fromIdx,toIdx),z:12}}
    if(toIsCard)anims[toIdx]={...calcOffset(toIdx,fromIdx),z:10}
    cardAnims.value=anims
  },250)
  later(()=>{
    skipTransition.value=true
    const arr=[...cards.value];[arr[fromIdx],arr[toIdx]]=[arr[toIdx],arr[fromIdx]];cards.value=arr
    cardAnims.value={}
    const ls2=new Set(legSet.value);ls2.delete(fromIdx);ls2.delete(toIdx);legSet.value=ls2
    mischiefMsg.value='';nextTick(()=>{skipTransition.value=false;locked.value=false})
  },950)
}

// ★ crawl 用短距离偏移
function maybeCrawl() {
  if(locked.value) return
  const em=emptyIdxs(); if(em.length<6) return
  const um=unmatchedIdxs().filter(i=>!crawlAnims.value[i]); if(!um.length) return
  const count=remaining.value<=6?Math.min(2,um.length):1
  for(let n=0;n<count;n++){
    const pool=um.filter(i=>!crawlAnims.value[i]); if(!pool.length) break
    const idx=pick(pool), toIdx=pick(em)
    const off = calcPartialOffset(idx, toIdx)  // ★ 只走 35%
    const ls=new Set(legSet.value);ls.add(idx);legSet.value=ls
    crawlAnims.value={...crawlAnims.value,[idx]:off}
    later(()=>{
      const ca2={...crawlAnims.value};delete ca2[idx];crawlAnims.value=ca2
      const ls2=new Set(legSet.value);ls2.delete(idx);legSet.value=ls2
    },3000+n*500)
  }
}

function makeBurstParticles(count,small=false){
  const syms=['✦','♡','◇','★','❀','✧','◆','♪'],cols=['#FF9EB3','#7EC8E3','#8FD98F','#FFD166','#FF85A1','#C4A4FF','#9ED9A0','#FFB577']
  const ps=[],bs=small?9:15,sr=small?8:13,bd=small?55:90,dr=small?110:160
  for(let i=0;i<count;i++){const angle=(Math.PI*2*i)/count+(Math.random()-0.5)*0.5,dist=bd+Math.random()*dr
    ps.push({id:i,sym:syms[i%syms.length],color:cols[i%cols.length],dx:(Math.cos(angle)*dist).toFixed(0),dy:(Math.sin(angle)*dist).toFixed(0),size:bs+Math.floor(Math.random()*sr),delay:(Math.random()*0.1).toFixed(2)})}
  return ps
}

function triggerCelebration(rn){
  showCelebration.value=true
  burstParticles.value=makeBurstParticles(rn===2?30:24);later(()=>{burstParticles.value=[]},900)
  if(rn===2){later(()=>{burstParticles2.value=makeBurstParticles(20,true);later(()=>{burstParticles2.value=[]},900)},200)}
  later(()=>{glowActive.value=true},750)
  celebrating.value=true;later(()=>{celebrating.value=false},2000)
}



function wrapperStyle(idx){
  const anim=cardAnims.value[idx]
  if(skipTransition.value){
    if(anim)return{transform:`translate(${anim.x}, ${anim.y})`,transition:'none',zIndex:anim.z||10}
    return{transition:'none'}
  }
  if(anim)return{transform:`translate(${anim.x}, ${anim.y})`,transition:'transform 0.7s cubic-bezier(0.34,1.56,0.64,1)',zIndex:anim.z||10}
  const crawl=crawlAnims.value[idx]
  if(crawl)return{'--crawl-x':crawl.x,'--crawl-y':crawl.y,zIndex:5}
  return{}
}
</script>

<style scoped>
.game-page{display:flex;justify-content:center;min-height:100vh;padding:12px;padding-top:max(16px,5vh)}
.game-container{width:100%;max-width:380px;display:flex;flex-direction:column;align-items:center;gap:16px}
.game-header{text-align:center}
.header-text{font-size:13px;color:var(--text);font-weight:600;letter-spacing:1px}

/* ━━━ 收集栏 ━━━ */
.collection-bar{display:flex;gap:10px;justify-content:center;width:100%;max-width:340px;padding:10px 14px;border:2px solid var(--border,#C8C0B0);background:rgba(255,255,255,0.35);transition:background 0.6s,border-color 0.6s}
.collect-icon{display:flex;flex-direction:column;align-items:center;gap:2px;height:46px;filter:grayscale(1) brightness(0.15);transition:filter 0.8s ease}
.collect-icon.half{filter:grayscale(0.4) brightness(0.55)}.collect-icon.done{filter:grayscale(0) brightness(1)}
.collect-emoji{font-size:24px}
.collect-label{font-size:9px;font-weight:600;letter-spacing:0.5px;opacity:0;transition:opacity 0.4s ease}.collect-label.visible{opacity:1}
.collection-bar.celebrating .collect-icon{animation:collectPop 0.55s cubic-bezier(0.34,1.56,0.64,1)}
.collection-bar.celebrating .collect-icon:nth-child(1){animation-delay:0.1s}
.collection-bar.celebrating .collect-icon:nth-child(2){animation-delay:0.25s}
.collection-bar.celebrating .collect-icon:nth-child(3){animation-delay:0.4s}
.collection-bar.celebrating .collect-icon:nth-child(4){animation-delay:0.55s}
@keyframes collectPop{0%,100%{transform:scale(1) translateY(0)}45%{transform:scale(1.5) translateY(-5px)}75%{transform:scale(0.9) translateY(0)}}

/* ━━━ 卡牌网格 ━━━ */
.card-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;width:100%;max-width:340px}
.card-cell{aspect-ratio:1}
.card-slot-empty{width:100%;height:100%;border:2px dashed var(--border,#C8C0B0);opacity:0.25;border-radius:2px;transition:border-color 0.5s,opacity 0.5s}
.card-wrapper{width:100%;height:100%;position:relative;cursor:pointer;-webkit-tap-highlight-color:transparent}
.card-inner{width:100%;height:100%;position:relative;transform-style:preserve-3d;transition:transform 0.4s ease}
.card-inner.is-flipped{transform:rotateY(180deg)}
.card-back,.card-front{position:absolute;inset:0;backface-visibility:hidden;display:flex;flex-direction:column;align-items:center;justify-content:center;border:2px solid var(--border,#C8C0B0)}
.card-back{background:repeating-conic-gradient(var(--card-back,#FAF6F0) 0% 25%,#F0EBE4 0% 50%) 50%/12px 12px;box-shadow:0 3px 0 0 var(--shadow,rgba(0,0,0,0.06))}
.back-q{font-size:18px;color:var(--border,#C8C0B0);opacity:0.6}
.card-front{transform:rotateY(180deg);box-shadow:0 3px 0 0 var(--shadow,rgba(0,0,0,0.06));filter:saturate(0.5) brightness(1.08);transition:filter 0.5s ease}
.round-2 .card-front{filter:saturate(1) brightness(1)}
.round-2 .card-back{background:repeating-conic-gradient(#F0E8DE 0% 25%,#E5DCD0 0% 50%) 50%/12px 12px;border-color:var(--accent,#D4A574)}
.round-2 .back-q{color:var(--accent,#D4A574);opacity:0.5}
.card-icon-img{width:44px;height:44px;image-rendering:pixelated;object-fit:contain}.card-group-tag{font-size:8px;font-weight:600;letter-spacing:0.5px;margin-top:2px}

/* ★ 卡牌消失动画 */
.card-wrapper.is-vanishing{animation:cardVanish 0.42s ease forwards;pointer-events:none}
@keyframes cardVanish{0%{transform:scale(1);opacity:1}30%{transform:scale(1.08);opacity:0.9}100%{transform:scale(0);opacity:0}}

/* ━━━ 捣蛋 ━━━ */
.card-wrapper.is-shaking{animation:cardShake 0.55s ease}
@keyframes cardShake{0%,100%{transform:translateX(0) rotate(0)}15%{transform:translateX(-6px) rotate(-3deg)}35%{transform:translateX(6px) rotate(3deg)}55%{transform:translateX(-4px) rotate(-2deg)}75%{transform:translateX(4px) rotate(2deg)}}
.pixel-legs{position:absolute;bottom:-9px;left:50%;transform:translateX(-50%);display:flex;gap:8px;z-index:11}
.p-leg{display:block;width:4px;height:7px;background:var(--text-light,#8B7E6A);border-radius:0 0 1px 1px}
.has-legs .p-leg:first-child{animation:legWalk 0.28s step-end infinite}
.has-legs .p-leg:last-child{animation:legWalk 0.28s step-end 0.14s infinite}
@keyframes legWalk{0%,100%{transform:translateY(0)}50%{transform:translateY(-5px)}}
.card-wrapper.is-crawling{animation:crawlTrip 3s ease-in-out}
@keyframes crawlTrip{0%,100%{transform:translate(0,0)}35%,65%{transform:translate(var(--crawl-x,0px),var(--crawl-y,0px))}}

/* ━━━ 底部 ━━━ */
.game-footer{display:flex;justify-content:flex-end;align-items:center;width:100%;max-width:340px;margin-top:4px}
.cheat-btn{padding:4px 12px;border:2px solid var(--border,#C8C0B0);background:var(--card-back,#FAF6F0);color:var(--text-light,#8B7E6A);font-size:11px;cursor:pointer;font-family:inherit;transition:transform 0.1s}
.cheat-btn:active{transform:translateY(2px)}.done-actions{margin-top:20px}

/* ━━━ 浮层 ━━━ */
.overlay-root{position:fixed;inset:0;pointer-events:none;z-index:50}.overlay-root>*{pointer-events:auto}
.album-fixed{position:fixed;bottom:24px;right:20px;width:40px;height:40px;border:2px solid var(--border,#C8C0B0);background:var(--card-back,#FAF6F0);font-size:18px;display:flex;align-items:center;justify-content:center;cursor:default;font-family:inherit;opacity:0.35;transition:opacity 0.4s;z-index:10}
.album-fixed.unlocked{opacity:0.8;cursor:pointer}
.mischief-toast{position:fixed;bottom:16%;left:50%;transform:translateX(-50%);background:#fff;border:2px solid var(--border,#C8C0B0);padding:8px 18px;font-size:13px;font-weight:600;color:var(--text,#4A3F35);z-index:100;box-shadow:0 3px 0 0 var(--shadow,rgba(0,0,0,0.06));white-space:nowrap;letter-spacing:0.5px}
.mischief-enter-active{animation:mischiefIn 0.3s ease}.mischief-leave-active{transition:opacity 0.25s ease}.mischief-leave-to{opacity:0}
@keyframes mischiefIn{from{opacity:0;transform:translateX(-50%) translateY(6px)}to{opacity:1;transform:translateX(-50%) translateY(0)}}
.group-toast{position:fixed;top:12px;left:50%;transform:translateX(-50%);display:flex;align-items:center;gap:8px;background:#fff;border:2px solid var(--border,#C8C0B0);padding:8px 18px;z-index:80;box-shadow:0 3px 0 0 var(--shadow,rgba(0,0,0,0.06));white-space:nowrap}
.toast-icon{font-size:18px}.toast-text{font-size:13px;font-weight:600;letter-spacing:0.5px;color:var(--text,#4A3F35)}.toast-sub{font-size:11px;font-weight:600;letter-spacing:0.5px}

/* ═══ 遮罩 ═══ */
.overlay-bg{position:fixed;inset:0;background:rgba(93,78,60,0.6);z-index:200}

/* ═══ 庆祝 ═══ */
.cele-layer{position:fixed;inset:0;pointer-events:none;z-index:250;overflow:hidden}
.cele-spread{position:absolute;top:42%;left:50%;transform:translate(-50%,-50%) scale(0);width:420px;height:420px;border-radius:50%;background:radial-gradient(circle,rgba(255,220,160,0.6) 0%,rgba(255,200,130,0.35) 40%,transparent 70%);animation:spreadGlow 0.8s cubic-bezier(0.22,0.61,0.36,1) forwards}
.cele-spread.intense{width:520px;height:520px;background:radial-gradient(circle,rgba(255,220,160,0.75) 0%,rgba(255,200,130,0.45) 40%,transparent 70%)}
@keyframes spreadGlow{0%{transform:translate(-50%,-50%) scale(0);opacity:0.9}50%{transform:translate(-50%,-50%) scale(1.2);opacity:0.85}100%{transform:translate(-50%,-50%) scale(1);opacity:0}}
.cele-ambient{position:absolute;top:42%;left:50%;transform:translate(-50%,-50%);width:400px;height:400px;border-radius:50%;background:radial-gradient(circle,rgba(255,215,150,0.5) 0%,rgba(255,195,120,0.28) 40%,transparent 70%);opacity:0;transition:opacity 0.5s ease}
.cele-ambient.active{opacity:1;animation:ambientPulse 3s ease-in-out infinite}
.cele-ambient.intense{width:500px;height:500px;background:radial-gradient(circle,rgba(255,215,150,0.65) 0%,rgba(255,195,120,0.38) 40%,transparent 70%)}
@keyframes ambientPulse{0%,100%{transform:translate(-50%,-50%) scale(1);opacity:0.75}50%{transform:translate(-50%,-50%) scale(1.12);opacity:1}}
.burst-particle{position:absolute;left:50%;top:42%;pointer-events:none;animation:burstOut 0.8s cubic-bezier(0.16,1,0.3,1) both}
.burst-particle.wave2{animation:burstOut 0.7s cubic-bezier(0.16,1,0.3,1) both}
@keyframes burstOut{0%{transform:translate(-50%,-50%) scale(1.6);opacity:1}55%{opacity:0.85}100%{transform:translate(calc(-50% + var(--bp-dx,0px)),calc(-50% + var(--bp-dy,0px))) scale(0.15);opacity:0}}
.star-dust{position:absolute;top:-15px;pointer-events:none;animation:dustFall linear infinite;animation-fill-mode:both}
@keyframes dustFall{0%{transform:translateY(0) translateX(0) rotate(0deg);opacity:0}8%{opacity:var(--sd-op,0.5)}88%{opacity:var(--sd-op,0.5)}100%{transform:translateY(110vh) translateX(18px) rotate(180deg);opacity:0}}

/* ═══ 弹窗 ═══ */
.popup-layer{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;z-index:300}
.overlay-box{background:#fff;border:3px solid var(--border-dark,#5D4E3C);padding:28px 22px;text-align:center;max-width:310px;width:90%;box-shadow:0 6px 0 0 var(--shadow,rgba(0,0,0,0.08))}
.popup-emoji{font-size:36px;margin-bottom:10px}.popup-title{font-size:15px;font-weight:700;margin-bottom:4px}
.popup-sub{font-size:12px;color:var(--text-light,#8B7E6A);margin-bottom:4px}
.popup-hint{font-size:12px;color:var(--accent,#D4A574);padding:8px;background:#FFF5F0;border:1px solid #F0D8C8;margin:12px 0}
.popup-btns{display:flex;gap:12px;justify-content:center;margin-top:14px}

/* ═══ 隐藏照片揭晓 ═══ */
.reveal-overlay{position:fixed;inset:0;background:rgba(30,25,20,0.88);display:flex;flex-direction:column;align-items:center;justify-content:center;gap:24px;z-index:400;cursor:pointer;-webkit-tap-highlight-color:transparent}
.reveal-frame{background:#fff;border:4px solid var(--border,#C8C0B0);padding:10px 10px 14px;max-width:88vw;width:100%;display:flex;flex-direction:column;align-items:center;gap:10px;box-shadow:0 6px 0 0 var(--shadow,rgba(0,0,0,0.08));cursor:default}
.reveal-photo{max-width:80vw;width:100%;border-radius:1px}.reveal-text{font-size:14px;font-weight:600;color:var(--text-light,#8B7E6A);letter-spacing:1.5px}
.reveal-hint{font-size:12px;color:rgba(255,255,255,0.35);letter-spacing:1px;animation:hintBreathe 2.5s ease-in-out infinite}
@keyframes hintBreathe{0%,100%{opacity:0.35}50%{opacity:0.65}}
.reveal-fade-enter-active{transition:opacity 0.4s ease}.reveal-fade-enter-active .reveal-frame{animation:boxPop 0.4s cubic-bezier(0.34,1.56,0.64,1)}
.reveal-fade-leave-active{transition:opacity 0.3s ease}.reveal-fade-enter-from,.reveal-fade-leave-to{opacity:0}

/* ━━━ 过渡 ━━━ */
.overlay-fade-enter-active{transition:opacity 0.3s ease}.overlay-fade-leave-active{transition:opacity 0.25s ease}.overlay-fade-enter-from,.overlay-fade-leave-to{opacity:0}
.cele-fade-enter-active{transition:opacity 0.2s ease}.cele-fade-leave-active{transition:opacity 0.5s ease}.cele-fade-enter-from,.cele-fade-leave-to{opacity:0}
.popup-pop-enter-active{animation:boxPop 0.4s cubic-bezier(0.34,1.56,0.64,1)}.popup-pop-leave-active{transition:opacity 0.2s ease,transform 0.2s ease}.popup-pop-leave-to{opacity:0;transform:scale(0.95)}
@keyframes boxPop{from{opacity:0;transform:scale(0.8)}to{opacity:1;transform:scale(1)}}
.toast-enter-active{animation:toastIn 0.35s cubic-bezier(0.34,1.56,0.64,1)}.toast-leave-active{transition:opacity 0.4s ease,transform 0.4s ease}.toast-leave-to{opacity:0;transform:translateX(-50%) translateY(-20px)}
@keyframes toastIn{from{opacity:0;transform:translateX(-50%) translateY(-20px)}to{opacity:1;transform:translateX(-50%) translateY(0)}}

/* ━━━ 背景装饰 ━━━ */
.game-deco-layer{position:fixed;inset:0;pointer-events:none;z-index:0;overflow:hidden}
.deco{position:absolute;inset:0}.deco-enter-active,.deco-leave-active{transition:opacity 1.2s ease}.deco-enter-from,.deco-leave-to{opacity:0}
.g-glow{position:absolute;width:12px;height:12px;border-radius:50%;background:rgba(245,190,80,0.45);box-shadow:0 0 40px 20px rgba(245,190,80,0.15),0 0 80px 45px rgba(240,170,60,0.08)}
.g-glow-1{top:10%;left:8%;animation:candleBreathe 3s ease-in-out infinite}
.g-glow-2{top:35%;right:6%;animation:candleBreathe 3.5s ease-in-out 0.8s infinite}
.g-glow-3{top:60%;left:50%;animation:candleBreathe 4s ease-in-out 1.6s infinite}
.g-glow-4{bottom:20%;left:15%;animation:candleBreathe 3.2s ease-in-out 0.4s infinite}
.g-glow-5{bottom:35%;right:18%;animation:candleBreathe 3.8s ease-in-out 1.2s infinite}
@keyframes candleBreathe{0%,100%{opacity:0.4;transform:scale(1)}50%{opacity:0.85;transform:scale(1.2)}}
.g-wave{position:absolute;width:50%;height:2px;background:linear-gradient(90deg,transparent,rgba(100,190,170,0.22),transparent);animation:gWaveFloat linear infinite}
.g-wave-1{top:20%;animation-duration:7s}.g-wave-2{top:50%;animation-duration:9s;animation-delay:1.2s;height:2.5px}.g-wave-3{top:75%;animation-duration:8s;animation-delay:2.5s}
@keyframes gWaveFloat{0%{transform:translateX(-60%);opacity:0}15%{opacity:0.8}85%{opacity:0.8}100%{transform:translateX(160%);opacity:0}}
.g-petal{position:absolute;top:-15px;color:rgba(220,150,170,0.35);animation:gPetalFall linear infinite;animation-fill-mode:both}
@keyframes gPetalFall{0%{transform:translateY(0) translateX(0) rotate(0deg);opacity:0}8%{opacity:0.55}92%{opacity:0.55}100%{transform:translateY(105vh) translateX(20px) rotate(200deg);opacity:0}}
.g-receipt{position:absolute;display:flex;flex-direction:column;gap:14px;padding:20px 14px;opacity:0.12;border:1.5px solid var(--border,#C8C0B0);border-radius:2px}
.g-receipt-1{width:130px;height:170px;top:15%;left:8%;transform:rotate(-5deg)}.g-receipt-2{width:100px;height:130px;bottom:18%;right:6%;transform:rotate(3deg)}
.g-rline{display:block;height:2px;background:var(--border,#C8C0B0);width:100%;border-radius:1px}.g-rline.short{width:45%}.g-rline.medium{width:70%}
.g-rstain{position:absolute;width:22px;height:20px;border-radius:50%;background:rgba(200,170,90,0.25);top:22%;right:14%}
.g-rstain.stain-2{width:16px;height:16px;top:68%;left:18%;background:rgba(180,150,70,0.2)}
.g-rstain.stain-3{width:18px;height:16px;top:45%;right:20%;background:rgba(190,160,80,0.22)}

/* ★ 第二轮颜色加深 */
.round2-deco .collection-bar{background:rgba(255,248,235,0.5);border-color:var(--accent,#D4A574)}
.round2-deco .card-slot-empty{border-color:var(--accent,#D4A574);opacity:0.2}
.round2-deco .g-wave{height:3.5px;background:linear-gradient(90deg,transparent,rgba(100,200,185,0.42),transparent)}
.round2-deco .g-petal{color:rgba(225,140,170,0.65)}
.round2-deco .g-receipt{opacity:0.22}.round2-deco .g-rstain{background:rgba(210,180,100,0.4)}
</style>