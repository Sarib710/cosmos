<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>COSMOS — The Universe Encyclopedia</title>

  <!-- Google Fonts: Display + Body -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;900&family=Space+Grotesk:wght@300;400;500;600;700&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet" />

  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- GSAP + ScrollTrigger -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>

  <!-- Three.js -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

  <!-- ColorThief -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/color-thief/2.3.0/color-thief.umd.js"></script>

  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            display: ['Space Grotesk', 'sans-serif'],
            body: ['Inter', 'sans-serif'],
            mono: ['Space Mono', 'monospace'],
          },
        }
      }
    }
  </script>

  <style>
    /* ─── Reset & Base ─────────────────────────────────── */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --c-void:    #020408;
      --c-deep:    #060d1a;
      --c-nebula:  #0a1628;
      --c-dust:    #0d1f3c;
      --c-glow:    #1a3a6e;
      --c-star:    #e8f4ff;
      --c-pale:    #a8c8f0;
      --c-accent:  #4d9fff;
      --c-gold:    #f0b429;
      --c-violet:  #7c3aed;
      --c-plasma:  #06b6d4;
      --c-solar:   #ff6b2b;
      --c-corona:  #ffb347;
      --scroll-ease: cubic-bezier(0.25, 1, 0.5, 1);
    }

    html { scroll-behavior: smooth; overflow-x: hidden; }

    body {
      background: var(--c-void);
      color: var(--c-star);
      font-family: 'Inter', sans-serif;
      overflow-x: hidden;
      cursor: none;
    }

    /* ─── Custom Cursor ───────────────────────────────── */
    #cursor-dot {
      position: fixed; top: 0; left: 0; width: 8px; height: 8px;
      background: var(--c-accent); border-radius: 50%;
      pointer-events: none; z-index: 9999;
      transform: translate(-50%, -50%);
      transition: transform 0.05s ease, background 0.2s ease;
      will-change: transform;
    }
    #cursor-ring {
      position: fixed; top: 0; left: 0; width: 36px; height: 36px;
      border: 1px solid rgba(77, 159, 255, 0.5);
      border-radius: 50%; pointer-events: none; z-index: 9998;
      transform: translate(-50%, -50%);
      transition: width 0.3s var(--scroll-ease), height 0.3s var(--scroll-ease),
                  opacity 0.3s ease;
      will-change: transform;
    }
    body.hovering #cursor-dot  { transform: translate(-50%, -50%) scale(2); background: var(--c-gold); }
    body.hovering #cursor-ring { width: 56px; height: 56px; opacity: 0.3; border-color: var(--c-gold); }

    /* ─── Canvas Starfield ────────────────────────────── */
    #starfield-canvas {
      position: fixed; top: 0; left: 0; width: 100%; height: 100%;
      z-index: 0; pointer-events: none;
      will-change: transform;
    }

    /* ─── Hero Section ────────────────────────────────── */
    #hero {
      position: relative; z-index: 10;
      min-height: 100vh;
      display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      overflow: hidden;
    }

    .hero-eyebrow {
      font-family: 'Space Mono', monospace;
      font-size: 11px; letter-spacing: 0.35em;
      color: var(--c-accent); text-transform: uppercase;
      opacity: 0; transform: translateY(20px);
    }

    .hero-title {
      font-family: 'Space Grotesk', sans-serif;
      font-size: clamp(72px, 14vw, 180px);
      font-weight: 700; line-height: 0.88;
      letter-spacing: -0.04em;
      text-align: center;
      background: linear-gradient(
        160deg,
        #ffffff 0%,
        #c8e0ff 30%,
        #8ab4e0 60%,
        #4d9fff 100%
      );
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
      opacity: 0; transform: translateY(60px) scale(0.95);
      will-change: transform, opacity;
    }

    .hero-sub {
      font-family: 'Inter', sans-serif;
      font-size: clamp(14px, 2vw, 18px);
      font-weight: 300; color: var(--c-pale);
      letter-spacing: 0.02em; text-align: center;
      max-width: 480px; line-height: 1.7;
      opacity: 0; transform: translateY(30px);
    }

    .hero-scroll-hint {
      position: absolute; bottom: 40px;
      display: flex; flex-direction: column;
      align-items: center; gap: 8px;
      opacity: 0;
    }
    .scroll-line {
      width: 1px; height: 60px;
      background: linear-gradient(to bottom, var(--c-accent), transparent);
      animation: scrollPulse 2s ease-in-out infinite;
    }
    .scroll-label {
      font-family: 'Space Mono', monospace;
      font-size: 10px; letter-spacing: 0.2em;
      color: var(--c-pale); opacity: 0.6;
      text-transform: uppercase;
    }

    @keyframes scrollPulse {
      0%, 100% { opacity: 0.4; transform: scaleY(1); }
      50% { opacity: 1; transform: scaleY(1.1); }
    }

    /* ─── Nebula aurora behind hero ─────────────────── */
    .nebula-aurora {
      position: absolute; width: 900px; height: 900px;
      border-radius: 50%; pointer-events: none;
      filter: blur(120px); opacity: 0;
    }
    .aurora-1 {
      background: radial-gradient(circle, rgba(77, 159, 255, 0.15) 0%, transparent 70%);
      top: 50%; left: 50%; transform: translate(-50%, -55%);
    }
    .aurora-2 {
      background: radial-gradient(circle, rgba(124, 58, 237, 0.12) 0%, transparent 65%);
      top: 50%; left: 30%; transform: translate(-50%, -40%);
      width: 600px; height: 600px;
    }
    .aurora-3 {
      background: radial-gradient(circle, rgba(255, 107, 43, 0.1) 0%, transparent 65%);
      top: 50%; left: 70%; transform: translate(-50%, -40%);
      width: 500px; height: 500px;
    }

    /* ─── Transition Spacer ───────────────────────────── */
    #scroll-spacer { height: 120px; }

    /* ─── Section Reveal Container ───────────────────── */
    #explore-reveal {
      position: relative; z-index: 10;
      min-height: 100vh;
      display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      padding: 80px 24px;
      gap: 80px;
    }

    .section-label {
      font-family: 'Space Mono', monospace;
      font-size: 11px; letter-spacing: 0.4em;
      color: var(--c-accent); text-transform: uppercase;
      opacity: 0; transform: translateY(16px);
    }

    /* ─── Bento Cards ─────────────────────────────────── */
    .bento-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 20px;
      width: 100%; max-width: 1100px;
    }

    .bento-card {
      position: relative; overflow: hidden;
      border-radius: 24px;
      border: 1px solid rgba(255, 255, 255, 0.06);
      background: rgba(10, 22, 40, 0.6);
      backdrop-filter: blur(24px);
      -webkit-backdrop-filter: blur(24px);
      cursor: pointer;
      aspect-ratio: 0.72;
      will-change: transform;
      transform: translateY(80px); opacity: 0;
      transition: border-color 0.4s ease;
    }

    .bento-card:hover {
      border-color: rgba(255, 255, 255, 0.15);
    }

    /* Card-specific accent glows */
    .card-solar .card-glow { background: radial-gradient(circle at 50% 100%, rgba(255, 180, 60, 0.35) 0%, transparent 65%); }
    .card-galaxy .card-glow { background: radial-gradient(circle at 50% 100%, rgba(124, 58, 237, 0.4) 0%, transparent 65%); }
    .card-black-hole .card-glow { background: radial-gradient(circle at 50% 60%, rgba(6, 182, 212, 0.3) 0%, transparent 60%); }

    .card-glow {
      position: absolute; inset: 0;
      pointer-events: none; opacity: 0;
      transition: opacity 0.5s ease;
    }
    .bento-card:hover .card-glow { opacity: 1; }

    /* Card visual area (top 65%) */
    .card-visual {
      position: absolute; top: 0; left: 0;
      width: 100%; height: 65%;
      display: flex; align-items: center;
      justify-content: center;
      overflow: hidden;
    }

    /* Card canvas for each animated object */
    .card-canvas {
      position: absolute; inset: 0;
      width: 100%; height: 100%;
    }

    /* Card content (bottom 35%) */
    .card-content {
      position: absolute; bottom: 0; left: 0;
      width: 100%; padding: 28px 28px 32px;
    }

    .card-tag {
      display: inline-block;
      font-family: 'Space Mono', monospace;
      font-size: 10px; letter-spacing: 0.25em;
      text-transform: uppercase; padding: 4px 10px;
      border-radius: 100px;
      border: 1px solid rgba(255,255,255,0.12);
      color: rgba(255,255,255,0.5);
      margin-bottom: 12px;
    }

    .card-title {
      font-family: 'Space Grotesk', sans-serif;
      font-size: clamp(26px, 3vw, 34px);
      font-weight: 700; line-height: 1.0;
      letter-spacing: -0.02em; color: #ffffff;
      margin-bottom: 10px;
    }

    .card-desc {
      font-size: 13px; line-height: 1.65;
      color: rgba(168, 200, 240, 0.7);
      font-weight: 300;
    }

    .card-arrow {
      position: absolute; bottom: 28px; right: 28px;
      width: 42px; height: 42px;
      border: 1px solid rgba(255,255,255,0.1);
      border-radius: 50%;
      display: flex; align-items: center; justify-content: center;
      opacity: 0;
      transition: opacity 0.3s ease, transform 0.3s var(--scroll-ease),
                  background 0.3s ease;
    }
    .bento-card:hover .card-arrow { opacity: 1; transform: scale(1.05); }
    .card-arrow svg { width: 16px; height: 16px; color: white; }

    /* Card Solar accent */
    .card-solar .card-title { background: linear-gradient(135deg, #fff 0%, #ffb347 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
    .card-solar .card-tag { border-color: rgba(255, 179, 71, 0.3); color: rgba(255, 179, 71, 0.8); }
    .card-solar .card-arrow { border-color: rgba(255, 179, 71, 0.3); background: rgba(255, 179, 71, 0.08); }

    .card-galaxy .card-title { background: linear-gradient(135deg, #fff 0%, #c084fc 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
    .card-galaxy .card-tag { border-color: rgba(192, 132, 252, 0.3); color: rgba(192, 132, 252, 0.8); }
    .card-galaxy .card-arrow { border-color: rgba(192, 132, 252, 0.3); background: rgba(192, 132, 252, 0.08); }

    .card-black-hole .card-title { background: linear-gradient(135deg, #fff 0%, #06b6d4 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
    .card-black-hole .card-tag { border-color: rgba(6, 182, 212, 0.3); color: rgba(6, 182, 212, 0.8); }
    .card-black-hole .card-arrow { border-color: rgba(6, 182, 212, 0.3); background: rgba(6, 182, 212, 0.08); }

    /* ─── Stats Bar ───────────────────────────────────── */
    .stats-bar {
      width: 100%; max-width: 1100px;
      display: grid; grid-template-columns: repeat(4, 1fr);
      gap: 1px;
      background: rgba(255,255,255,0.04);
      border-radius: 16px; overflow: hidden;
      border: 1px solid rgba(255,255,255,0.05);
      opacity: 0; transform: translateY(40px);
    }

    .stat-item {
      padding: 28px 32px;
      background: rgba(6, 13, 26, 0.8);
      backdrop-filter: blur(20px);
    }

    .stat-value {
      font-family: 'Space Grotesk', sans-serif;
      font-size: 32px; font-weight: 700;
      background: linear-gradient(135deg, #fff 0%, var(--c-accent) 100%);
      -webkit-background-clip: text; -webkit-text-fill-color: transparent;
      background-clip: text; letter-spacing: -0.03em;
    }

    .stat-label {
      font-size: 12px; color: rgba(168, 200, 240, 0.5);
      letter-spacing: 0.05em; margin-top: 4px;
      font-family: 'Space Mono', monospace;
      text-transform: uppercase;
    }

    /* ─── Detail Page (Part 2 target) ────────────────── */
    #detail-page {
      display: none;
      position: fixed; inset: 0;
      z-index: 100; overflow-y: auto;
      background: var(--c-void);
    }

    /* ─── Scroll Progress Bar ─────────────────────────── */
    #progress-bar {
      position: fixed; top: 0; left: 0;
      height: 2px; width: 0%;
      background: linear-gradient(to right, var(--c-accent), var(--c-violet));
      z-index: 200; transition: width 0.1s linear;
      will-change: width;
    }

    /* ─── Nav ─────────────────────────────────────────── */
    #nav {
      position: fixed; top: 0; left: 0; right: 0;
      z-index: 150; padding: 24px 48px;
      display: flex; align-items: center;
      justify-content: space-between;
      opacity: 0;
      transition: background 0.3s ease, backdrop-filter 0.3s ease;
    }
    #nav.scrolled {
      background: rgba(2, 4, 8, 0.7);
      backdrop-filter: blur(24px);
      -webkit-backdrop-filter: blur(24px);
    }

    .nav-logo {
      font-family: 'Space Grotesk', sans-serif;
      font-size: 18px; font-weight: 700;
      letter-spacing: -0.02em; color: white;
      display: flex; align-items: center; gap: 10px;
    }
    .nav-logo-dot {
      width: 8px; height: 8px; border-radius: 50%;
      background: var(--c-accent);
      box-shadow: 0 0 12px var(--c-accent);
      animation: dotPulse 2.5s ease-in-out infinite;
    }
    @keyframes dotPulse {
      0%,100% { transform: scale(1); box-shadow: 0 0 10px var(--c-accent); }
      50% { transform: scale(1.4); box-shadow: 0 0 22px var(--c-accent), 0 0 40px rgba(77,159,255,0.4); }
    }

    .nav-links {
      display: flex; gap: 36px; list-style: none;
    }
    .nav-links a {
      font-size: 13px; font-weight: 400;
      color: rgba(168, 200, 240, 0.6);
      text-decoration: none; letter-spacing: 0.03em;
      transition: color 0.2s ease;
    }
    .nav-links a:hover { color: white; }

    /* ─── Shooting stars ──────────────────────────────── */
    .shooting-star {
      position: fixed; pointer-events: none; z-index: 1;
      width: 2px; height: 2px; background: white;
      border-radius: 50%;
      box-shadow: 0 0 6px 1px rgba(255,255,255,0.8);
    }

    /* ─── Loading screen ──────────────────────────────── */
    #loader {
      position: fixed; inset: 0;
      background: var(--c-void);
      z-index: 1000;
      display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      gap: 24px;
    }

    .loader-ring {
      width: 64px; height: 64px;
      border: 1px solid rgba(77, 159, 255, 0.1);
      border-top-color: var(--c-accent);
      border-radius: 50%;
      animation: spin 1s linear infinite;
    }
    .loader-ring-2 {
      position: absolute;
      width: 80px; height: 80px;
      border: 1px solid rgba(124, 58, 237, 0.1);
      border-bottom-color: var(--c-violet);
      border-radius: 50%;
      animation: spin 1.8s linear infinite reverse;
    }

    @keyframes spin { to { transform: rotate(360deg); } }

    .loader-text {
      font-family: 'Space Mono', monospace;
      font-size: 11px; letter-spacing: 0.3em;
      color: rgba(168, 200, 240, 0.4);
      text-transform: uppercase;
      animation: fadeOscillate 1.5s ease-in-out infinite;
    }
    @keyframes fadeOscillate { 0%,100%{opacity:0.4} 50%{opacity:1} }

    .loader-progress {
      width: 200px; height: 1px;
      background: rgba(255,255,255,0.05);
      border-radius: 1px; overflow: hidden;
    }
    .loader-progress-fill {
      height: 100%;
      background: linear-gradient(to right, var(--c-accent), var(--c-violet));
      border-radius: 1px;
      width: 0%; transition: width 0.1s linear;
    }

    /* ─── Responsive ──────────────────────────────────── */
    @media (max-width: 768px) {
      .bento-grid { grid-template-columns: 1fr; max-width: 420px; }
      .stats-bar { grid-template-columns: repeat(2, 1fr); }
      #nav { padding: 20px 24px; }
      .nav-links { display: none; }
      .bento-card { aspect-ratio: 0.85; }
    }

    /* ─── Utility ─────────────────────────────────────── */
    .will-animate { will-change: transform, opacity; }
    ::selection { background: rgba(77, 159, 255, 0.3); }
  </style>
</head>

<body>

  <!-- Custom Cursor -->
  <div id="cursor-dot"></div>
  <div id="cursor-ring"></div>

  <!-- Scroll Progress -->
  <div id="progress-bar"></div>

  <!-- Loading Screen -->
  <div id="loader">
    <div style="position:relative;display:flex;align-items:center;justify-content:center;width:80px;height:80px;">
      <div class="loader-ring"></div>
      <div class="loader-ring-2"></div>
    </div>
    <div class="loader-text">Calibrating starfield</div>
    <div class="loader-progress"><div class="loader-progress-fill" id="loader-fill"></div></div>
  </div>

  <!-- Particle Canvas (fixed background) -->
  <canvas id="starfield-canvas"></canvas>

  <!-- Navigation -->
  <nav id="nav">
    <div class="nav-logo">
      <div class="nav-logo-dot"></div>
      COSMOS
    </div>
    <ul class="nav-links">
      <li><a href="#explore-reveal" id="nav-explore">Explore</a></li>
      <li><a href="#stars-live">Star Guide</a></li>
      <li><a href="#">Observatory</a></li>
      <li><a href="#">About</a></li>
    </ul>
  </nav>

  <!-- ═══ HERO SECTION ════════════════════════════════════ -->
  <section id="hero">
    <div class="nebula-aurora aurora-1" id="aurora-1"></div>
    <div class="nebula-aurora aurora-2" id="aurora-2"></div>
    <div class="nebula-aurora aurora-3" id="aurora-3"></div>

    <div style="display:flex;flex-direction:column;align-items:center;gap:28px;position:relative;z-index:2;">
      <div class="hero-eyebrow will-animate" id="hero-eyebrow">The Universe Encyclopedia</div>
      <h1 class="hero-title will-animate" id="hero-title">COSMOS</h1>
      <p class="hero-sub will-animate" id="hero-sub">
        Journey through the infinite. From the heartbeat of our Sun to the silence beyond supermassive black holes.
      </p>
    </div>

    <div class="hero-scroll-hint will-animate" id="scroll-hint">
      <div class="scroll-line"></div>
      <span class="scroll-label">Scroll to explore</span>
    </div>
  </section>

  <div id="scroll-spacer"></div>

  <!-- ═══ EXPLORE SECTION ══════════════════════════════════ -->
  <section id="explore-reveal">

    <div class="section-label will-animate" id="section-label">— Choose Your Journey —</div>

    <!-- Bento Grid -->
    <div class="bento-grid" id="bento-grid">

      <!-- Card 1: Solar System -->
      <div class="bento-card card-solar will-animate" id="card-solar" data-section="solar" tabindex="0" role="button" aria-label="Explore the Solar System">
        <div class="card-glow"></div>
        <div class="card-visual">
          <canvas class="card-canvas" id="canvas-solar"></canvas>
        </div>
        <div class="card-content">
          <span class="card-tag">8 Planets</span>
          <h2 class="card-title">Solar<br/>System</h2>
          <p class="card-desc">Our cosmic neighborhood — from the fiery corona of the Sun to icy Pluto at the edge of the void.</p>
        </div>
        <div class="card-arrow">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M7 17L17 7"/><path d="M7 7h10v10"/></svg>
        </div>
      </div>

      <!-- Card 2: Galaxies -->
      <div class="bento-card card-galaxy will-animate" id="card-galaxy" data-section="galaxy" tabindex="0" role="button" aria-label="Explore Galaxies">
        <div class="card-glow"></div>
        <div class="card-visual">
          <canvas class="card-canvas" id="canvas-galaxy"></canvas>
        </div>
        <div class="card-content">
          <span class="card-tag">2 Trillion Galaxies</span>
          <h2 class="card-title">Galaxies</h2>
          <p class="card-desc">Islands of stars beyond counting — spiral arms, elliptical giants, and colliding universes of light.</p>
        </div>
        <div class="card-arrow">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M7 17L17 7"/><path d="M7 7h10v10"/></svg>
        </div>
      </div>

      <!-- Card 3: Black Holes -->
      <div class="bento-card card-black-hole will-animate" id="card-blackhole" data-section="blackhole" tabindex="0" role="button" aria-label="Explore Black Holes">
        <div class="card-glow"></div>
        <div class="card-visual">
          <canvas class="card-canvas" id="canvas-blackhole"></canvas>
        </div>
        <div class="card-content">
          <span class="card-tag">∞ Gravity</span>
          <h2 class="card-title">Black<br/>Holes</h2>
          <p class="card-desc">Where spacetime ends. The most extreme objects in existence — born from dying stars, consuming light itself.</p>
        </div>
        <div class="card-arrow">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M7 17L17 7"/><path d="M7 7h10v10"/></svg>
        </div>
      </div>

    </div>

    <!-- Stats Bar -->
    <div class="stats-bar will-animate" id="stats-bar">
      <div class="stat-item">
        <div class="stat-value" data-target="200">0</div>
        <div class="stat-label">Billion stars in Milky Way</div>
      </div>
      <div class="stat-item">
        <div class="stat-value" data-target="5600">0</div>
        <div class="stat-label">Exoplanets confirmed</div>
      </div>
      <div class="stat-item">
        <div class="stat-value" data-target="93">0</div>
        <div class="stat-label">Billion light-years wide</div>
      </div>
      <div class="stat-item">
        <div class="stat-value" data-target="13">0</div>
        <div class="stat-label">Billion years since Big Bang</div>
      </div>
    </div>

  </section>

  <!-- Stars Live section placeholder (Part 3) -->
  <section id="stars-live" style="min-height:60px;position:relative;z-index:10;"></section>

  <!-- Detail Page (Part 2) -->
  <div id="detail-page"></div>

  <!-- ═══════════════════════════════════════════════════════
       JAVASCRIPT
  ══════════════════════════════════════════════════════════ -->
  <script>
  /* ─────────────────────────────────────────────────────────
     AUDIO ENGINE — Spatial UI sounds
  ───────────────────────────────────────────────────────── */
  class CosmosAudio {
    constructor() {
      this.ctx = null;
      this.enabled = true;
      this._init();
    }

    _init() {
      try {
        this.ctx = new (window.AudioContext || window.webkitAudioContext)();
      } catch(e) { this.enabled = false; }
    }

    _resume() {
      if (this.ctx && this.ctx.state === 'suspended') this.ctx.resume();
    }

    // Deep space ambient "chime" — for page load
    playDeepChime() {
      if (!this.enabled) return;
      this._resume();
      const freqs = [55, 110, 165, 220];
      freqs.forEach((freq, i) => {
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        const pan = this.ctx.createStereoPanner();
        osc.connect(gain); gain.connect(pan); pan.connect(this.ctx.destination);
        osc.type = 'sine';
        osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(freq * 1.5, this.ctx.currentTime + 3);
        pan.pan.value = (i / freqs.length) * 2 - 1;
        gain.gain.setValueAtTime(0, this.ctx.currentTime);
        gain.gain.linearRampToValueAtTime(0.04, this.ctx.currentTime + 0.5 + i * 0.3);
        gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + 4 + i * 0.5);
        osc.start(this.ctx.currentTime);
        osc.stop(this.ctx.currentTime + 5 + i * 0.5);
      });
    }

    // Stellar hover — crystalline ping
    playHover() {
      if (!this.enabled) return;
      this._resume();
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      const reverb = this._createReverb(0.8);
      osc.connect(gain); gain.connect(reverb); reverb.connect(this.ctx.destination);
      osc.type = 'sine';
      osc.frequency.setValueAtTime(880, this.ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(1320, this.ctx.currentTime + 0.15);
      gain.gain.setValueAtTime(0, this.ctx.currentTime);
      gain.gain.linearRampToValueAtTime(0.07, this.ctx.currentTime + 0.02);
      gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + 0.6);
      osc.start(this.ctx.currentTime);
      osc.stop(this.ctx.currentTime + 0.7);
    }

    // Warp — card click / section entry
    playWarp() {
      if (!this.enabled) return;
      this._resume();
      [200, 300, 450, 600].forEach((freq, i) => {
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        const pan = this.ctx.createStereoPanner();
        osc.connect(gain); gain.connect(pan); pan.connect(this.ctx.destination);
        osc.type = i % 2 === 0 ? 'sine' : 'triangle';
        osc.frequency.setValueAtTime(freq, this.ctx.currentTime + i * 0.06);
        osc.frequency.exponentialRampToValueAtTime(freq * 3, this.ctx.currentTime + 0.4 + i * 0.06);
        pan.pan.value = Math.cos(i * 1.2) * 0.7;
        gain.gain.setValueAtTime(0, this.ctx.currentTime + i * 0.06);
        gain.gain.linearRampToValueAtTime(0.06, this.ctx.currentTime + 0.05 + i * 0.06);
        gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + 0.5 + i * 0.08);
        osc.start(this.ctx.currentTime + i * 0.06);
        osc.stop(this.ctx.currentTime + 0.6 + i * 0.1);
      });
    }

    // Scroll reveal — soft whoosh
    playReveal() {
      if (!this.enabled) return;
      this._resume();
      const bufferSize = this.ctx.sampleRate * 0.5;
      const buf = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
      const data = buf.getChannelData(0);
      for (let i = 0; i < bufferSize; i++) data[i] = (Math.random() * 2 - 1) * 0.15;
      const src = this.ctx.createBufferSource();
      src.buffer = buf;
      const filter = this.ctx.createBiquadFilter();
      filter.type = 'bandpass';
      filter.frequency.value = 800;
      filter.Q.value = 2;
      const gain = this.ctx.createGain();
      src.connect(filter); filter.connect(gain); gain.connect(this.ctx.destination);
      gain.gain.setValueAtTime(0, this.ctx.currentTime);
      gain.gain.linearRampToValueAtTime(0.15, this.ctx.currentTime + 0.1);
      gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + 0.5);
      src.start(this.ctx.currentTime);
    }

    // Pulsar — stat counter tick
    playTick() {
      if (!this.enabled) return;
      this._resume();
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      osc.connect(gain); gain.connect(this.ctx.destination);
      osc.type = 'square';
      osc.frequency.value = 440 + Math.random() * 200;
      gain.gain.setValueAtTime(0.02, this.ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + 0.04);
      osc.start(this.ctx.currentTime);
      osc.stop(this.ctx.currentTime + 0.05);
    }

    _createReverb(seconds) {
      const convolver = this.ctx.createConvolver();
      const rate = this.ctx.sampleRate;
      const length = rate * seconds;
      const impulse = this.ctx.createBuffer(2, length, rate);
      for (let c = 0; c < 2; c++) {
        const channel = impulse.getChannelData(c);
        for (let i = 0; i < length; i++)
          channel[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, 2);
      }
      convolver.buffer = impulse;
      return convolver;
    }
  }

  const audio = new CosmosAudio();

  /* ─────────────────────────────────────────────────────────
     PARTICLE STARFIELD — Three.js WebGL
  ───────────────────────────────────────────────────────── */
  class Starfield {
    constructor(canvas) {
      this.canvas = canvas;
      this.mouse = { x: 0, y: 0, vx: 0, vy: 0, tx: 0, ty: 0 };
      this.touch = { active: false, x: 0, y: 0, vx: 0, vy: 0 };
      this.scroll = 0;
      this.time = 0;
      this._setup();
      this._createStars();
      this._createNebulaDust();
      this._bindEvents();
      this._animate();
    }

    _setup() {
      this.renderer = new THREE.WebGLRenderer({
        canvas: this.canvas,
        antialias: false,
        alpha: true,
        powerPreference: 'high-performance'
      });
      this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setClearColor(0x000000, 0);

      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 2000);
      this.camera.position.z = 400;

      window.addEventListener('resize', () => {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
      });
    }

    _createStars() {
      // Layer 1 — distant tiny white stars (5000)
      const count1 = 5000;
      const geo1 = new THREE.BufferGeometry();
      const pos1 = new Float32Array(count1 * 3);
      const col1 = new Float32Array(count1 * 3);
      for (let i = 0; i < count1; i++) {
        pos1[i*3]   = (Math.random() - 0.5) * 2400;
        pos1[i*3+1] = (Math.random() - 0.5) * 2400;
        pos1[i*3+2] = (Math.random() - 0.5) * 1200 - 200;
        const t = Math.random();
        const r = t > 0.85 ? 0.9 + Math.random() * 0.1 : 0.75 + Math.random() * 0.2;
        const g = t > 0.85 ? 0.85 + Math.random() * 0.15 : 0.82 + Math.random() * 0.15;
        const b = t > 0.85 ? 0.95 + Math.random() * 0.05 : 0.88 + Math.random() * 0.12;
        col1[i*3] = r; col1[i*3+1] = g; col1[i*3+2] = b;
      }
      geo1.setAttribute('position', new THREE.BufferAttribute(pos1, 3));
      geo1.setAttribute('color', new THREE.BufferAttribute(col1, 3));
      const mat1 = new THREE.PointsMaterial({ size: 1.2, vertexColors: true, sizeAttenuation: true, transparent: true, opacity: 0.8 });
      this.stars1 = new THREE.Points(geo1, mat1);
      this.scene.add(this.stars1);

      // Layer 2 — mid stars (1500)
      const count2 = 1500;
      const geo2 = new THREE.BufferGeometry();
      const pos2 = new Float32Array(count2 * 3);
      const col2 = new Float32Array(count2 * 3);
      for (let i = 0; i < count2; i++) {
        pos2[i*3]   = (Math.random() - 0.5) * 1800;
        pos2[i*3+1] = (Math.random() - 0.5) * 1800;
        pos2[i*3+2] = (Math.random() - 0.5) * 600;
        // Color tints: blue-white, warm white, blue
        const type = Math.random();
        if (type > 0.7) { col2[i*3]=0.65; col2[i*3+1]=0.82; col2[i*3+2]=1.0; }
        else if (type > 0.4) { col2[i*3]=1.0; col2[i*3+1]=0.96; col2[i*3+2]=0.85; }
        else { col2[i*3]=0.85; col2[i*3+1]=0.9; col2[i*3+2]=1.0; }
      }
      geo2.setAttribute('position', new THREE.BufferAttribute(pos2, 3));
      geo2.setAttribute('color', new THREE.BufferAttribute(col2, 3));
      const mat2 = new THREE.PointsMaterial({ size: 2.2, vertexColors: true, sizeAttenuation: true, transparent: true, opacity: 0.9 });
      this.stars2 = new THREE.Points(geo2, mat2);
      this.scene.add(this.stars2);

      // Layer 3 — large bright foreground stars (200)
      const count3 = 200;
      const geo3 = new THREE.BufferGeometry();
      const pos3 = new Float32Array(count3 * 3);
      const col3 = new Float32Array(count3 * 3);
      for (let i = 0; i < count3; i++) {
        pos3[i*3]   = (Math.random() - 0.5) * 1400;
        pos3[i*3+1] = (Math.random() - 0.5) * 1400;
        pos3[i*3+2] = (Math.random() - 0.5) * 300 + 50;
        const type = Math.floor(Math.random() * 4);
        if (type===0) { col3[i*3]=0.6; col3[i*3+1]=0.75; col3[i*3+2]=1.0; }
        else if (type===1) { col3[i*3]=1.0; col3[i*3+1]=0.95; col3[i*3+2]=0.8; }
        else if (type===2) { col3[i*3]=1.0; col3[i*3+1]=0.85; col3[i*3+2]=0.5; }
        else { col3[i*3]=0.9; col3[i*3+1]=0.9; col3[i*3+2]=1.0; }
      }
      geo3.setAttribute('position', new THREE.BufferAttribute(pos3, 3));
      geo3.setAttribute('color', new THREE.BufferAttribute(col3, 3));
      const mat3 = new THREE.PointsMaterial({ size: 3.8, vertexColors: true, sizeAttenuation: true, transparent: true, opacity: 1.0 });
      this.stars3 = new THREE.Points(geo3, mat3);
      this.scene.add(this.stars3);

      this._rawPos2 = pos2;
      this._rawPos3 = pos3;
      this._twinkleOffsets = Array.from({length: count2}, () => Math.random() * Math.PI * 2);
    }

    _createNebulaDust() {
      const count = 600;
      const geo = new THREE.BufferGeometry();
      const pos = new Float32Array(count * 3);
      const col = new Float32Array(count * 3);
      const palettes = [
        [0.3, 0.5, 1.0],   // blue
        [0.5, 0.2, 0.9],   // violet
        [0.1, 0.6, 0.8],   // teal
        [0.9, 0.4, 0.1],   // orange
      ];
      for (let i = 0; i < count; i++) {
        pos[i*3]   = (Math.random() - 0.5) * 1800;
        pos[i*3+1] = (Math.random() - 0.5) * 1800;
        pos[i*3+2] = -200 - Math.random() * 400;
        const p = palettes[Math.floor(Math.random() * palettes.length)];
        const dim = 0.12 + Math.random() * 0.18;
        col[i*3] = p[0] * dim; col[i*3+1] = p[1] * dim; col[i*3+2] = p[2] * dim;
      }
      geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
      geo.setAttribute('color', new THREE.BufferAttribute(col, 3));
      const mat = new THREE.PointsMaterial({ size: 28, vertexColors: true, sizeAttenuation: true, transparent: true, opacity: 0.25 });
      this.nebula = new THREE.Points(geo, mat);
      this.scene.add(this.nebula);
    }

    _bindEvents() {
      document.addEventListener('mousemove', (e) => {
        this.mouse.tx = (e.clientX / window.innerWidth  - 0.5) * 2;
        this.mouse.ty = (e.clientY / window.innerHeight - 0.5) * 2;
      });

      let lastTouchX = 0, lastTouchY = 0;
      document.addEventListener('touchstart', (e) => {
        const t = e.touches[0];
        lastTouchX = t.clientX; lastTouchY = t.clientY;
        this.touch.active = true;
      });
      document.addEventListener('touchmove', (e) => {
        const t = e.touches[0];
        this.touch.vx = (t.clientX - lastTouchX) * 0.003;
        this.touch.vy = (t.clientY - lastTouchY) * 0.003;
        this.touch.x += this.touch.vx;
        this.touch.y += this.touch.vy;
        lastTouchX = t.clientX; lastTouchY = t.clientY;
      });
      document.addEventListener('touchend', () => { this.touch.active = false; });

      window.addEventListener('scroll', () => {
        this.scroll = window.scrollY / (document.body.scrollHeight - window.innerHeight);
      });
    }

    _animate() {
      requestAnimationFrame(() => this._animate());
      this.time += 0.008;

      // Mouse inertia
      this.mouse.vx += (this.mouse.tx - this.mouse.x) * 0.04;
      this.mouse.vy += (this.mouse.ty - this.mouse.y) * 0.04;
      this.mouse.vx *= 0.88;
      this.mouse.vy *= 0.88;
      this.mouse.x += this.mouse.vx;
      this.mouse.y += this.mouse.vy;

      const mx = this.touch.active ? this.touch.x : this.mouse.x;
      const my = this.touch.active ? this.touch.y : this.mouse.y;

      // Gentle parallax rotation
      this.stars1.rotation.y = mx * 0.015 + this.time * 0.003;
      this.stars1.rotation.x = my * 0.010;
      this.stars2.rotation.y = mx * 0.025 + this.time * 0.005;
      this.stars2.rotation.x = my * 0.018;
      this.stars3.rotation.y = mx * 0.04 + this.time * 0.007;
      this.stars3.rotation.x = my * 0.03;
      this.nebula.rotation.y = mx * 0.008 + this.time * 0.002;
      this.nebula.rotation.x = my * 0.005;

      // Twinkling
      const pos2 = this.stars2.geometry.attributes.position.array;
      this._twinkleOffsets.forEach((off, i) => {
        const twinkle = 0.5 + 0.5 * Math.sin(this.time * 1.5 + off);
        this.stars2.material.opacity = 0.7 + twinkle * 0.3;
      });

      // Scroll parallax — pull camera forward
      this.camera.position.z = 400 - this.scroll * 120;

      this.renderer.render(this.scene, this.camera);
    }
  }

  /* ─────────────────────────────────────────────────────────
     CARD CANVASES — Mini animated objects on each card
  ───────────────────────────────────────────────────────── */

  // Solar System Card Canvas
  function animateSolarCard(canvas) {
    const ctx = canvas.getContext('2d');
    let t = 0;
    const planets = [
      { name: 'Mercury', a: 38, b: 12,  speed: 4.1,  r: 3,  color: '#b5b5b5', phase: 0   },
      { name: 'Venus',   a: 58, b: 18,  speed: 1.6,  r: 5,  color: '#e8c87a', phase: 1.2 },
      { name: 'Earth',   a: 80, b: 26,  speed: 1.0,  r: 5.5,color: '#4d9fff', phase: 2.1 },
      { name: 'Mars',    a: 104,b: 34,  speed: 0.53, r: 4.2,color: '#d85e3e', phase: 0.8 },
      { name: 'Jupiter', a: 134,b: 44,  speed: 0.084,r: 10, color: '#c8a876', phase: 3.5 },
    ];

    function draw() {
      const W = canvas.width, H = canvas.height;
      ctx.clearRect(0, 0, W, H);
      const cx = W * 0.5, cy = H * 0.58;

      // Orbits
      planets.forEach(p => {
        ctx.beginPath();
        ctx.ellipse(cx, cy, p.a, p.b, 0, 0, Math.PI * 2);
        ctx.strokeStyle = 'rgba(255,255,255,0.06)';
        ctx.lineWidth = 0.8;
        ctx.stroke();
      });

      // Sun
      const sunGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, 22);
      sunGrad.addColorStop(0, '#fffbe0');
      sunGrad.addColorStop(0.3, '#ffdd60');
      sunGrad.addColorStop(0.7, '#ff8c00');
      sunGrad.addColorStop(1, 'transparent');
      ctx.beginPath();
      ctx.arc(cx, cy, 22, 0, Math.PI*2);
      ctx.fillStyle = sunGrad;
      ctx.fill();

      // Corona pulse
      const coronaGrad = ctx.createRadialGradient(cx, cy, 16, cx, cy, 38 + 4*Math.sin(t*2));
      coronaGrad.addColorStop(0, 'rgba(255, 200, 60, 0.15)');
      coronaGrad.addColorStop(1, 'transparent');
      ctx.beginPath();
      ctx.arc(cx, cy, 38 + 4*Math.sin(t*2), 0, Math.PI*2);
      ctx.fillStyle = coronaGrad;
      ctx.fill();

      // Planets
      planets.forEach(p => {
        const angle = p.phase + t * p.speed;
        const x = cx + p.a * Math.cos(angle);
        const y = cy + p.b * Math.sin(angle);

        // Glow
        const gGrad = ctx.createRadialGradient(x, y, 0, x, y, p.r * 2.5);
        gGrad.addColorStop(0, p.color);
        gGrad.addColorStop(1, 'transparent');
        ctx.beginPath();
        ctx.arc(x, y, p.r * 2.5, 0, Math.PI*2);
        ctx.fillStyle = gGrad;
        ctx.fill();

        ctx.beginPath();
        ctx.arc(x, y, p.r, 0, Math.PI*2);
        ctx.fillStyle = p.color;
        ctx.fill();

        // Jupiter ring
        if (p.name === 'Jupiter') {
          ctx.save();
          ctx.translate(x, y);
          ctx.rotate(0.3);
          ctx.scale(1, 0.28);
          ctx.beginPath();
          ctx.arc(0, 0, p.r * 2.1, 0, Math.PI*2);
          ctx.strokeStyle = 'rgba(200, 168, 118, 0.4)';
          ctx.lineWidth = 1.5;
          ctx.stroke();
          ctx.restore();
        }
      });

      // Asteroid belt hint
      for (let i = 0; i < 40; i++) {
        const angle = (i/40)*Math.PI*2 + t * 0.02;
        const d = 118 + (Math.sin(i*7.3)*5);
        const ax = cx + d * Math.cos(angle);
        const ay = cy + (d*0.32) * Math.sin(angle);
        ctx.beginPath();
        ctx.arc(ax, ay, 0.7, 0, Math.PI*2);
        ctx.fillStyle = 'rgba(180, 160, 140, 0.3)';
        ctx.fill();
      }

      t += 0.006;
      requestAnimationFrame(draw);
    }
    function resize() {
      canvas.width = canvas.offsetWidth;
      canvas.height = canvas.offsetHeight;
    }
    resize();
    window.addEventListener('resize', resize);
    draw();
  }

  // Galaxy Card Canvas
  function animateGalaxyCard(canvas) {
    const ctx = canvas.getContext('2d');
    let t = 0;
    const ARM_COUNT = 4;
    const STARS_PER_ARM = 120;

    function draw() {
      const W = canvas.width, H = canvas.height;
      ctx.clearRect(0, 0, W, H);
      const cx = W*0.5, cy = H*0.5;
      const scale = Math.min(W, H) * 0.42;

      // Core glow
      const coreGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, scale*0.22);
      coreGrad.addColorStop(0, 'rgba(220, 200, 255, 0.9)');
      coreGrad.addColorStop(0.3, 'rgba(160, 120, 220, 0.5)');
      coreGrad.addColorStop(0.7, 'rgba(80, 40, 160, 0.2)');
      coreGrad.addColorStop(1, 'transparent');
      ctx.beginPath();
      ctx.ellipse(cx, cy, scale*0.22, scale*0.1, t*0.1, 0, Math.PI*2);
      ctx.fillStyle = coreGrad;
      ctx.fill();

      // Spiral arms
      for (let arm = 0; arm < ARM_COUNT; arm++) {
        const armOffset = (arm / ARM_COUNT) * Math.PI * 2;
        for (let i = 0; i < STARS_PER_ARM; i++) {
          const frac = i / STARS_PER_ARM;
          const radius = frac * scale;
          const angle = armOffset + frac * Math.PI * 3.5 + t * (0.15 - frac*0.1);
          const spread = frac * scale * 0.18 + 2;
          const sx = cx + radius * Math.cos(angle) + (Math.random()-0.5)*spread*0.05;
          const sy = cy + radius * Math.sin(angle) * 0.38 + (Math.random()-0.5)*spread*0.05;
          const alpha = (1 - frac) * 0.85 + 0.1;
          const size = (1 - frac) * 3 + 0.4 + Math.random()*0.6;

          // Color by arm
          const armColors = ['#c8a8ff', '#a8d4ff', '#ffd8a8', '#a8ffd4'];
          ctx.beginPath();
          ctx.arc(sx, sy, size, 0, Math.PI*2);
          ctx.fillStyle = armColors[arm].replace(')', `,${alpha})`).replace('rgb', 'rgba');
          // parse hex to rgba
          const hex = armColors[arm];
          const ri = parseInt(hex.slice(1,3),16);
          const gi = parseInt(hex.slice(3,5),16);
          const bi = parseInt(hex.slice(5,7),16);
          ctx.fillStyle = `rgba(${ri},${gi},${bi},${alpha})`;
          ctx.fill();
        }
      }

      t += 0.003;
      requestAnimationFrame(draw);
    }
    function resize() { canvas.width = canvas.offsetWidth; canvas.height = canvas.offsetHeight; }
    resize(); window.addEventListener('resize', resize); draw();
  }

  // Black Hole Card Canvas
  function animateBlackholeCard(canvas) {
    const ctx = canvas.getContext('2d');
    let t = 0;

    function draw() {
      const W = canvas.width, H = canvas.height;
      ctx.clearRect(0, 0, W, H);
      const cx = W*0.5, cy = H*0.48;
      const R = Math.min(W, H) * 0.28;

      // Accretion disk outer glow
      const diskGrad = ctx.createRadialGradient(cx, cy, R*0.6, cx, cy, R*2.2);
      diskGrad.addColorStop(0, 'rgba(255, 160, 40, 0.3)');
      diskGrad.addColorStop(0.3, 'rgba(255, 80, 20, 0.2)');
      diskGrad.addColorStop(0.6, 'rgba(6, 182, 212, 0.12)');
      diskGrad.addColorStop(1, 'transparent');
      ctx.save();
      ctx.scale(1, 0.3);
      ctx.beginPath();
      ctx.arc(cx, cy/0.3, R*2.2, 0, Math.PI*2);
      ctx.fillStyle = diskGrad;
      ctx.fill();
      ctx.restore();

      // Accretion disk rings
      for (let ring = 5; ring > 0; ring--) {
        const rr = R * (1.2 + ring * 0.18);
        const alpha = 0.08 + (ring/5)*0.12;
        const hot = ring > 3;
        ctx.save();
        ctx.scale(1, 0.28);
        ctx.beginPath();
        ctx.ellipse(cx, cy/0.28, rr, rr, t * (0.3 + ring*0.05), 0, Math.PI*2);
        ctx.strokeStyle = hot ? `rgba(255,${100+ring*20},40,${alpha})` : `rgba(6,182,212,${alpha})`;
        ctx.lineWidth = 2 + ring*0.8;
        ctx.stroke();
        ctx.restore();
      }

      // Bright hot side (Doppler)
      ctx.save();
      ctx.scale(1, 0.28);
      const brightGrad = ctx.createRadialGradient(cx - R*1.4, cy/0.28, 0, cx - R*1.4, cy/0.28, R*0.8);
      brightGrad.addColorStop(0, 'rgba(255, 220, 100, 0.5)');
      brightGrad.addColorStop(1, 'transparent');
      ctx.beginPath();
      ctx.arc(cx - R*1.4, cy/0.28, R*0.8, 0, Math.PI*2);
      ctx.fillStyle = brightGrad;
      ctx.fill();
      ctx.restore();

      // Gravitational lensing arcs
      ctx.save();
      ctx.scale(1, 0.32);
      for (let arc = 0; arc < 3; arc++) {
        ctx.beginPath();
        ctx.ellipse(cx, cy/0.32, R*(1.18 + arc*0.05), R*(1.18 + arc*0.05), 0, 0, Math.PI*2);
        ctx.strokeStyle = `rgba(200,240,255,${0.15 - arc*0.04})`;
        ctx.lineWidth = 1;
        ctx.stroke();
      }
      ctx.restore();

      // Event horizon (black disc with gradient edge)
      const bhGrad = ctx.createRadialGradient(cx-R*0.15, cy-R*0.08, 0, cx, cy, R*1.02);
      bhGrad.addColorStop(0, '#000000');
      bhGrad.addColorStop(0.75, '#000000');
      bhGrad.addColorStop(0.88, 'rgba(0,0,0,0.92)');
      bhGrad.addColorStop(1, 'rgba(0,0,0,0)');
      ctx.beginPath();
      ctx.arc(cx, cy, R*1.02, 0, Math.PI*2);
      ctx.fillStyle = bhGrad;
      ctx.fill();

      // Photon sphere — thin bright ring
      const photonGrad = ctx.createRadialGradient(cx, cy, R*0.96, cx, cy, R*1.08);
      photonGrad.addColorStop(0, 'rgba(255, 200, 120, 0)');
      photonGrad.addColorStop(0.4, 'rgba(255, 200, 120, 0.4)');
      photonGrad.addColorStop(0.7, 'rgba(255, 180, 60, 0.6)');
      photonGrad.addColorStop(1, 'rgba(255, 160, 40, 0)');
      ctx.beginPath();
      ctx.arc(cx, cy, R*1.04, 0, Math.PI*2);
      ctx.strokeStyle = 'rgba(255, 190, 60, 0.5)';
      ctx.lineWidth = 1.5;
      ctx.stroke();

      // Particles spiraling in
      for (let i = 0; i < 24; i++) {
        const frac = ((i/24) + t * 0.04) % 1;
        const spiral = frac * Math.PI * 6;
        const r = R * (1.05 + (1-frac) * 2.2);
        const px = cx + r * Math.cos(spiral) * 0.88;
        const py = cy + r * 0.28 * Math.sin(spiral);
        const alpha = Math.max(0, frac * 1.4 - 0.4);
        ctx.beginPath();
        ctx.arc(px, py, 1.2, 0, Math.PI*2);
        ctx.fillStyle = `rgba(255, 200, 80, ${alpha * 0.7})`;
        ctx.fill();
      }

      t += 0.004;
      requestAnimationFrame(draw);
    }
    function resize() { canvas.width = canvas.offsetWidth; canvas.height = canvas.offsetHeight; }
    resize(); window.addEventListener('resize', resize); draw();
  }

  /* ─────────────────────────────────────────────────────────
     CUSTOM CURSOR
  ───────────────────────────────────────────────────────── */
  const cursorDot  = document.getElementById('cursor-dot');
  const cursorRing = document.getElementById('cursor-ring');
  let cursorX = 0, cursorY = 0;
  let ringX = 0, ringY = 0;

  document.addEventListener('mousemove', (e) => {
    cursorX = e.clientX; cursorY = e.clientY;
    cursorDot.style.left = cursorX + 'px';
    cursorDot.style.top  = cursorY + 'px';
  });

  function animateCursor() {
    ringX += (cursorX - ringX) * 0.12;
    ringY += (cursorY - ringY) * 0.12;
    cursorRing.style.left = ringX + 'px';
    cursorRing.style.top  = ringY + 'px';
    requestAnimationFrame(animateCursor);
  }
  animateCursor();

  // Hover effect
  document.querySelectorAll('a, button, .bento-card, [role="button"]').forEach(el => {
    el.addEventListener('mouseenter', () => { document.body.classList.add('hovering'); audio.playHover(); });
    el.addEventListener('mouseleave', () => document.body.classList.remove('hovering'));
  });

  /* ─────────────────────────────────────────────────────────
     SCROLL PROGRESS
  ───────────────────────────────────────────────────────── */
  const progressBar = document.getElementById('progress-bar');
  const nav = document.getElementById('nav');
  window.addEventListener('scroll', () => {
    const pct = window.scrollY / (document.body.scrollHeight - window.innerHeight) * 100;
    progressBar.style.width = pct + '%';
    nav.classList.toggle('scrolled', window.scrollY > 60);
  });

  /* ─────────────────────────────────────────────────────────
     SHOOTING STARS
  ───────────────────────────────────────────────────────── */
  function spawnShootingStar() {
    const star = document.createElement('div');
    star.className = 'shooting-star';
    const startX = Math.random() * window.innerWidth;
    const startY = Math.random() * (window.innerHeight * 0.6);
    const angle = 20 + Math.random() * 30;
    const dist  = 200 + Math.random() * 400;
    star.style.cssText = `left:${startX}px;top:${startY}px;width:${1+Math.random()}px;height:${1+Math.random()}px;`;
    document.body.appendChild(star);

    const trail = document.createElement('div');
    trail.style.cssText = `
      position:fixed;left:${startX}px;top:${startY}px;
      width:${80+Math.random()*120}px;height:1px;
      background:linear-gradient(to right,transparent,rgba(255,255,255,0.6),transparent);
      transform:rotate(${angle}deg);transform-origin:left center;
      pointer-events:none;z-index:1;
      transition:opacity 0.4s ease;
    `;
    document.body.appendChild(trail);

    gsap.to(star, {
      x: Math.cos(angle * Math.PI/180) * dist,
      y: Math.sin(angle * Math.PI/180) * dist,
      duration: 0.6 + Math.random() * 0.5,
      ease: 'power1.out',
      opacity: 0,
      onComplete: () => { star.remove(); trail.remove(); }
    });
    gsap.to(trail, { opacity: 0, duration: 0.4, delay: 0.2, onComplete: () => trail.remove() });
  }

  setInterval(spawnShootingStar, 3200 + Math.random() * 2000);

  /* ─────────────────────────────────────────────────────────
     LOADER & ENTRY SEQUENCE
  ───────────────────────────────────────────────────────── */
  function initApp() {
    const loader = document.getElementById('loader');
    const fill   = document.getElementById('loader-fill');
    let progress = 0;

    // Fake progressive load
    const loadInterval = setInterval(() => {
      progress += Math.random() * 18 + 4;
      if (progress >= 100) { progress = 100; clearInterval(loadInterval); }
      fill.style.width = progress + '%';
    }, 120);

    setTimeout(() => {
      // Init Three.js starfield
      const starfieldCanvas = document.getElementById('starfield-canvas');
      new Starfield(starfieldCanvas);

      // Hide loader
      gsap.to(loader, {
        opacity: 0, duration: 0.6, ease: 'power2.out',
        onComplete: () => { loader.style.display = 'none'; }
      });

      // Reveal nav
      gsap.to('#nav', { opacity: 1, duration: 0.8, delay: 0.3 });

      // Aurora glow
      gsap.to(['.aurora-1', '.aurora-2', '.aurora-3'], {
        opacity: 1, duration: 2.5, stagger: 0.3, ease: 'power2.out', delay: 0.5
      });

      // Hero sequence — Apple-style staggered reveal
      const tl = gsap.timeline({ delay: 0.6 });
      tl.to('#hero-eyebrow', {
        opacity: 1, y: 0, duration: 0.7, ease: 'power3.out'
      })
      .to('#hero-title', {
        opacity: 1, y: 0, scale: 1, duration: 1.1, ease: 'power3.out'
      }, '-=0.3')
      .to('#hero-sub', {
        opacity: 1, y: 0, duration: 0.8, ease: 'power2.out'
      }, '-=0.5')
      .to('#scroll-hint', {
        opacity: 1, duration: 0.8, ease: 'power2.out',
        onStart: () => audio.playDeepChime()
      }, '-=0.2');

      // Init card canvases
      setTimeout(() => {
        animateSolarCard(document.getElementById('canvas-solar'));
        animateGalaxyCard(document.getElementById('canvas-galaxy'));
        animateBlackholeCard(document.getElementById('canvas-blackhole'));
      }, 400);

      // ScrollTrigger setup
      setupScrollTriggers();

    }, 1600);
  }

  /* ─────────────────────────────────────────────────────────
     GSAP SCROLL TRIGGERS
  ───────────────────────────────────────────────────────── */
  function setupScrollTriggers() {
    gsap.registerPlugin(ScrollTrigger);

    // Hero title shrinks + fades on scroll
    gsap.to('#hero-title', {
      scale: 0.65,
      opacity: 0,
      y: -80,
      ease: 'none',
      scrollTrigger: {
        trigger: '#hero',
        start: 'top top',
        end: 'bottom top',
        scrub: 1.2,
      }
    });

    gsap.to('#hero-sub', {
      opacity: 0, y: -40,
      ease: 'none',
      scrollTrigger: {
        trigger: '#hero',
        start: 'top top',
        end: '60% top',
        scrub: 1,
      }
    });

    gsap.to('#hero-eyebrow', {
      opacity: 0, y: -20,
      ease: 'none',
      scrollTrigger: {
        trigger: '#hero',
        start: 'top top',
        end: '40% top',
        scrub: 1,
      }
    });

    gsap.to('#scroll-hint', {
      opacity: 0,
      ease: 'none',
      scrollTrigger: {
        trigger: '#hero',
        start: 'top top',
        end: '25% top',
        scrub: 1,
      }
    });

    // Section label
    ScrollTrigger.create({
      trigger: '#explore-reveal',
      start: 'top 80%',
      onEnter: () => {
        gsap.to('#section-label', {
          opacity: 1, y: 0, duration: 0.6, ease: 'power2.out',
          onStart: () => audio.playReveal()
        });
      }
    });

    // Bento cards stagger reveal
    ScrollTrigger.create({
      trigger: '#bento-grid',
      start: 'top 75%',
      onEnter: () => {
        gsap.to('.bento-card', {
          opacity: 1, y: 0,
          duration: 0.9,
          ease: 'cubic-bezier(0.25,1,0.5,1)',
          stagger: 0.18,
          onStart: () => audio.playReveal()
        });
      }
    });

    // Stats bar reveal + counter animation
    ScrollTrigger.create({
      trigger: '#stats-bar',
      start: 'top 85%',
      onEnter: () => {
        gsap.to('#stats-bar', {
          opacity: 1, y: 0, duration: 0.8, ease: 'power2.out'
        });

        // Animate stat counters
        document.querySelectorAll('.stat-value').forEach((el, i) => {
          const target = parseInt(el.dataset.target);
          const obj = { val: 0 };
          gsap.to(obj, {
            val: target,
            duration: 2.5,
            delay: i * 0.2,
            ease: 'power2.out',
            onUpdate: () => {
              el.textContent = Math.round(obj.val).toLocaleString();
              if (Math.round(obj.val) % 5 === 0) audio.playTick();
            },
            onComplete: () => { el.textContent = target.toLocaleString() + (target === 200 ? 'B+' : target === 13 ? '.8B' : '+'); }
          });
        });
      }
    });
  }

  /* ─────────────────────────────────────────────────────────
     CARD INTERACTIONS
  ───────────────────────────────────────────────────────── */
  document.querySelectorAll('.bento-card').forEach(card => {
    card.addEventListener('click', () => {
      audio.playWarp();
      const section = card.dataset.section;

      // Pulse animation before navigation
      gsap.to(card, {
        scale: 0.97,
        duration: 0.12,
        ease: 'power2.out',
        yoyo: true,
        repeat: 1,
        onComplete: () => {
          // Part 2 will handle the full expansion
          // For now: visual indicator
          const arrow = card.querySelector('.card-arrow');
          gsap.to(arrow, { x: 8, y: -8, opacity: 0, duration: 0.3 });
          gsap.to(card, {
            scale: 1.03,
            borderColor: 'rgba(255,255,255,0.25)',
            duration: 0.4, ease: 'power2.out'
          });

          setTimeout(() => {
            gsap.to(card, { scale: 1, borderColor: 'rgba(255,255,255,0.06)', duration: 0.5, ease: 'power3.out' });
            gsap.to(arrow, { x: 0, y: 0, opacity: 1, duration: 0.3, delay: 0.2 });
            alert(`🚀 "${section.toUpperCase()}" section — Ready for Part 2: Full detail page expansion!`);
          }, 600);
        }
      });
    });

    // Keyboard support
    card.addEventListener('keydown', (e) => { if (e.key === 'Enter' || e.key === ' ') card.click(); });
  });

  /* ─────────────────────────────────────────────────────────
     BOOT
  ───────────────────────────────────────────────────────── */
  window.addEventListener('load', initApp);
  </script>

</body>
</html>
