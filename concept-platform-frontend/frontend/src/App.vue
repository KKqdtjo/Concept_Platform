<script setup>
import { RouterView } from 'vue-router'
</script>

<template>
  <div class="app-shell">
    <div class="app-bg">
      <div class="bg-orbit bg-orbit-1"></div>
      <div class="bg-orbit bg-orbit-2"></div>
      <div class="bg-orbit bg-orbit-3"></div>

      <div class="bg-geo geo-circle"></div>
      <div class="bg-geo geo-pentagon"></div>
      <div class="bg-geo geo-hexagon"></div>

      <div class="bg-lines lines-horizontal"></div>
      <div class="bg-lines lines-diagonal"></div>
    </div>
    <RouterView />
  </div>
</template>

<style>
.app-shell {
  min-height: 100vh;
  position: relative;
  overflow: hidden;
}

.app-bg {
  position: fixed;
  inset: 0;
  background:
    radial-gradient(1200px circle at 10% 0%, rgba(0, 191, 255, 0.22), transparent 40%),
    radial-gradient(900px circle at 90% 100%, rgba(147, 112, 219, 0.24), transparent 45%),
    radial-gradient(1100px circle at 50% 80%, rgba(57, 255, 20, 0.12), transparent 48%),
    linear-gradient(180deg, #0a1929 0%, #020617 55%, #050816 100%);
  pointer-events: none;
  z-index: 0;
}

.bg-orbit {
  position: absolute;
  border-radius: 999px;
  border: 1px solid rgba(148, 163, 184, 0.45);
  box-shadow: 0 0 40px rgba(15, 23, 42, 0.2);
  backdrop-filter: blur(20px);
  opacity: 0.7;
}

.bg-orbit-1 {
  width: 520px;
  height: 520px;
  top: -160px;
  left: -80px;
  background: radial-gradient(circle at 0 0, rgba(129, 140, 248, 0.35), transparent 55%);
  animation: float-orbit-1 28s linear infinite;
}

.bg-orbit-2 {
  width: 420px;
  height: 420px;
  top: 40%;
  right: -140px;
  background: radial-gradient(circle at 100% 0, rgba(56, 189, 248, 0.26), transparent 55%);
  animation: float-orbit-2 32s linear infinite;
}

.bg-orbit-3 {
  width: 560px;
  height: 560px;
  bottom: -260px;
  left: 30%;
  background: radial-gradient(circle at 50% 100%, rgba(59, 130, 246, 0.28), transparent 55%);
  animation: float-orbit-3 36s linear infinite;
}

.bg-geo {
  position: absolute;
  width: 160px;
  height: 160px;
  background: conic-gradient(from 120deg, rgba(59, 130, 246, 0.7), rgba(56, 189, 248, 0.4), rgba(129, 140, 248, 0.7), rgba(59, 130, 246, 0.7));
  mix-blend-mode: screen;
  opacity: 0.9;
  filter: blur(0.5px);
}

.geo-circle {
  top: 14%;
  right: 16%;
  border-radius: 50%;
  animation: geo-drift-1 26s ease-in-out infinite;
}

.geo-pentagon {
  bottom: 18%;
  left: 10%;
  clip-path: polygon(50% 0%, 100% 38%, 82% 100%, 18% 100%, 0% 38%);
  animation: geo-drift-2 30s ease-in-out infinite;
}

.geo-hexagon {
  top: 55%;
  left: 55%;
  clip-path: polygon(25% 3%, 75% 3%, 100% 50%, 75% 97%, 25% 97%, 0% 50%);
  animation: geo-drift-3 34s ease-in-out infinite;
}

.bg-lines {
  position: absolute;
  inset: 0;
  opacity: 0.42;
  background-size: 50px 50px;
  mask-image: radial-gradient(circle at 50% 50%, black 0%, transparent 70%);
}

.lines-horizontal {
  background-image:
    linear-gradient(
      to right,
      rgba(72, 209, 204, 0.4) 1px,
      transparent 1px
    ),
    linear-gradient(
      to bottom,
      rgba(72, 209, 204, 0.4) 1px,
      transparent 1px
    );
  animation: lines-pan-x 40s linear infinite;
}

.lines-diagonal {
  background-image:
    repeating-linear-gradient(
      135deg,
      rgba(0, 255, 255, 0.22) 0,
      rgba(0, 255, 255, 0.22) 1px,
      transparent 1px,
      transparent 18px
    );
  mix-blend-mode: soft-light;
  animation: lines-pan-y 46s linear infinite;
}

.app-shell > :not(.app-bg) {
  position: relative;
  z-index: 1;
}

@keyframes float-orbit-1 {
  0% {
    transform: translate3d(0, 0, 0) rotate(0deg);
  }
  50% {
    transform: translate3d(20px, 40px, 0) rotate(5deg);
  }
  100% {
    transform: translate3d(0, 0, 0) rotate(0deg);
  }
}

@keyframes float-orbit-2 {
  0% {
    transform: translate3d(0, 0, 0) rotate(0deg);
  }
  50% {
    transform: translate3d(-30px, -30px, 0) rotate(-6deg);
  }
  100% {
    transform: translate3d(0, 0, 0) rotate(0deg);
  }
}

@keyframes float-orbit-3 {
  0% {
    transform: translate3d(0, 0, 0) rotate(0deg);
  }
  50% {
    transform: translate3d(10px, -40px, 0) rotate(8deg);
  }
  100% {
    transform: translate3d(0, 0, 0) rotate(0deg);
  }
}

@keyframes geo-drift-1 {
  0% {
    transform: translate3d(0, 0, 0) rotate(0deg) scale(1);
  }
  50% {
    transform: translate3d(-20px, 18px, 0) rotate(35deg) scale(1.05);
  }
  100% {
    transform: translate3d(0, 0, 0) rotate(0deg) scale(1);
  }
}

@keyframes geo-drift-2 {
  0% {
    transform: translate3d(0, 0, 0) rotate(-8deg) scale(0.95);
  }
  50% {
    transform: translate3d(18px, -16px, 0) rotate(10deg) scale(1.06);
  }
  100% {
    transform: translate3d(0, 0, 0) rotate(-8deg) scale(0.95);
  }
}

@keyframes geo-drift-3 {
  0% {
    transform: translate3d(0, 0, 0) rotate(4deg) scale(0.96);
  }
  50% {
    transform: translate3d(-24px, 22px, 0) rotate(-6deg) scale(1.04);
  }
  100% {
    transform: translate3d(0, 0, 0) rotate(4deg) scale(0.96);
  }
}

@keyframes lines-pan-x {
  0% {
    transform: translate3d(0, 0, 0);
  }
  100% {
    transform: translate3d(-120px, 0, 0);
  }
}

@keyframes lines-pan-y {
  0% {
    transform: translate3d(0, 0, 0);
  }
  100% {
    transform: translate3d(0, -180px, 0);
  }
}
</style>
