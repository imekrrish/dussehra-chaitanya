<template>
  <div
    class="modal-backdrop"
    @click.self="close"
    role="dialog"
    aria-modal="true"
    :aria-labelledby="headingId"
    :aria-describedby="descId"
  >
    <div class="modal" ref="modalEl">
      <div class="modal-top-ornament" aria-hidden="true"></div>

      <h3 :id="headingId" class="title">
        Question for {{ headName }}
      </h3>

      <p :id="descId" class="question">
        {{ question }}
      </p>

      <label class="input-wrap">
        <span class="visually-hidden">Your answer</span>
        <input
          ref="answerEl"
          v-model="value"
          @keyup.enter="submit"
          placeholder="Type your answer"
          autocomplete="off"
          spellcheck="false"
          inputmode="text"
        />
      </label>

      <div class="actions">
        <button class="btn primary" @click="submit">Submit</button>
        <button class="btn ghost" @click="close">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps<{ question: string; headName: string }>()
const emit = defineEmits(['close', 'submitAnswer'])

const value = ref('')
const answerEl = ref<HTMLInputElement | null>(null)
const modalEl  = ref<HTMLDivElement | null>(null)

// a11y ids
const headingId = `modal-title-${Math.random().toString(36).slice(2)}`
const descId    = `modal-desc-${Math.random().toString(36).slice(2)}`

watch(() => props.question, () => (value.value = ''))

function close() {
  emit('close')
}

function submit() {
  emit('submitAnswer', value.value || '')
}

// Focus input on mount; close on ESC
function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape') {
    e.preventDefault()
    close()
  }
}

onMounted(() => {
  answerEl.value?.focus()
  window.addEventListener('keydown', onKey)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKey)
})
</script>

<style scoped>
/* Theme tokens (tweak to taste) */
:root {
  --clr-bg: rgba(0, 0, 0, 0.35);       /* lighter black, works with blur */
  --panel: #1a1a1a;
  --panel-grad: linear-gradient(180deg, #21130d 0%, #1b0f0a 50%, #1a1a1a 100%);
  --saffron: #ffb000;
  --maroon: #6e0d0d;
  --gold: #ffd24a;
  --ink: #111;
  --text: #f7f3e8;
  --muted: #c8bdb3;
  --ring: rgba(255, 210, 74, 0.9);
  --shadow: 0 12px 40px rgba(0, 0, 0, 0.6);
}

/* Backdrop with heavy blur for frosted glass effect */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: var(--clr-bg);
  backdrop-filter: blur(14px);      /* ⬅️ strong blur */
  -webkit-backdrop-filter: blur(14px);
  display: grid;
  place-items: center;
  padding: 1rem;
  z-index: 2000;
}

/* Panel with saffron-gold trim and subtle texture feel */
.modal {
  width: min(92vw, 420px);
  background: var(--panel-grad);
  color: var(--text);
  border-radius: 14px;
  border: 1px solid rgba(255, 210, 74, 0.35);
  box-shadow: var(--shadow);
  position: relative;
  padding: 1.25rem 1.25rem 1rem;
  overflow: hidden;
}

/* Ornamental top strip */
.modal-top-ornament {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 10px;
  background:
    repeating-linear-gradient(
      90deg,
      transparent 0 14px,
      rgba(255, 210, 74, 0.35) 14px 16px
    ),
    linear-gradient(90deg, #b86b00, #ffd24a, #b86b00);
  box-shadow: 0 1px 0 rgba(0,0,0,0.35) inset;
}

.title {
  margin: 0 0 .5rem 0;
  text-align: center;
  font-size: 1.25rem;
  font-weight: 800;
  letter-spacing: 0.3px;
  color: var(--gold);
  text-shadow: 0 1px 0 #3a2412;
}

.question {
  margin: 0 0 .9rem 0;
  text-align: center;
  color: var(--text);
  line-height: 1.35;
}

/* Input */
.input-wrap {
  display: block;
  margin-bottom: .9rem;
}
input {
  width: 100%;
  padding: .7rem .9rem;
  border-radius: 10px;
  border: 1px solid rgba(255, 210, 74, 0.35);
  background: #ffffff;            /* pure white for contrast */
  color: #111;                    /* ⬅️ dark text */
  font-size: 1rem;
  outline: none;
  transition: box-shadow .15s ease, border-color .15s ease, transform .05s ease;
}
input::placeholder { color: #666; }
input:focus {
  border-color: var(--gold);
  box-shadow: 0 0 0 3px var(--ring);
  transform: translateY(-1px);
}

/* Buttons */
.actions {
  display: flex;
  gap: .6rem;
  justify-content: flex-end;
}
.btn {
  appearance: none;
  border: 1px solid rgba(255, 210, 74, 0.45);
  border-radius: 10px;
  padding: .6rem 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: transform .05s ease, box-shadow .15s ease, background .15s ease, color .15s ease;
}
.btn:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--ring);
}
.btn.primary {
  background: linear-gradient(180deg, #ffb000, #e89200);
  color: #2b1708;
  text-shadow: 0 1px 0 rgba(255,255,255,0.45);
}
.btn.primary:hover { filter: brightness(1.05); }
.btn.primary:active { transform: translateY(1px); }

.btn.ghost {
  background: transparent;
  color: var(--muted);
  border-color: rgba(255, 210, 74, 0.3);
}
.btn.ghost:hover { color: var(--text); border-color: rgba(255, 210, 74, 0.55); }
.btn.ghost:active { transform: translateY(1px); }

/* Accessibility helpers */
.visually-hidden {
  position: absolute !important;
  height: 1px; width: 1px;
  overflow: hidden; clip: rect(1px, 1px, 1px, 1px);
  white-space: nowrap;
}
</style>

