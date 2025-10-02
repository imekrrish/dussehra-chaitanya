<template>
  <div
    class="head"
    :class="[head.state, { center: isCenter, disabled }]"
    @click="$emit('clickHead', head)"
    role="button"
    :aria-disabled="disabled"
  >
    <div class="face">
      <div class="eyes"></div>
      <div class="mouth"></div>
    </div>

    <div v-if="head.state === 'burning'" class="flames"></div>
    <div v-if="head.state === 'burnt'" class="ash">⚫</div>

    <div class="label">{{ head.name }}</div>
  </div>
</template>

<script setup lang="ts">
import { PropType } from 'vue'
interface HeadObj { id:number; name:string; question:string; answer:string; state:string }
const props = defineProps({
  head: { type: Object as PropType<HeadObj>, required: true },
  isCenter: { type: Boolean, default: false },
  disabled: { type: Boolean, default: false }
})
</script>

<style scoped>
.head {
  width: 120px;
  height: 120px;
  background: linear-gradient(#f8d8b8, #f2b28c);
  border-radius: 50%;
  display:flex;
  justify-content:center;
  align-items:center;
  position: relative;
  cursor: pointer;
  transition: transform .15s ease, opacity .3s ease;
  box-shadow: 0 8px 20px rgba(0,0,0,0.35);
}
.head .label {
  position: absolute;
  bottom: -1.6rem;
  font-size: .85rem;
  color: #fff;
  text-shadow: 0 1px 2px rgba(0,0,0,0.6);
}
.head .face { width:70px; height:70px; display:flex; flex-direction:column; align-items:center; justify-content:center; }
.head .eyes { width:40px; height:10px; border-radius:6px; background:#2d2d2d; margin-bottom:8px; }
.head .mouth { width:36px; height:8px; border-radius:6px; background:#7b2b2b; }

.head:hover { transform: translateY(-6px) scale(1.02); }

.head.center { border: 4px solid gold; box-shadow: 0 10px 30px rgba(255,200,0,0.12); }

.head.disabled { opacity: .45; cursor:not-allowed; transform: none; }

.head.burning {
  animation: shake 1.0s linear infinite;
  background: linear-gradient(#ff9f7a, #ff6b3c);
}
.head.burning .flames {
  position:absolute; top:-24px; left:50%; transform:translateX(-50%);
  width:60px; height:60px; background: radial-gradient(circle at 50% 30%, rgba(255,180,50,0.9), rgba(255,80,0,0.6) 40%, transparent 60%);
  filter: blur(4px);
  animation: flicker 0.9s infinite;
  pointer-events:none;
}
.head.burnt {
  background: linear-gradient(#333, #111);
  transform: rotate(-8deg) scale(.95);
  opacity: .7;
}
.head .ash {
  position:absolute; font-size:28px; color:#222; transform:translateY(6px);
}

/* tiny animations */
@keyframes flicker {
  0%{ transform: translateX(-50%) scale(1) translateY(0) rotate(0deg) }
  50%{ transform: translateX(-50%) scale(1.05) translateY(-4px) rotate(-2deg) }
  100%{ transform: translateX(-50%) scale(1) translateY(0) rotate(0deg) }
}
@keyframes shake {
  0%{ transform: translateY(-2px) rotate(-1deg) }
  50%{ transform: translateY(2px) rotate(1deg) }
  100%{ transform: translateY(-2px) rotate(-1deg) }
}
</style>
