<template>
  <div class="wrap">
    <!-- Rules panel -->
    <section class="rules" aria-labelledby="rules-title">
      <h2 id="rules-title">Rules</h2>
      <ol>
        <li>Tap a head to answer a question. Correct answers burn the head.</li>
        <li>Burn all heads <strong>except the 5th</strong> first.</li>
        <li>Only after the other 9 are gone, the <strong>5th head</strong> will unlock.</li>
      </ol>
      <p class="status" aria-live="polite">
        Remaining (excluding 5th): {{ remainingExceptSpecial }}
      </p>
    </section>

    <div class="row" :class="{ blocked: !!activeHead }" :inert="!!activeHead">
      <button
        v-for="h in heads"
        :key="h.id"
        class="head"
        :class="[
          h.state,
          { special: h.id === specialId, disabled: isDisabled(h) }
        ]"
        :aria-label="`${h.name} (No. ${h.id})`"
        :disabled="isDisabled(h)"
        @click="onClickHead(h)"
      >
        <!-- Number badge -->
        <span class="badge" :aria-hidden="true">{{ h.id }}</span>

        <span class="imgwrap" :class="{ selected: h.id===specialId && h.state==='intact' }">
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
}

const specialId = 5 // 🔒 5th Ravana unlocks only after others are gone

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
  Array.from({ length: 10 }, (_, i) => {
    const q = qa[i] ?? ['No question', []];
    return {
      id: i + 1,
      name: `Head ${i + 1}`,
      question: q[0],
      answers: q[1],
      state: 'intact' as HeadState,
    }
  })
)

const activeHead = ref<HeadObj | null>(null)
const showGreeting = ref(false)

// Are all *other* heads gone (excluding the special one)?
const othersAllGone = computed(() =>
  heads.filter(h => h.id !== specialId).every(h => h.state === 'gone')
)

// For UI: how many remaining except the special head
const remainingExceptSpecial = computed(() =>
  heads.filter(h => h.id !== specialId && h.state !== 'gone').length
)

function isDisabled(h: HeadObj) {
  // Special 5th head: only enabled when all others are gone and it's still intact
  if (h.id === specialId) return !othersAllGone.value || h.state !== 'intact'
  // All other heads: disabled if not intact
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
  const h = activeHead.value as HeadObj | null
  if (h === null) return

  const userAns = normalize(answer)
  const ok = (h.answers ?? []).some(a => normalize(a) === userAns)
  if (!ok) {
    alert('Wrong answer — try again!')
    return
  }

  activeHead.value = null
  h.state = 'burning'
  await new Promise(r => setTimeout(r, 1100))
  h.state = 'gone'

  // When all 10 are gone, show final greeting
  if (heads.every(x => x.state === 'gone')) {
    showGreeting.value = true
  }
}
</script>

<style scoped>
/* Container */
/* --- Global dark background --- */
:root { color-scheme: dark; }
html, body { background: #000; margin: 0; }

/* Container */
.wrap { 
  width: min(1100px, 96vw); 
  margin: 1rem auto 2rem; 
  position: relative;
  background: #000; /* keep wrap black */
}

/* Rules panel */
.rules {
  background: #1e1e1e;   /* dark background */
  color: #f0f0f0;        /* light text */
  border: 1px solid #333;
  border-radius: 10px;
  padding: 0.75rem 1rem;
  margin-bottom: 0.75rem;
}
.rules h2 {
  font-size: 1rem; margin: 0 0 0.25rem; font-weight: 700;
}
.rules ol {
  margin: 0 0 0.25rem 1.1rem; padding: 0; line-height: 1.4;
}
.rules .status { margin: 0; font-size: 0.9rem; opacity: 0.85; }

/* ✅ Flat, side-by-side single row */
.row { 
  display: flex; 
  align-items: center;
  gap: clamp(10px, 2vw, 18px);
  padding: 0.75rem 0.25rem;
  scroll-snap-type: x proximity;
  background: #000;
}
.row.blocked { pointer-events: none; }

/* Head button */
.head {
  position: relative;
  width: clamp(56px, 9vw, 110px);
  aspect-ratio: 1/1;
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
  scroll-snap-align: start;
}

/* Number badge */
.badge {
  position: absolute;
  top: -6px; left: -6px;
  min-width: 22px; height: 22px;
  padding: 0 6px;
  display: grid; place-items: center;
  font-size: 12px; font-weight: 700;
  color: #222;
  background: #ffd666;
  border: 1px solid #e8b339;
  border-radius: 999px;
  box-shadow: 0 2px 6px rgba(0,0,0,.12);
  z-index: 2;
}

/* Image */
.head-img { 
  width: 100%; height: 100%; object-fit: contain; display: block; 
  filter: drop-shadow(0 4px 10px rgba(0,0,0,.18));
}
.imgwrap {
  display: block; width: 100%; height: 100%;
  -webkit-clip-path: ellipse(46% 48% at 50% 47%);
          clip-path: ellipse(46% 48% at 50% 47%);
  position: relative;
}

/* Optional highlight for special (5th) */
.head.special .imgwrap.selected::after {
  content: "";
  position: absolute; inset: -6%;
  border-radius: 50%;
  border: 2px dashed rgba(255, 223, 0, .9);
  box-shadow: 0 0 12px rgba(255, 215, 0, .35);
  pointer-events: none;
  -webkit-clip-path: ellipse(52% 54% at 50% 50%);
          clip-path: ellipse(52% 54% at 50% 50%);
}

/* Hover (only desktops) */
@media (hover:hover) and (pointer:fine) {
  .head:hover:not(.disabled):not(.gone):not(.burning) .imgwrap {
    transform: translateY(-2px);
  }
}

/* Burn effect */
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
  50%  { transform: translateY(-12%) scale(1.04); opacity: .75; }
  100% { transform: translateY(-8%) scale(1);   opacity: .9; }
}

/* States */
.head.gone { visibility: hidden; }
.head.disabled { cursor: not-allowed; }
.head.special.disabled .imgwrap { filter: grayscale(0.8) opacity(0.7); }

/* Greeting overlay */
.greeting {
  position: fixed; inset: 0;
  background: rgba(0,0,0,.86);
  display: grid; place-items: center;
  z-index: 2000;
}
.greeting img {
  opacity: 0;
  transform: scale(0.92) translateY(8px);
  animation: pop-in 850ms cubic-bezier(.2,.9,.2,1) 60ms forwards,
             glow 1800ms ease-in-out 900ms infinite alternate;
  max-width: 90vw; max-height: 85vh;
  border-radius: 12px;
}
@keyframes pop-in {
  to { opacity: 1; transform: scale(1) translateY(0); }
}
@keyframes glow {
  from { box-shadow: 0 18px 50px rgba(255,215,0,0.20); }
  to   { box-shadow: 0 22px 68px rgba(255,215,0,0.35); }
}

/* 📱 Mobile: 4 heads top, 5th centered middle, 5 heads bottom */
@media (max-width: 480px) {
  .row {
    display: grid;
    grid-template-columns: repeat(5, 1fr); /* 5 equal columns */
    gap: 6px;
    padding: 8px 10px;
    justify-items: center;
    align-items: center;
    overflow: hidden;       /* no horizontal scroll */
    background: #000;       /* keep full black */
  }

  /* Each cell is a square; head fills its grid cell */
  .head {
    width: 100%;
    max-width: 72px;        /* optional safety cap */
    aspect-ratio: 1 / 1;
  }

  /* Top row: heads 1–4 (columns 1–4), leave column 5 empty */
  .head:nth-child(-n+4) { grid-row: 1; }

  /* Middle row: head 5 centered in column 3 */
  .head:nth-child(5) { grid-row: 2; grid-column: 3; }

  /* Bottom row: heads 6–10 auto-flow into row 3, columns 1–5 */
  .head:nth-child(n+6) { grid-row: 3; }

  /* Compact badge for tight grid */
  .badge {
    min-width: 20px; height: 20px; font-size: 11px;
    top: -4px; left: -4px;
  }
}


</style>
