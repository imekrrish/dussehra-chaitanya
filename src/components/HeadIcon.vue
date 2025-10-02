<template>
  <!-- Simple, stylable vector head -->
  <svg viewBox="0 0 160 160" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
    <!-- crown -->
    <g>
      <ellipse cx="80" cy="42" rx="60" ry="26" fill="#f3c34b"/>
      <circle v-for="i in 9" :key="i"
              :cx="18 + (i-1)*14" cy="42"
              r="4" fill="#d24b2a" />
      <path d="M20 40 L12 26 L28 34 Z" fill="#f3c34b"/>
      <path d="M140 40 L132 26 L148 34 Z" fill="#f3c34b"/>
      <path d="M80 40 L72 24 L88 32 Z" fill="#f3c34b"/>
    </g>
    <!-- face -->
    <circle cx="80" cy="90" r="48" fill="#f0b98f"/>
    <!-- tilak -->
    <rect x="77" y="56" width="6" height="16" rx="2" fill="#fff"/>
    <rect x="72" y="56" width="4" height="12" rx="2" fill="#d24b2a"/>
    <rect x="84" y="56" width="4" height="12" rx="2" fill="#d24b2a"/>
    <!-- eyes -->
    <rect x="55" y="82" width="22" height="9" rx="4.5" fill="#2c2c2c"/>
    <rect x="83" y="82" width="22" height="9" rx="4.5" fill="#2c2c2c"/>
    <!-- moustache -->
    <path d="M30 110 C55 92, 65 110, 80 110 C95 110, 105 92, 130 110
             C110 102, 94 120, 80 120 C66 120, 50 102, 30 110 Z"
          fill="#1f1f1f"/>
  </svg>
</template>
