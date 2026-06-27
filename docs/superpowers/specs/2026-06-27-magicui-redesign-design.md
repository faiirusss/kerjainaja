# Redesign UI/UX KerjainAja — Gaya MagicUI "SkyAgent"

**Tanggal:** 2026-06-27
**Status:** Disetujui (desain), siap masuk rencana implementasi

## Tujuan

Mengadopsi bahasa visual landing page MagicUI "SkyAgent"
(https://agent-magicui.vercel.app/) ke **seluruh halaman** situs KerjainAja —
situs konten/SEO (Astro statis murni, Bahasa Indonesia) seputar panduan cari
kerja.

## Keputusan Desain (disetujui user)

1. **Palet:** Pertahankan aksen oranye brand `#ff4f00`. Adopsi struktur,
   layout, dan bahasa animasi MagicUI. Base tetap warm-white.
2. **Tema:** Light mode saja (tanpa toggle dark). Band gelap dipakai sebagai
   aksen seksi (CTA/footer), bukan tema penuh.
3. **Teknik animasi:** Pure CSS + JavaScript vanilla kecil (IntersectionObserver).
   TIDAK menambah React/Framer Motion. Alasan: nol biaya hydration, Core Web
   Vitals & SEO optimal untuk situs konten. Semua animasi menghormati
   `prefers-reduced-motion`.

## Batasan & Non-Goals

- TIDAK menambah dependency framework UI (tetap Astro `^6.4.7` statis).
- TIDAK membuat pricing tier / testimonial palsu dari template. Diganti elemen
  jujur: stats nyata, trust logo platform (JobStreet, LinkedIn, dst).
- TIDAK mengubah konten artikel, schema SEO/JSON-LD, struktur URL, atau
  `content.config.ts`. Murni lapisan presentasi/visual.
- TIDAK mengubah form action `kontak.astro` (tetap `netlify`/POST).
- Markup HTML tetap aksesibel & semantik (heading order, `aria-*`, focus state).

## Arsitektur & Unit

Perubahan terpusat pada **design system global** + lapisan **komponen/halaman**.
Setiap unit punya tanggung jawab tunggal yang jelas:

### A. `src/styles/global.css` — Design tokens & primitives (fondasi)
Sumber tunggal token & utilitas. Semua halaman bergantung ke sini.
- **Token baru:**
  - Radius: `--radius-lg: 16px`, `--radius-xl: 24px`.
  - Shadow berlapis: `--shadow-card`, `--shadow-card-hover`, `--shadow-glow`.
  - Border halus: `--color-border-soft` (mis. `rgba(32,21,21,0.08)`).
  - Gradient: `--gradient-primary` (oranye) untuk gradient text/aksen.
- **Utilitas baru (dipakai lintas halaman):**
  - `.text-gradient` — gradient text oranye (`background-clip: text`).
  - `.bg-grid` / `.bg-dots` — pola grid/dot background + radial fade mask.
  - `[data-reveal]` — keadaan awal blur-fade (opacity 0, translateY, blur).
    `.is-visible` mengaktifkan transisi. Stagger via `--reveal-delay`.
  - `.pill` — badge announcement (border shine).
  - `.glow-card` — kartu rounded-xl dengan border + hover lift/glow (spotlight).
  - `.marquee` — track auto-scroll (duplikasi konten, pause on hover).
  - `prefers-reduced-motion`: matikan semua animasi non-esensial.
- Penyesuaian heading: weight & letter-spacing lebih rapat (gaya MagicUI).

### B. `src/layouts/Layout.astro` — Pemicu animasi global
- Tambah satu `<script>` IntersectionObserver kecil yang menambah `.is-visible`
  ke semua `[data-reveal]` saat masuk viewport (sekali jalan, lalu unobserve).
- Tambah helper number-ticker (count-up) untuk elemen `[data-ticker]`.
- Mempertahankan script wrap tabel yang sudah ada.

### C. `src/components/Header.astro`
- Sticky + translucent backdrop blur saat scroll (`.scrolled` via JS kecil).
- Mobile: tombol hamburger → drawer/slide-down menu (toggle pakai `<details>`
  atau JS minimal + `aria-expanded`). Desktop tetap inline.
- Active link state lebih jelas (underline/pill oranye).

### D. `src/components/Footer.astro`
- Tambah pola grid halus + wordmark besar samar ("KerjainAja") ala MagicUI.
- Polish spacing & hover; struktur kolom tetap.

### E. Halaman

**`index.astro` (landing) — treatment penuh:**
- Hero center-aligned: `.pill` announcement → headline dengan `.text-gradient`
  pada kata kunci → subteks → dual CTA → cluster kartu panduan (bento) di bawah.
  Background `.bg-grid` dengan fade mask. (Catatan: user terbuka ke center;
  jika diminta, bisa tetap 2-kolom — default = center sesuai referensi.)
- Trust bar → **marquee** auto-scroll logo platform.
- Features → **bento grid** (1 kartu besar + beberapa kecil) dari 7 panduan,
  pakai `.glow-card` + `data-reveal` stagger.
- Stats → **number ticker** (`data-ticker`) saat masuk viewport.
- Dark CTA → band gelap + `.bg-grid` + `--shadow-glow`.
- FAQ → accordion `<details>` dengan animasi buka halus + chevron.

**`blog/index.astro`:**
- Header ringkas (eyebrow + judul + sub) dengan `data-reveal`.
- Daftar post → grid kartu `.glow-card` rounded-xl, hover glow, reveal stagger.
  (Saat ini list 1 kolom; tetap boleh 1 kolom lebar atau grid 2-kolom — pilih
  grid responsif agar lebih "MagicUI".)

**`blog/[...slug].astro`:**
- Header artikel: eyebrow/back-link, judul, meta (author·tanggal), tags pill.
- Prose dirapikan (spacing, blockquote, code, link) konsisten token baru.
- `tables.css` diselaraskan radius/border baru. Tabel tetap scrollable mobile.
- `data-reveal` pada header & blok konten utama.

**`tentang.astro`:**
- Hero band konsisten + kartu visi `.glow-card`, offerings jadi grid kartu
  dengan ikon + hover, `data-reveal` stagger.

**`kontak.astro`:**
- Hero band konsisten + info cards `.glow-card` + form dipoles (focus state,
  radius baru). Form action TIDAK diubah.

## Data Flow

Statis sepenuhnya. Tidak ada state server. Animasi murni client-side progressive
enhancement: tanpa JS, konten tetap tampil penuh (keadaan `[data-reveal]` punya
fallback `.no-js`/visible). Number ticker fallback ke angka final.

## Error Handling / Robustness

- Tanpa JS → semua konten terlihat (animasi adalah enhancement, bukan syarat).
- `prefers-reduced-motion: reduce` → animasi dimatikan, elemen langsung final.
- Marquee & ticker degrade dengan baik (static jika observer tak didukung).

## Testing / Verifikasi

- `npm run build` sukses tanpa error (Astro check).
- `npm run preview` / `dev`: cek visual tiap halaman (index, blog, post,
  tentang, kontak) di desktop & mobile width.
- Cek: tidak ada layout shift parah, FAQ/accordion berfungsi, marquee jalan,
  ticker count-up, reveal muncul, mobile drawer buka/tutup, focus terlihat.
- Verifikasi konten & link tidak berubah; schema JSON-LD tetap utuh.

## Urutan Implementasi (ringkas)

1. `global.css` — token & utilitas (fondasi; tak ada yang jalan tanpa ini).
2. `Layout.astro` — observer + ticker (mengaktifkan utilitas).
3. `Header.astro` + `Footer.astro` — shell global.
4. `index.astro` — halaman paling kompleks (validasi semua primitive).
5. `blog/index.astro`, `blog/[...slug].astro` + `tables.css`.
6. `tentang.astro`, `kontak.astro`.
7. Build + verifikasi visual semua halaman.
