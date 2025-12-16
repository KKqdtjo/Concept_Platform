<script setup>
import { RouterView } from 'vue-router'
import { onMounted } from 'vue'

// 高性能 3D 卡片悬停效果（对所有 .panel 生效）
onMounted(() => {
  if (typeof window === 'undefined') return

  const panels = Array.from(document.querySelectorAll('.panel'))
  panels.forEach((el) => {
    let raf = null
    let bounds = null

    const handleMove = (e) => {
      if (!bounds) bounds = el.getBoundingClientRect()
      const x = e.clientX - bounds.left
      const y = e.clientY - bounds.top
      const cx = bounds.width / 2
      const cy = bounds.height / 2
      const rotateY = (x - cx) / 25
      const rotateX = (cy - y) / 25
      const lightX = (x / bounds.width) * 100
      const lightY = (y / bounds.height) * 100

      if (raf) cancelAnimationFrame(raf)
      raf = requestAnimationFrame(() => {
        el.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateZ(0)`
        el.style.backgroundImage = `radial-gradient(circle at ${lightX}% ${lightY}%, rgba(0,243,255,0.06) 0%, transparent 45%), linear-gradient(180deg,var(--dark-space), rgba(10,14,24,0.6))`
      })
    }

    const handleLeave = () => {
      if (raf) cancelAnimationFrame(raf)
      raf = requestAnimationFrame(() => {
        el.style.transform = ''
        el.style.backgroundImage = ''
      })
      bounds = null
    }

    el.addEventListener('mousemove', handleMove, { passive: true })
    el.addEventListener('mouseleave', handleLeave)
  })
})
</script>

<template>
  <div class="tech-root">
    <div class="bg-gradient" aria-hidden></div>
    <div class="bg-noise" aria-hidden></div>
    <div class="app-shell">
      <header class="topbar">
        <div class="topbar-inner">
          <div class="brand">
            <div class="brand-logo">
              <img src="/renmin-university-of-china-logo-1024px.png" alt="logo" class="brand-img" />
            </div>
            <div class="brand-text">
              <div class="site-title tech-title">概念平台</div>
              <div class="site-sub">项目申报与评审平台</div>
            </div>
          </div>
        </div>
      </header>

      <section class="hero-section">
        <div class="hero-content">
          <h1 class="hero-title">
            <span class="highlight">概念平台</span> — 项目申报与评审
          </h1>
          <p class="hero-desc">
            我们将创新想法汇聚为可落地的应用，支持项目申报、专家评审与管理。
          </p>
          <div class="feature-tags">
            <span class="tag">功能完备</span>
            <span class="tag">流程透明</span>
            <span class="tag">专家评审</span>
          </div>
        </div>
        <div class="hero-decoration">
          <div class="geo-shape shape-1"></div>
          <div class="geo-shape shape-2"></div>
          <div class="geo-shape shape-3"></div>
        </div>
      </section>

      <main class="app-container">
        <RouterView />
      </main>

      <footer class="app-footer">
        © 概念平台 — 项目申报与评审 | 持续迭代
      </footer>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Rajdhani:wght@400;700&family=Inter:wght@400;600&display=swap');
:root{
  /* 采用要求的科技色彩方案 */
  --cyber-blue: #00f3ff;      /* 霓虹蓝 - 主色调 */
  --matrix-green: #00ff41;    /* 矩阵绿 - 强调色 */
  --dark-space: #0a0a0f;      /* 深空黑 - 背景色 */
  --dark-space-2: #121420;    /* 深一点的背景过渡 */
  --neon-purple: #9d00ff;     /* 霓虹紫 - 点缀色 */
  --hologram-white: rgba(255,255,255,0.9);
  --electric-yellow: #ffea00;

  --accent: var(--cyber-blue);
  --accent-2: var(--neon-purple);
  --card-blur: 14px;
  --smooth-time: 260ms;
}

*{box-sizing:border-box}
html,body,#app{height:100%;}
body{
  margin:0;
  /* 字体栈：正文使用 Inter / Poppins，中文回落到系统中文字体，避免中文乱码 */
  font-family: 'Inter','Poppins',-apple-system,BlinkMacSystemFont,'Segoe UI','Microsoft YaHei','Noto Sans SC',sans-serif;
  -webkit-font-smoothing:antialiased;
  -moz-osx-font-smoothing:grayscale;
  text-rendering:optimizeLegibility;
  background: linear-gradient(180deg,var(--dark-space),var(--dark-space-2));
  color: var(--hologram-white);
}

.tech-root{position:relative;min-height:100vh;overflow:hidden}
.bg-gradient{position:fixed;inset:0;z-index:0;background:radial-gradient(600px 300px at 10% 20%, rgba(157,0,255,0.08), transparent 10%), radial-gradient(500px 250px at 90% 80%, rgba(0,243,255,0.06), transparent 12%);pointer-events:none;mix-blend-mode:screen;}
.bg-noise{position:fixed;inset:0;z-index:1;background-image:linear-gradient(transparent 0 50%, rgba(255,255,255,0.012) 100%);opacity:0.18;pointer-events:none}
.app-shell{position:relative;z-index:2;min-height:100vh;display:flex;flex-direction:column}

/* 卡片 / 面板 统一样式 */
.panel{
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border:1px solid rgba(255,255,255,0.04);
  backdrop-filter: blur(var(--card-blur));
  border-radius:12px;padding:16px;transition:transform var(--smooth-time) ease,box-shadow var(--smooth-time) ease;will-change:transform,background-image
}
.panel:where(:hover){transform:translateY(-6px) scale(1.005);box-shadow:0 18px 40px rgba(0,0,0,0.6),0 0 40px rgba(0,243,255,0.04)}

/* 按钮风格 */
.btn-tech{background:linear-gradient(90deg,var(--accent),var(--accent-2));color:#041226;border:none;padding:8px 14px;border-radius:10px;cursor:pointer;transition:transform var(--smooth-time) ease,filter var(--smooth-time);box-shadow:0 8px 30px rgba(0,243,255,0.06),0 4px 12px rgba(157,0,255,0.03)}
.btn-tech:active{transform:scale(0.985)}

/* 链接与高亮 */
a{color:var(--accent);text-decoration:none}

/* 简洁的出现动画，避免阻塞主线程 */
.key-fadeInUp{animation:fadeInUp 420ms cubic-bezier(.2,.9,.25,1) both}
/* 兼容老的类名，视图中使用了 .fade-in-up */
.fade-in-up{animation:fadeInUp 420ms cubic-bezier(.2,.9,.25,1) both}
@keyframes fadeInUp{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}

/* 减少动画给低配设备的压力 */
@media (prefers-reduced-motion: reduce){
  *{animation:none!important;transition:none!important}
}

/* 调整 Element Plus 组件的默认背景，让组件继承科技感 */
.el-container, .el-main, .el-aside, .el-header{background:transparent}

/* 标题等使用科技感无衬线字体，若包含中文则回落到系统中文字体，避免乱码 */
.tech-title{font-family:'Orbitron','Rajdhani','Exo 2','Poppins','Microsoft YaHei',sans-serif;font-weight:700;color:var(--hologram-white)}

/* 优化文本抗锯齿，防止中文显示异常 */
body,button,input,textarea{ -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale; }

/* 全局滚动平滑 */
html{scroll-behavior:smooth}

/* 性能提示：只对转换和不触发布局的属性做动画 */
.fast-transform{will-change:transform,opacity}

/* 强制 Element Plus 卡片透明化，避免白色背景穿插 */
::v-deep .el-card{
  background: transparent !important;
  border-radius: 12px !important;
  box-shadow: none !important;
}
::v-deep .el-card__body{
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)) !important;
  border: 1px solid rgba(255,255,255,0.04) !important;
  backdrop-filter: blur(12px);
  padding: 16px !important;
}

/* 未来主义字体与标题样式 */
.tech-title{font-family:'Orbitron','Rajdhani','Exo 2','Poppins','Microsoft YaHei',sans-serif;font-weight:700;color:var(--hologram-white);letter-spacing:0.6px}
h1,h2,h3{font-family:'Orbitron','Rajdhani','Exo 2','Poppins','Microsoft YaHei',sans-serif}

/* 面板几何伪元素（六边形 / 三角）用于产生科技几何感 */
.panel{position:relative;overflow:visible}
.panel::before,.panel::after{
  content:'';position:absolute;pointer-events:none;opacity:0.06;filter:blur(6px);transition:opacity 400ms ease,transform 400ms ease
}
.panel::before{
  width:180px;height:180px;right:-40px;top:-40px;background:conic-gradient(from 180deg at 50% 50%, rgba(157,0,255,0.18), rgba(0,243,255,0.12));transform:rotate(18deg);clip-path:polygon(50% 0%,100% 25%,100% 75%,50% 100%,0% 75%,0% 25%);
}
.panel::after{
  width:120px;height:120px;left:-30px;bottom:-30px;background:linear-gradient(135deg, rgba(0,243,255,0.12), rgba(157,0,255,0.08));transform:rotate(-14deg);clip-path:polygon(50% 0%,100% 38%,82% 100%,18% 100%,0% 38%);
}
.panel:hover::before,.panel:hover::after{opacity:0.12;transform:scale(1.03)}

/* 金属质感边框和微光 */
.panel{border-image: linear-gradient(90deg, rgba(255,255,255,0.02), rgba(0,243,255,0.06), rgba(157,0,255,0.04)) 1;border-radius:12px}

/* 小型 3D 悬浮微交互（仅在支持的设备上启用） */
@media (hover: hover) and (pointer: fine){
  .panel{transition:transform 360ms cubic-bezier(.2,.9,.25,1),box-shadow 360ms}
  .panel:active{transform:translateY(2px)}
}

/* 图标和按钮的科技装饰 */
.icon-tech{display:inline-flex;align-items:center;justify-content:center;width:36px;height:36px;border-radius:8px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));box-shadow:0 6px 18px rgba(0,0,0,0.6);color:var(--accent);}

/* 顶部 / hero / 布局样式（参考附件） */
.topbar{background: linear-gradient(90deg, rgba(10,12,20,0.6) 0%, rgba(18,24,40,0.6) 100%);color:var(--hologram-white);border-bottom:1px solid rgba(0,243,255,0.04);backdrop-filter:blur(8px);position:sticky;top:0;z-index:30}
.topbar-inner{max-width:1200px;margin:0 auto;padding:12px 20px;display:flex;align-items:center}
.brand{display:flex;align-items:center}
.brand-logo{width:44px;height:44px;border-radius:8px;background:linear-gradient(135deg, rgba(0,243,255,0.06), rgba(157,0,255,0.04));display:flex;align-items:center;justify-content:center;margin-right:12px;border:1px solid rgba(0,243,255,0.06)}
.brand-img{width:36px;height:36px;object-fit:contain}
.site-title{font-weight:700;font-size:18px}
.site-sub{font-size:12px;opacity:0.85}

.hero-section{max-width:1200px;margin:28px auto 36px;padding:28px 20px;position:relative;overflow:hidden}
.hero-content{position:relative;z-index:2;text-align:center}
.hero-title{font-size:40px;font-weight:800;margin:0 0 14px;background:linear-gradient(90deg,var(--accent),var(--accent-2));-webkit-background-clip:text;-webkit-text-fill-color:transparent}
.hero-title .highlight{color:var(--accent);-webkit-text-fill-color:var(--accent)}
.hero-desc{font-size:15px;color:rgba(235,245,255,0.9);max-width:720px;margin:0 auto 18px;line-height:1.6}
.feature-tags{display:flex;gap:10px;justify-content:center;flex-wrap:wrap}
.tag{padding:8px 14px;border-radius:20px;background:linear-gradient(90deg, rgba(0,243,255,0.04), rgba(157,0,255,0.03));border:1px solid rgba(0,243,255,0.04);color:var(--hologram-white);font-size:13px;font-weight:600}

.hero-decoration{position:absolute;width:100%;height:100%;top:0;left:0;z-index:1;opacity:0.28}
.geo-shape{position:absolute;animation:float 6s ease-in-out infinite}
.shape-1{width:150px;height:150px;background:linear-gradient(135deg, var(--accent), transparent);clip-path:polygon(50% 0%,100% 50%,50% 100%,0% 50%);top:-40px;right:10%}
.shape-2{width:120px;height:120px;background:linear-gradient(45deg,var(--accent-2),transparent);clip-path:polygon(25% 0%,100% 0%,75% 100%,0% 100%);bottom:-30px;left:12%}
.shape-3{width:100px;height:100px;background:linear-gradient(225deg,var(--accent),transparent);border-radius:50%;bottom:20px;right:20%}

.app-container{max-width:1200px;margin:0 auto 60px;padding:0 20px;min-height:400px}
.app-footer{text-align:center;padding:18px;color:rgba(220,230,255,0.8);font-size:13px;border-top:1px solid rgba(0,243,255,0.02);margin-top:40px}

@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(18px)}}

</style>
