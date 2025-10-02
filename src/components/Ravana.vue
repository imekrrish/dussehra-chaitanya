<template>
  <div class="wrap">
    <div class="row" :class="{ blocked: !!activeHead }" :inert="!!activeHead">
      <button
        v-for="h in heads"
        :key="h.id"
        class="head"
        :class="[
          h.state,
          { center: h.id === centerId, disabled: isDisabled(h) }
        ]"
        :style="styleFor(h)"
        :aria-label="h.name"
        :disabled="isDisabled(h)"
        @click="onClickHead(h)"
      >
        <span class="imgwrap" :class="{ selected: h.id===centerId && h.state==='intact' }">
          <img class="head-img" src="/src/assets/ravana.png" alt="" />
        </span>
        <span v-if="h.state==='burning'" class="flame"></span>
      </button>
    </div>

    <QuestionModal
      v-if="activeHead"
      :question="activeHead.question"
      :head-name="activeHead.name"
      @close="closeModal"
      @submitAnswer="checkAnswer"
    />

    <div v-if="showGreeting" class="greeting">
      <img src="/src/assets/greeting.png" alt="Happy Dussehra" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, computed } from 'vue'
import QuestionModal from './QuestionModal.vue'

type HeadState = 'intact' | 'burning' | 'gone'
interface HeadObj {
  id: number
  name: string
  question: string
  answers: string[]
  state: HeadState
  translateY: number
  scale: number
  z: number
}

const centerId = 6

// Layout for arc effect
const layout = [
  { y: 25, s: 0.68, z: 1 },
  { y: 18, s: 0.75, z: 2 },
  { y: 12, s: 0.82, z: 3 },
  { y: 6,  s: 0.90, z: 4 },
  { y: 2,  s: 0.96, z: 5 },
  { y: 0,  s: 1.15, z: 10 }, // CENTER
  { y: 2,  s: 0.96, z: 5 },
  { y: 6,  s: 0.90, z: 4 },
  { y: 12, s: 0.82, z: 3 },
  { y: 18, s: 0.75, z: 2 },
]

// Questions + multiple valid answers
const qa: Array<[string, string[]]> = [
  ['Who killed Ravana in the Ramayana?', ['lord rama', 'shri ram', 'rama', 'ram']],
  ['Which festival celebrates Lord Rama’s victory over Ravana?', ['dussehra', 'vijayadashami']],
  ['What is the name of Lord Rama’s wife?', ['sita', 'mata sita', 'goddess sita']],
  ['Which brother of Lord Rama fought bravely against Ravana?', ['lakshmana', 'laxman']],
  ['Where was Sita held captive by Ravana?', ['lanka', 'sri lanka', 'srilanka']],
  ['Which devotee of Lord Rama burned down Lanka with his tail?', ['hanuman', 'lord hanuman', 'anjaneya', 'maruti']],
  ['Which brother of Ravana supported Lord Rama in the war?', ['vibhishana', 'vibhishan']],
  ['Who among the vanaras built the bridge (Rama Setu) to Lanka?', ['nala']],
  ['Who is the sage-poet who composed the Ramayana?', ['valmiki', 'maharishi valmiki', 'sage valmiki']],
  ['What was the name of Lord Rama’s kingdom?', ['ayodhya']],
];


const heads = reactive<HeadObj[]>(
  Array.from({ length: 10 }, (_, i) => ({
    id: i + 1,
    name: `Head ${i + 1}`,
    question: qa[i][0],
    answers: qa[i][1],
    state: 'intact' as HeadState,
    translateY: layout[i].y,
    scale: layout[i].s,
    z: i === 5 ? 100 : 50 - Math.abs(5 - i),
  }))
)

const activeHead = ref<HeadObj | null>(null)
const showGreeting = ref(false)

const othersAllGone = computed(() =>
  heads.filter(h => h.id !== centerId).every(h => h.state === 'gone')
)

function isDisabled(h: HeadObj) {
  if (h.id === centerId) return !othersAllGone.value || h.state !== 'intact'
  return h.state !== 'intact'
}

function onClickHead(h: HeadObj) {
  if (isDisabled(h)) return
  activeHead.value = h
}

function closeModal() { activeHead.value = null }

// Normalize input: lowercase, trim, remove punctuation/spaces
function normalize(str: string) {
  return str.toLowerCase().trim().replace(/[^a-z0-9]/g, '')
}

async function checkAnswer(answer: string) {
  if (!activeHead.value) return

  const userAns = normalize(answer)
  const ok = activeHead.value.answers.some(a => normalize(a) === userAns)

  if (!ok) { 
    alert('Wrong answer — try again!')
    return 
  }

  // ✅ close modal immediately
  const head = activeHead.value
  activeHead.value = null

  head.state = 'burning'
  await new Promise(r => setTimeout(r, 1100))
  head.state = 'gone'

  if (heads.every(h => h.state === 'gone')) {
    showGreeting.value = true
  }
}

function styleFor(h: HeadObj) {
  const baseOverlap = -72
  const ml = `${baseOverlap * h.scale}px`
  return {
    '--y': `${h.translateY}px`,
    '--s': h.scale,
    '--z': h.z,
    marginLeft: h.id === 1 ? '0px' : ml,
  } as any
}
</script>

<style scoped>
.wrap { width: min(1100px, 96vw); margin: 1rem auto 2rem; position: relative; }
.row { display: flex; justify-content: center; align-items: flex-end; flex-wrap: nowrap; }
.row.blocked { pointer-events: none; }

.head {
  position: relative;
  width: clamp(60px, 9vw, 130px);
  aspect-ratio: 1/1;
  transform: translateY(var(--y, 0px)) scale(var(--s, 1));
  z-index: var(--z, 1);
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
}
.head-img { width: 100%; height: 100%; object-fit: contain; display: block; filter: drop-shadow(0 6px 16px rgba(0,0,0,.35)); }
.imgwrap {
  display: block; width: 100%; height: 100%;
  -webkit-clip-path: ellipse(46% 48% at 50% 47%);
          clip-path: ellipse(46% 48% at 50% 47%);
  position: relative;
}
.imgwrap.selected::after {
  content: "";
  position: absolute; inset: -6%;
  border-radius: 50%;
  border: 2px dashed rgba(255, 223, 0, .85);
  box-shadow: 0 0 20px rgba(255, 215, 0, .35);
  pointer-events: none;
  -webkit-clip-path: ellipse(52% 54% at 50% 50%);
          clip-path: ellipse(52% 54% at 50% 50%);
}
.head:hover:not(.disabled):not(.gone):not(.burning) .imgwrap {
  filter: drop-shadow(0 10px 22px rgba(255,215,0,.35));
  transform: translateY(-4px) scale(1.02);
}
.head.burning::after,
.head.burning .flame {
  content: "";
  position: absolute; inset: -8% -8% auto -8%;
  height: 140%;
  background:
    radial-gradient(circle at 50% 20%, rgba(255,210,80,.95), rgba(255,90,0,.6) 45%, transparent 60%);
  filter: blur(4px);
  animation: flame 0.9s infinite ease-in-out;
  pointer-events: none;
  border-radius: 50%;
}
@keyframes flame {
  0%   { transform: translateY(-8%) scale(1);   opacity: .9; }
  50%  { transform: translateY(-14%) scale(1.06); opacity: .7; }
  100% { transform: translateY(-8%) scale(1);   opacity: .9; }
}
.head.gone { visibility: hidden; }
.head.disabled { cursor: not-allowed; }

.greeting {
  position: fixed; inset: 0;
  background: rgba(0,0,0,.86);
  display: grid; place-items: center;
  z-index: 2000;
}
.greeting img {
  max-width: 90vw; max-height: 85vh;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.6);
}

/* ✅ Mobile responsive */
@media (max-width: 768px) {
  .wrap { width: 100%; }
  .head { width: clamp(50px, 16vw, 90px); }
}
@media (max-width: 480px) {
  .head { width: clamp(42px, 18vw, 80px); }
  .greeting img { max-width: 95vw; max-height: 80vh; }
}
</style>
