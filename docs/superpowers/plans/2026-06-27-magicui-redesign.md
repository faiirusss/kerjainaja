# MagicUI Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reskin seluruh halaman KerjainAja dengan bahasa visual MagicUI "SkyAgent" (hero center + pill, blur-fade scroll, marquee, bento, number ticker, glow cards) sambil mempertahankan aksen oranye brand, tanpa menambah framework.

**Architecture:** Tambah token + utilitas reusable di `global.css`, aktifkan via 1 script kecil di `Layout.astro` (IntersectionObserver reveal + number ticker), lalu terapkan utilitas itu di Header, Footer, dan 5 halaman. Semua animasi pure CSS/vanilla JS, progressive enhancement, hormati `prefers-reduced-motion`.

**Tech Stack:** Astro 6 (statis), CSS murni, JavaScript vanilla. TANPA React/Framer Motion/dependency baru.

## Global Constraints

- Tetap Astro `^6.4.7` statis — JANGAN tambah dependency framework UI.
- Aksen brand oranye `#ff4f00` dipertahankan; base warm-white; light mode saja.
- JANGAN ubah konten artikel, JSON-LD/schema, struktur URL, `content.config.ts`.
- JANGAN ubah `<form>` action di `kontak.astro` (tetap `method="POST" action="/kontak" netlify`).
- Semua animasi adalah enhancement: tanpa JS konten tetap tampil penuh; `prefers-reduced-motion: reduce` mematikan animasi.
- Markup tetap semantik & aksesibel (urutan heading, `aria-*`, focus state terlihat).
- Verifikasi tiap task: `npm run build` sukses (tanpa error). Verifikasi visual akhir via `npm run dev`.

---

### Task 1: Design tokens & utilitas global

**Files:**
- Modify: `src/styles/global.css` (tambah token di `:root`, tambah blok utilitas, tambah blok `prefers-reduced-motion`)

**Interfaces:**
- Produces (dipakai semua task berikut): token `--radius-lg`, `--radius-xl`, `--color-border-soft`, `--color-border-hair`, `--shadow-card`, `--shadow-card-hover`, `--shadow-glow`, `--gradient-primary`; class `.text-gradient`, `.bg-grid`, `.bg-dots`, `.fade-mask`, `.pill`, `.pill-dot`, `.glow-card`, `.marquee`, `.marquee-track`; attribute hook `[data-reveal]` / `.is-visible`.

- [ ] **Step 1: Tambah token baru ke `:root`** (setelah blok Container, sebelum `/* ── Reset ── */`)

```css
  /* MagicUI additions */
  --radius-lg: 16px;
  --radius-xl: 24px;

  --color-border-soft: rgba(32, 21, 21, 0.08);
  --color-border-hair: rgba(32, 21, 21, 0.055);

  --shadow-card: 0 1px 2px rgba(32, 21, 21, 0.04), 0 10px 30px rgba(32, 21, 21, 0.05);
  --shadow-card-hover: 0 2px 6px rgba(32, 21, 21, 0.06), 0 18px 44px rgba(255, 79, 0, 0.10);
  --shadow-glow: 0 0 90px rgba(255, 79, 0, 0.22);

  --gradient-primary: linear-gradient(120deg, #ff4f00 0%, #ff8a3d 100%);
```

- [ ] **Step 2: Tighten heading tracking** — ubah `h1` & `h2` di blok Typography agar lebih MagicUI:
  - `h1`: tambahkan `font-weight: 600;` (ganti dari 500) dan `letter-spacing: -1.5px;`
  - `h2`: tambahkan `font-weight: 600;` (ganti dari 500) dan `letter-spacing: -1px;`

- [ ] **Step 3: Tambah blok utilitas baru** (sebelum `/* ── Responsive ── */`)

```css
/* ── MagicUI Utilities ── */
.text-gradient {
  background: var(--gradient-primary);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;
}

/* Backgrounds */
.bg-grid {
  background-image:
    linear-gradient(to right, var(--color-border-hair) 1px, transparent 1px),
    linear-gradient(to bottom, var(--color-border-hair) 1px, transparent 1px);
  background-size: 56px 56px;
}
.bg-dots {
  background-image: radial-gradient(var(--color-border-soft) 1px, transparent 1px);
  background-size: 22px 22px;
}
/* Radial fade for decorative bg layers placed behind content */
.fade-mask {
  -webkit-mask-image: radial-gradient(ellipse 70% 60% at 50% 35%, #000 40%, transparent 78%);
  mask-image: radial-gradient(ellipse 70% 60% at 50% 35%, #000 40%, transparent 78%);
}

/* Announcement pill */
.pill {
  display: inline-flex;
  align-items: center;
  gap: var(--space-sm);
  background: var(--color-canvas);
  border: 1px solid var(--color-border-soft);
  border-radius: var(--radius-pill);
  padding: 5px 14px 5px 6px;
  font-size: 13px;
  font-weight: 500;
  line-height: 20px;
  color: var(--color-ink-soft);
  box-shadow: var(--shadow-soft);
}
.pill-dot {
  width: 18px; height: 18px;
  display: inline-grid; place-items: center;
  border-radius: var(--radius-pill);
  background: rgba(255, 79, 0, 0.12);
  color: var(--color-primary);
  font-size: 11px;
}

/* Glow card */
.glow-card {
  position: relative;
  background: var(--color-canvas);
  border: 1px solid var(--color-border-soft);
  border-radius: var(--radius-lg);
  transition: transform 0.25s ease, box-shadow 0.25s ease, border-color 0.25s ease;
}
.glow-card:hover {
  transform: translateY(-4px);
  border-color: rgba(255, 79, 0, 0.35);
  box-shadow: var(--shadow-card-hover);
}

/* Marquee */
.marquee {
  display: flex;
  overflow: hidden;
  width: 100%;
  -webkit-mask-image: linear-gradient(to right, transparent, #000 10%, #000 90%, transparent);
  mask-image: linear-gradient(to right, transparent, #000 10%, #000 90%, transparent);
}
.marquee-track {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  gap: var(--space-4xl);
  padding-right: var(--space-4xl);
  animation: marquee 30s linear infinite;
  white-space: nowrap;
}
.marquee:hover .marquee-track { animation-play-state: paused; }
@keyframes marquee {
  from { transform: translateX(0); }
  to   { transform: translateX(-100%); }
}

/* Reveal on scroll (blur-fade) */
[data-reveal] {
  opacity: 0;
  transform: translateY(24px);
  filter: blur(8px);
  transition: opacity 0.7s cubic-bezier(0.22, 1, 0.36, 1),
              transform 0.7s cubic-bezier(0.22, 1, 0.36, 1),
              filter 0.7s cubic-bezier(0.22, 1, 0.36, 1);
  transition-delay: var(--reveal-delay, 0s);
}
[data-reveal].is-visible {
  opacity: 1;
  transform: translateY(0);
  filter: blur(0);
}
```

- [ ] **Step 4: Tambah blok reduced-motion** (di akhir file)

```css
/* ── Reduced Motion ── */
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  [data-reveal] { opacity: 1 !important; transform: none !important; filter: none !important; transition: none !important; }
  .marquee-track { animation: none !important; }
}
```

- [ ] **Step 5: Verify build** — Run: `npm run build` — Expected: build sukses tanpa error.

---

### Task 2: Aktivator animasi di Layout

**Files:**
- Modify: `src/layouts/Layout.astro` (tambah script reveal + ticker; pertahankan script wrap tabel)

**Interfaces:**
- Consumes: `[data-reveal]`/`.is-visible` dari Task 1.
- Produces: aktivasi `[data-reveal]`; number ticker untuk `[data-ticker]` (atribut `data-ticker="<num>"`, opsional `data-suffix="+"`/`"%"`).

- [ ] **Step 1: Tambah script** di dalam `<body>` setelah script wrap tabel yang ada (jangan hapus yang lama):

```html
    <script>
      const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

      // Blur-fade reveal
      const revealEls = document.querySelectorAll('[data-reveal]');
      if ('IntersectionObserver' in window && !reduce) {
        const io = new IntersectionObserver((entries) => {
          entries.forEach((entry) => {
            if (entry.isIntersecting) {
              entry.target.classList.add('is-visible');
              io.unobserve(entry.target);
            }
          });
        }, { rootMargin: '0px 0px -8% 0px', threshold: 0.12 });
        revealEls.forEach((el) => io.observe(el));
      } else {
        revealEls.forEach((el) => el.classList.add('is-visible'));
      }

      // Number ticker
      const tickers = document.querySelectorAll('[data-ticker]');
      const runTicker = (el) => {
        const target = parseFloat(el.getAttribute('data-ticker')) || 0;
        const suffix = el.getAttribute('data-suffix') || '';
        if (reduce) { el.textContent = String(target) + suffix; return; }
        const duration = 1400;
        let startTs = null;
        const tick = (ts) => {
          if (startTs === null) startTs = ts;
          const p = Math.min((ts - startTs) / duration, 1);
          const eased = 1 - Math.pow(1 - p, 3);
          el.textContent = Math.round(target * eased) + suffix;
          if (p < 1) requestAnimationFrame(tick);
        };
        requestAnimationFrame(tick);
      };
      if ('IntersectionObserver' in window) {
        const tio = new IntersectionObserver((entries) => {
          entries.forEach((entry) => {
            if (entry.isIntersecting) { runTicker(entry.target); tio.unobserve(entry.target); }
          });
        }, { threshold: 0.6 });
        tickers.forEach((el) => tio.observe(el));
      } else {
        tickers.forEach((el) => runTicker(el));
      }
    </script>
```

- [ ] **Step 2: Verify build** — Run: `npm run build` — Expected: sukses. Run `npm run dev` dan buka `/` untuk konfirmasi tak ada error console (sementara belum ada `[data-reveal]` di halaman, script no-op aman).

---

### Task 3: Header — sticky blur + mobile drawer

**Files:**
- Modify: `src/components/Header.astro`

**Interfaces:**
- Consumes: token Task 1.
- Produces: header responsif dengan drawer mobile (`#nav-toggle`).

- [ ] **Step 1: Ganti markup `<nav>`** untuk tambah tombol hamburger + bungkus link agar bisa jadi drawer di mobile. Logo tetap. Struktur baru:

```html
<header class="nav-bar" id="site-header">
  <div class="container nav-inner">
    <a href="/" class="logo">Kerjain<span class="logo-accent">Aja</span></a>
    <button class="nav-burger" id="nav-toggle" aria-label="Buka menu" aria-expanded="false" aria-controls="nav-menu">
      <span></span><span></span><span></span>
    </button>
    <nav class="nav-links" id="nav-menu" aria-label="Navigasi utama">
      {nav.map(item => (
        <a href={item.href} class={`nav-link${path === item.href ? ' active' : ''}`}>{item.label}</a>
      ))}
      <a href="/blog" class="btn btn-primary nav-cta">Jelajahi Panduan</a>
    </nav>
  </div>
</header>
```

- [ ] **Step 2: Ganti `<style>`** — tambah sticky-blur `.scrolled`, active link pill, dan drawer mobile. Ganti seluruh isi `<style>` Header dengan:

```css
  .nav-bar {
    background: color-mix(in srgb, var(--color-canvas) 80%, transparent);
    backdrop-filter: saturate(180%) blur(12px);
    -webkit-backdrop-filter: saturate(180%) blur(12px);
    border-bottom: 1px solid transparent;
    position: sticky; top: 0; z-index: 100;
    transition: border-color 0.3s, box-shadow 0.3s, background 0.3s;
  }
  .nav-bar.scrolled {
    border-bottom-color: var(--color-border-soft);
    box-shadow: 0 4px 20px rgba(32, 21, 21, 0.04);
  }
  .nav-inner { display: flex; justify-content: space-between; align-items: center; height: 64px; }
  .logo { font-family: var(--font-display); font-size: 24px; font-weight: 700; text-decoration: none; color: var(--color-ink); letter-spacing: -0.8px; }
  .logo:hover { color: var(--color-ink); text-decoration: none; }
  .logo-accent { color: var(--color-primary); }
  .nav-links { display: flex; align-items: center; gap: var(--space-xl); }
  .nav-link { font-size: 15px; font-weight: 500; color: var(--color-body); text-decoration: none; transition: color 0.2s; position: relative; }
  .nav-link:hover { color: var(--color-ink); text-decoration: none; }
  .nav-link.active { color: var(--color-ink); }
  .nav-link.active::after {
    content: ''; position: absolute; left: 0; right: 0; bottom: -22px; height: 2px;
    background: var(--color-primary); border-radius: 2px;
  }
  .nav-cta { font-size: 14.4px; font-weight: 700; line-height: 14.4px; padding: var(--space-sm) var(--space-lg); letter-spacing: 0.144px; }
  .nav-cta.active::after { display: none; }

  .nav-burger {
    display: none; flex-direction: column; gap: 5px; background: none; border: none;
    cursor: pointer; padding: 8px; border-radius: var(--radius-sm);
  }
  .nav-burger span { width: 22px; height: 2px; background: var(--color-ink); border-radius: 2px; transition: transform 0.25s, opacity 0.25s; }
  .nav-burger[aria-expanded="true"] span:nth-child(1) { transform: translateY(7px) rotate(45deg); }
  .nav-burger[aria-expanded="true"] span:nth-child(2) { opacity: 0; }
  .nav-burger[aria-expanded="true"] span:nth-child(3) { transform: translateY(-7px) rotate(-45deg); }

  @media (max-width: 768px) {
    .nav-burger { display: flex; }
    .nav-links {
      position: absolute; top: 64px; left: 0; right: 0;
      flex-direction: column; align-items: stretch; gap: var(--space-xs);
      background: var(--color-canvas); border-bottom: 1px solid var(--color-border-soft);
      padding: var(--space-lg); box-shadow: 0 12px 24px rgba(32,21,21,0.08);
      transform: translateY(-8px); opacity: 0; pointer-events: none; transition: transform 0.25s, opacity 0.25s;
    }
    .nav-links.open { transform: translateY(0); opacity: 1; pointer-events: auto; }
    .nav-link { padding: var(--space-md) var(--space-sm); font-size: 16px; }
    .nav-link.active::after { display: none; }
    .nav-link.active { color: var(--color-primary); }
    .nav-cta { margin-top: var(--space-sm); justify-content: center; }
  }
```

- [ ] **Step 3: Tambah `<script>`** di akhir Header.astro untuk toggle drawer + scrolled state:

```html
<script>
  const header = document.getElementById('site-header');
  const onScroll = () => header && header.classList.toggle('scrolled', window.scrollY > 8);
  onScroll();
  window.addEventListener('scroll', onScroll, { passive: true });

  const toggle = document.getElementById('nav-toggle');
  const menu = document.getElementById('nav-menu');
  toggle && toggle.addEventListener('click', () => {
    const open = menu.classList.toggle('open');
    toggle.setAttribute('aria-expanded', String(open));
    toggle.setAttribute('aria-label', open ? 'Tutup menu' : 'Buka menu');
  });
</script>
```

- [ ] **Step 4: Verify** — `npm run build` sukses; `npm run dev` cek desktop (nav inline, underline aktif, blur saat scroll) & mobile width <768px (burger buka/tutup drawer).

---

### Task 4: Footer — grid pattern + wordmark

**Files:**
- Modify: `src/components/Footer.astro`

- [ ] **Step 1: Tambah elemen dekoratif** — di dalam `<footer class="footer">` paling atas (sebelum `.footer-inner`'s container), tambahkan watermark + bungkus agar relatif:
  - Tambahkan `class="footer"` tetap; tambah `<div class="footer-wordmark" aria-hidden="true">KerjainAja</div>` tepat sebelum `<div class="footer-bottom">`.

- [ ] **Step 2: Tambah CSS** ke `<style>` Footer:

```css
  .footer { position: relative; overflow: hidden; }
  .footer::before {
    content: ''; position: absolute; inset: 0;
    background-image:
      linear-gradient(to right, rgba(255,254,251,0.04) 1px, transparent 1px),
      linear-gradient(to bottom, rgba(255,254,251,0.04) 1px, transparent 1px);
    background-size: 56px 56px;
    -webkit-mask-image: radial-gradient(ellipse 80% 70% at 50% 0%, #000, transparent 75%);
    mask-image: radial-gradient(ellipse 80% 70% at 50% 0%, #000, transparent 75%);
    pointer-events: none;
  }
  .footer-inner, .footer-bottom { position: relative; }
  .footer-wordmark {
    position: relative;
    text-align: center;
    font-family: var(--font-display);
    font-weight: 700;
    font-size: clamp(48px, 14vw, 180px);
    line-height: 0.9;
    letter-spacing: -4px;
    color: rgba(255, 254, 251, 0.04);
    user-select: none;
    margin: var(--space-2xl) 0 calc(var(--space-2xl) * -0.4);
  }
```

- [ ] **Step 3: Verify** — `npm run build` sukses; cek footer ada grid halus + wordmark samar.

---

### Task 5: Landing (`index.astro`) — treatment penuh MagicUI

**Files:**
- Modify: `src/pages/index.astro` (markup + `<style>`)

**Interfaces:** Consumes semua utilitas Task 1–2 (`pill`, `text-gradient`, `bg-grid`, `glow-card`, `marquee`, `[data-reveal]`, `[data-ticker]`).

- [ ] **Step 1: Hero center-aligned.** Ganti `<section class="hero-section">…</section>` jadi hero terpusat dengan bg grid, pill, gradient headline, dual CTA, lalu cluster kartu panduan (bento) di bawah. Markup:

```html
  <section class="hero-section">
    <div class="hero-bg bg-grid fade-mask" aria-hidden="true"></div>
    <div class="container hero-inner">
      <span class="pill" data-reveal><span class="pill-dot">✦</span> Diperbarui 2026 — panduan baru tiap minggu</span>
      <h1 data-reveal style="--reveal-delay:.05s">Panduan Lengkap <span class="text-gradient">Cari Kerja Online</span> di Indonesia</h1>
      <p class="hero-lead" data-reveal style="--reveal-delay:.1s">Tips CV ATS-friendly, trik lolos interview, review platform lowongan terpercaya — semuanya gratis, dari fresh graduate sampai profesional.</p>
      <div class="hero-actions" data-reveal style="--reveal-delay:.15s">
        <a href="/blog" class="btn btn-primary btn-lg">Jelajahi Semua Panduan →</a>
        <a href="/tentang" class="btn btn-tertiary btn-lg">Tentang Kami</a>
      </div>
      <div class="hero-cards" data-reveal style="--reveal-delay:.2s">
        <!-- 4 kartu pilihan (reuse hc data), grid responsif -->
        <a href="/blog/panduan-membuat-cv-2026" class="glow-card hcard"><span class="hcard-icon">📝</span><strong>Panduan CV 2026</strong><span>Bikin CV ATS-friendly</span></a>
        <a href="/blog/sukses-interview-kerja-online" class="glow-card hcard"><span class="hcard-icon">🎯</span><strong>Tips Interview</strong><span>Lolos wawancara online</span></a>
        <a href="/blog/perbandingan-platform-lowongan-kerja" class="glow-card hcard"><span class="hcard-icon">🔍</span><strong>Review Platform</strong><span>JobStreet vs LinkedIn</span></a>
        <a href="/blog/aplikasi-cari-kerja-gratis-2026" class="glow-card hcard"><span class="hcard-icon">📱</span><strong>Aplikasi Cari Kerja</strong><span>10 platform gratis</span></a>
      </div>
    </div>
  </section>
```

- [ ] **Step 2: Trust bar → marquee.** Ganti isi `.trust-logos` jadi struktur marquee dengan konten diduplikasi 2x agar loop mulus:

```html
  <section class="trust-bar">
    <p class="trust-label">Panduan untuk pencari kerja dari berbagai platform</p>
    <div class="marquee">
      <div class="marquee-track">
        <span class="trust-logo">JobStreet</span><span class="trust-logo">LinkedIn</span><span class="trust-logo">Glints</span><span class="trust-logo">Indeed</span><span class="trust-logo">Karirhub</span><span class="trust-logo">Kalibrr</span>
        <span class="trust-logo">JobStreet</span><span class="trust-logo">LinkedIn</span><span class="trust-logo">Glints</span><span class="trust-logo">Indeed</span><span class="trust-logo">Karirhub</span><span class="trust-logo">Kalibrr</span>
      </div>
    </div>
  </section>
```

- [ ] **Step 3: Features → bento grid.** Pertahankan 7 `feature-card` & isi teks, tapi ubah container jadi `.bento-grid`, jadikan tiap kartu `glow-card feature-card`, beri `data-reveal` + `--reveal-delay` bertingkat, dan kartu pertama (Panduan CV) jadi `feature-card--lg` (span 2 kolom + 2 baris). Tambahkan `data-reveal` pada `.section-head`.

- [ ] **Step 4: Stats → number ticker.** Untuk tiap `.stat-num`, ganti teks jadi span ticker:
  - `6` → `<span class="stat-num" data-ticker="6">0</span>`
  - `10+` → `<span class="stat-num" data-ticker="10" data-suffix="+">0</span>`
  - `5` → `<span class="stat-num" data-ticker="5">0</span>`
  - `100%` → `<span class="stat-num" data-ticker="100" data-suffix="%">0</span>`
  - Beri `data-reveal` pada `.stats-grid`. Bungkus tiap stat-card jadi `.glow-card` opsional (pakai border tipis).

- [ ] **Step 5: Dark CTA + grid glow.** Tambah `<div class="cta-bg bg-grid fade-mask" aria-hidden="true"></div>` di dalam `.section-dark`, beri posisi relatif & glow. Beri `.cta-block` atribut `data-reveal`. Ganti badge tombol jadi `.btn-lg`.

- [ ] **Step 6: FAQ accordion halus.** Tambah `data-reveal` pada `.faq-list`. Tambah chevron `summary::after` + animasi tinggi via `details[open]`. (CSS di Step 7.)

- [ ] **Step 7: Update `<style>`.** Tambahkan/ubah aturan berikut (hapus `.fade-in-up` lama karena diganti `[data-reveal]`; sisakan style lain yang masih dipakai):

```css
  /* Hero */
  .hero-section { position: relative; padding: var(--space-4xl) 0 var(--space-3xl); overflow: hidden; }
  .hero-bg { position: absolute; inset: 0; z-index: 0; }
  .hero-inner { position: relative; z-index: 1; text-align: center; max-width: 820px; }
  .hero-section .container { display: flex; flex-direction: column; align-items: center; }
  .hero-inner h1 { margin: var(--space-lg) 0; }
  .hero-lead { font-size: 20px; line-height: 30px; color: var(--color-body); margin: 0 auto var(--space-2xl); max-width: 600px; }
  .hero-actions { display: flex; gap: var(--space-md); flex-wrap: wrap; justify-content: center; margin-bottom: var(--space-3xl); }
  .btn-lg { font-size: 18px; padding: var(--space-md) var(--space-2xl); line-height: 27px; }
  .hero-cards { display: grid; grid-template-columns: repeat(4, 1fr); gap: var(--space-md); width: 100%; }
  .hcard { display: flex; flex-direction: column; gap: 4px; padding: var(--space-lg); text-decoration: none; color: inherit; text-align: left; }
  .hcard:hover { text-decoration: none; }
  .hcard-icon { font-size: 26px; }
  .hcard strong { font-size: 15px; color: var(--color-ink); }
  .hcard span:not(.hcard-icon) { font-size: 13px; color: var(--color-body-mid); }

  /* Trust marquee */
  .trust-bar { padding: var(--space-2xl) 0; background: var(--color-canvas); border-top: 1px solid var(--color-border-hair); border-bottom: 1px solid var(--color-border-hair); }
  .trust-label { text-align: center; font-size: 13px; color: var(--color-body-mid); letter-spacing: 0.5px; text-transform: uppercase; margin-bottom: var(--space-lg); }
  .trust-logo { font-size: 18px; font-weight: 600; color: var(--color-body-mid); opacity: 0.55; letter-spacing: -0.3px; }

  /* Sections */
  .section { padding: var(--space-4xl) 0; }
  .section-cream { background: var(--color-canvas-soft); }
  .section-light { background: var(--color-canvas); }
  .section-dark { position: relative; overflow: hidden; background: var(--color-ink); color: var(--color-on-primary); }
  .section-head { text-align: center; max-width: 620px; margin: 0 auto var(--space-3xl); }
  .section-head h2 { margin-bottom: var(--space-md); }
  .section-sub { font-size: 18px; line-height: 27px; color: var(--color-body); }

  /* Bento */
  .bento-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: var(--space-lg); grid-auto-rows: 1fr; }
  .feature-card { padding: var(--space-2xl); text-decoration: none; color: inherit; display: flex; flex-direction: column; }
  .feature-card:hover { text-decoration: none; }
  .feature-card--lg { grid-column: span 2; grid-row: span 2; background: var(--color-ink); color: var(--color-on-primary); }
  .feature-card--lg h3, .feature-card--lg p { color: var(--color-on-primary); }
  .feature-card--lg p { color: var(--color-body-mid); }
  .feature-icon { font-size: 32px; display: block; margin-bottom: var(--space-lg); }
  .feature-card h3 { font-size: 19px; font-weight: 600; line-height: 24px; margin-bottom: var(--space-sm); color: var(--color-ink); }
  .feature-card p { font-size: 15px; line-height: 22px; color: var(--color-body); margin-bottom: var(--space-lg); flex: 1; }
  .feature-cta { font-size: 14px; font-weight: 600; color: var(--color-primary); }

  /* Stats */
  .section-stats { background: var(--color-canvas); padding: var(--space-3xl) 0; }
  .stats-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: var(--space-xl); }
  .stat-card { text-align: center; padding: var(--space-xl); border-radius: var(--radius-lg); }
  .stat-num { display: block; font-size: 44px; font-weight: 700; color: var(--color-primary); line-height: 1; margin-bottom: var(--space-sm); letter-spacing: -1.5px; }
  .stat-label { font-size: 15px; color: var(--color-body-mid); }

  /* CTA */
  .cta-bg { position: absolute; inset: 0; z-index: 0; opacity: 0.5; }
  .section-dark .container { position: relative; z-index: 1; }
  .section-dark::after { content: ''; position: absolute; left: 50%; top: 50%; width: 480px; height: 240px; transform: translate(-50%,-50%); background: var(--color-primary); filter: blur(160px); opacity: 0.18; z-index: 0; pointer-events: none; }
  .cta-block { text-align: center; max-width: 620px; margin: 0 auto; }
  .cta-block h2 { margin-bottom: var(--space-md); }
  .cta-text { font-size: 18px; line-height: 27px; color: var(--color-body-mid); margin-bottom: var(--space-2xl); }

  /* FAQ */
  .faq-block { max-width: 760px; margin: 0 auto; }
  .faq-list { display: flex; flex-direction: column; gap: var(--space-md); }
  .faq-item { background: var(--color-canvas); padding: var(--space-lg) var(--space-xl); border-radius: var(--radius-lg); border: 1px solid var(--color-border-soft); transition: border-color 0.2s, box-shadow 0.2s; }
  .faq-item:hover { border-color: rgba(255,79,0,0.25); box-shadow: var(--shadow-card); }
  .faq-item summary { font-size: 16px; font-weight: 600; color: var(--color-ink); cursor: pointer; outline: none; list-style: none; display: flex; justify-content: space-between; align-items: center; gap: var(--space-md); }
  .faq-item summary::-webkit-details-marker { display: none; }
  .faq-item summary::after { content: '+'; font-size: 22px; font-weight: 400; color: var(--color-primary); transition: transform 0.25s; line-height: 1; }
  .faq-item[open] summary::after { transform: rotate(45deg); }
  .faq-item p { margin-top: var(--space-md); color: var(--color-body); font-size: 15px; line-height: 22px; }

  /* Responsive */
  @media (max-width: 1024px) {
    .bento-grid { grid-template-columns: repeat(2, 1fr); }
    .feature-card--lg { grid-column: span 2; grid-row: span 1; }
    .hero-cards { grid-template-columns: repeat(2, 1fr); }
  }
  @media (max-width: 768px) {
    .bento-grid { grid-template-columns: 1fr; }
    .feature-card--lg { grid-column: span 1; }
    .stats-grid { grid-template-columns: repeat(2, 1fr); gap: var(--space-lg); }
    .stat-num { font-size: 34px; }
    .hero-cards { grid-template-columns: 1fr 1fr; }
    .hero-lead { font-size: 18px; }
  }
```

- [ ] **Step 8: Verify** — `npm run build` sukses; `npm run dev` buka `/`: hero center + grid bg + pill + gradient, marquee jalan, bento tampil (kartu besar CV), stats count-up saat scroll, CTA glow, FAQ chevron animasi, reveal muncul. Cek mobile width.

---

### Task 6: Blog index (`blog/index.astro`) — card grid

**Files:**
- Modify: `src/pages/blog/index.astro`

- [ ] **Step 1: Header** — beri `data-reveal` pada elemen di `.blog-header` (eyebrow, h1, sub) atau pada container. Tambah `bg-grid fade-mask` opsional di band header.

- [ ] **Step 2: Card grid** — ubah `.blog-list` jadi grid responsif; tiap `.blog-card` pakai `.glow-card`, beri `data-reveal` + `--reveal-delay` bertingkat (`index * 0.06s`) via `style={`--reveal-delay:${i * 0.06}s`}` (gunakan `posts.map((post, i) =>`).

- [ ] **Step 3: Update `<style>`** — ganti `.blog-list`/`.blog-card` jadi:

```css
  .blog-list { display: grid; grid-template-columns: repeat(2, 1fr); gap: var(--space-xl); max-width: 980px; margin: 0 auto; }
  .blog-card { background: var(--color-canvas); display: flex; }
  .blog-card-link { display: flex; flex-direction: column; padding: var(--space-2xl); text-decoration: none; color: inherit; height: 100%; }
  .blog-card-link:hover { text-decoration: none; }
  .blog-meta { display: flex; align-items: center; gap: var(--space-md); margin-bottom: var(--space-md); font-size: 13px; color: var(--color-body-mid); flex-wrap: wrap; }
  .blog-tags { display: flex; gap: var(--space-xs); flex-wrap: wrap; }
  .blog-card h2 { font-size: 22px; font-weight: 600; line-height: 28px; letter-spacing: -0.6px; margin-bottom: var(--space-sm); transition: color 0.2s; }
  .blog-card:hover h2 { color: var(--color-primary); }
  .blog-excerpt { font-size: 15px; line-height: 23px; color: var(--color-body); margin-bottom: var(--space-lg); flex: 1; }
  @media (max-width: 768px) { .blog-list { grid-template-columns: 1fr; } }
```

- [ ] **Step 4: Verify** — `npm run build` sukses; `/blog` menampilkan grid kartu glow + reveal stagger.

---

### Task 7: Blog post (`blog/[...slug].astro`) + tabel

**Files:**
- Modify: `src/pages/blog/[...slug].astro`
- Modify: `src/styles/tables.css`

- [ ] **Step 1: Header artikel** — tambah `<span class="eyebrow">Artikel</span>` di atas `<h1>` dalam `.post-header`; tag pakai `.tag`; beri `data-reveal` pada `.post-header`. Tambah `data-reveal` pada `.post-content` (satu blok, delay kecil).

- [ ] **Step 2: Selaraskan prose** — di `<style>`: blockquote pakai `border-radius: var(--radius-md)` + background lembut; `code` radius `6px`; pertahankan ukuran. Tambah `.eyebrow { color: var(--color-primary); }` lokal opsional (atau biarkan default). Pastikan `h1` post tetap besar.

- [ ] **Step 3: tables.css** — perhalus: `.table-wrap` `border-radius: var(--radius-lg)`, border `var(--color-border-soft)`; `th` border-bottom `2px solid var(--color-primary)` (ganti dari ink) untuk aksen; hover row tetap.

- [ ] **Step 4: Verify** — `npm run build` sukses; buka satu artikel (mis. `/blog/perbandingan-platform-lowongan-kerja` yang punya tabel) — header reveal, prose rapi, tabel rounded + scroll mobile.

---

### Task 8: Tentang (`tentang.astro`)

**Files:**
- Modify: `src/pages/tentang.astro`

- [ ] **Step 1: Hero & reveal** — beri `data-reveal` pada eyebrow/h1/lead. Band header boleh tambah `bg-grid fade-mask`.

- [ ] **Step 2: Offerings → grid kartu glow** — ubah `.offerings` jadi grid 2 kolom; tiap `.offering` jadi `.glow-card` dengan ikon di atas + teks; `data-reveal` stagger. Visi card pakai `.glow-card` (atau dark card aksen).

- [ ] **Step 3: Update `<style>`**:

```css
  .offerings { display: grid; grid-template-columns: repeat(2, 1fr); gap: var(--space-md); margin: var(--space-xl) 0; }
  .offering { padding: var(--space-xl); background: var(--color-canvas); border: 1px solid var(--color-border-soft); border-radius: var(--radius-lg); font-size: 15px; line-height: 24px; }
  .about-vision { margin: var(--space-3xl) 0; padding: var(--space-2xl); border-radius: var(--radius-xl); background: var(--color-ink); color: var(--color-on-primary); }
  .about-vision h4 { color: var(--color-on-primary); margin-bottom: var(--space-md); }
  .about-vision .visi-text { color: var(--color-body-mid); }
  @media (max-width: 768px) { .offerings { grid-template-columns: 1fr; } }
```

- [ ] **Step 4: Verify** — `npm run build` sukses; `/tentang` konsisten: kartu glow, visi card aksen, reveal.

---

### Task 9: Kontak (`kontak.astro`)

**Files:**
- Modify: `src/pages/kontak.astro`

- [ ] **Step 1: Hero & reveal** — `data-reveal` pada eyebrow/h1/sub.
- [ ] **Step 2: Info cards & form** — info-card jadi `.glow-card`; form `.card` jadi radius `--radius-xl`, border-soft; beri `data-reveal` pada kolom info & form. JANGAN ubah atribut `<form>`.
- [ ] **Step 3: Update `<style>`**:

```css
  .info-card { padding: var(--space-xl); background: var(--color-canvas); border: 1px solid var(--color-border-soft); border-radius: var(--radius-lg); }
  .contact-form { border-radius: var(--radius-xl); border: 1px solid var(--color-border-soft); background: var(--color-canvas); padding: var(--space-2xl); }
```

- [ ] **Step 4: Verify** — `npm run build` sukses; `/kontak` kartu glow + form rapi; submit form markup tak berubah.

---

### Task 10: Verifikasi menyeluruh

- [ ] **Step 1:** `npm run build` — Expected: sukses tanpa error/warning blocking.
- [ ] **Step 2:** `npm run dev`, cek tiap halaman desktop + mobile width: `/`, `/blog`, `/blog/perbandingan-platform-lowongan-kerja`, `/tentang`, `/kontak`.
- [ ] **Step 3:** Konfirmasi: header sticky-blur + drawer mobile, reveal blur-fade muncul sekali, marquee jalan & pause on hover, ticker count-up, FAQ chevron, glow card hover, footer wordmark, tabel rounded+scroll. Tidak ada layout shift parah, focus state terlihat, console bersih.
- [ ] **Step 4:** Konfirmasi konten/teks/link & JSON-LD tidak berubah dari sebelumnya.

## Self-Review

- **Spec coverage:** global.css (T1), Layout aktivator (T2), Header (T3), Footer (T4), index full (T5), blog index (T6), blog post + tables (T7), tentang (T8), kontak (T9), verifikasi (T10). Semua unit di spec tercakup. ✓
- **Non-goals dijaga:** tanpa dependency baru, form action utuh, schema/konten tak diubah, reduced-motion, progressive enhancement. ✓
- **Type/nama konsisten:** `[data-reveal]`/`.is-visible`, `[data-ticker]`/`data-suffix`, `.glow-card`, `.marquee`/`.marquee-track`, `.pill`/`.pill-dot`, `.bg-grid`/`.fade-mask` dipakai konsisten lintas task. ✓
