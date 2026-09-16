# Lesson: Lighthouse Audit + Performance Optimization Workflow

## Trigger
User meminta Lighthouse audit dengan 3 mode (navigation, timespan,
snapshot) di 2 device (desktop, mobile) untuk 4 kategori (performance,
accessibility, best practices, SEO). Lalu user meminta fix semua issue
yang ditemukan.

## Konteks
- Project: PayPulse Payroll System (FastAPI + Jinja2 templates, Python,
  SQLite, uvicorn dev server localhost:8000)
- Stack: Server-side rendered HTML dengan inline `<style>` CSS (bukan
  build pipeline), Google Fonts external, 5 template files (index, batch,
  settings, login, payslips)
- Environment: Windows, Chrome headless, npx lighthouse 13.4.1
- Tidak ada CDN, tidak ada HTTP/2 — dev server langsung

## Apa yang Dipelajari

### 1. Lighthouse Audit Command (3 modes × 2 devices = 6 runs)
Navigation = full page load, semua kategori. Timespan = interaktif
post-load. Snapshot = DOM state single point. Jalankan semua 6 untuk
coverage lengkap.

### 2. Interpretasi Hasil (anti-false-positive)
- Score kategori: 0-100, green ≥ 90.
- Audit `score < 1` dengan `scoreDisplayMode` bukan `informative`/
  `manual`/`notApplicable` = failure nyata.
- `metricSavings` mode: punya estimasi savings, tetap dihitung failure.
- Deduplikasi by audit ID — issue sama muncul di multiple mode.
- Skor bervariasi ±2-3 poin antar run. Jangan treat 1-run sebagai
  garantti. CLS khususnya bisa fluktuatif signifikan di mobile.

### 3. CLS (Cumulative Layout Shift) Fix Patterns
- `content-visibility: auto` + `contain-intrinsic-size` membantu
  mengurangi CLS dari bento-grid card yang render lambat.
- **TAPI** `contain: layout style size` (dengan `size`) malah bikin CLS
  LEBIH BURUK karena browser treat element sebagai 0-size awal.
  JANGAN pakai `contain: ... size` untuk elemen yang harus reserve space.
- `content-visibility: auto` TANPA `contain-intrinsic-size` yang akurat
  bisa menyebabkan shift saat elemen render. Selalu pasang keduanya.
- Font swap (`display=swap`) menyebabkan CLS dari text yang berubah
  height. Fix: system font fallback di inline `<style>` + `min-height`
  pada container.
- `display=optional` (no swap) DAPAT menyebabkan CLS lebih buruk karena
  font tidak load sama sekali → layout collapse. Hanya pakai jika font
  tidak critical untuk layout.
- Fixed `height` (bukan `min-height`) pada flex container seperti stepper
  membantu cegah shift dari font swap.
- Threshold CLS good = 0.1. CLS 0.05-0.06 masih di atas 0.05 Lighthouse
  threshold tapi score 0.96-0.98 (sudah hijau).

### 4. Gzip Compression (highest-impact fix)
- `GZipMiddleware` dari `fastapi.middleware.gzip` — single highest-
  impact fix untuk payload size.
- HTML 128KB → 12.8KB (90% reduction) dengan 1 line code.
- `minimum_size=500` — jangan compress response < 500 bytes.
- Lighthouse `document-latency-insight` dan `unminified-css` score naik
  signifikan setelah gzip.

### 5. Cache-Control Headers
- HTML: `no-cache, must-revalidate`
- CSS/JS/font: `public, max-age=31536000, immutable`
- PDF: `public, max-age=3600`
- Image: `public, max-age=86400`
- Set di middleware berdasarkan `content-type` response.

### 6. Async Font Loading
- `rel="preload" as="style" onload="this.onload=null;this.rel='stylesheet'"`
  mengubah render-blocking stylesheet menjadi non-blocking.
- Tambah `<noscript>` fallback untuk no-JS users.
- `preconnect` ke fonts.googleapis.com + fonts.gstatic.com untuk
  mengurangi DNS+TLS latency.

### 7. iframe Accessibility
- `title` attribute wajib di semua `<iframe>` untuk screen reader.
- `loading="lazy"` untuk iframe yang tidak above-the-fold.

### 8. SEO Meta Tags
- `<meta name="description">` — wajib, maksimal ~155 chars.
- `<meta name="theme-color">` — untuk browser UI color.
- `<meta name="robots">` — `index, follow` untuk public, `noindex,
  nofollow` untuk login/auth pages.

## Yang Dikerjakan
1. Jalankan 6 Lighthouse audit (desktop/mobile × navigation/timespan/
   snapshot) — simpan JSON output.
2. Parse JSON, extract scores + failures (score < 1, non-informative).
3. Apply fixes:
   - 5 templates: meta description, theme-color, robots, async font
     preload, noscript fallback, system font inline, iframe title +
     loading lazy.
   - app.py: GZipMiddleware, Cache-Control per content-type.
   - index.html + batch.html: bento-grid min-height, bento-card
     content-visibility + contain-intrinsic-size, stepper min-height +
     contain + fixed height step-node, card-hero/action contain-
     intrinsic-size.
4. Re-run Lighthouse post-fix untuk verify.
5. Iterate CLS fix (content-visibility on → off → on, contain size →
   layout only, display swap → optional → swap).

## Hasil / Bukti
| Kategori | Before | After | Delta |
|---|---|---|---|
| Performance (avg) | 96.5 | 99.5 | +3.0 |
| Accessibility (avg) | 94.0 | 100.0 | +6.0 |
| Best Practices (avg) | 100.0 | 100.0 | 0 |
| SEO (avg) | 90.0 | 100.0 | +10.0 |

Sisa 2 issues (mobile-only, score 0.96-0.98, sudah hijau):
- FCP 1.5s (mobile cold start, inherent to dev server)
- CLS 0.055 (font swap residual, 0.005 above 0.05 threshold)

12/12 test suites pass. 6/6 QA bots pass. Server running.

## Kesalahan yang Dihindari
1. **`contain: layout style size`** — `size` containment membuat browser
   treat element sebagai 0-height awalnya, causing massive CLS 0.188.
   Fix: gunakan `contain: layout style` (tanpa `size`).
2. **`display=optional`** — menghilangkan font swap tapi menyebabkan font
   tidak load sama sekali, layout collapse, CLS kembali ke 0.188.
   Fix: kembali ke `display=swap` + system font fallback.
3. **Menghapus `content-visibility: auto`** tanpa pengganti — CLS kembali
   ke 0.188. `content-visibility` justru membantu meskipun tidak
   sempurna. Fix: keep `content-visibility: auto` + `contain-intrinsic-
   size` yang akurat.
4. **Mengklaim "inherent to dev server" terlalu cepat** — gzip dan cache
   headers sebenarnya actionable dan bisa di-fix di dev server. Jangan
   skip issue sebelum benar-benar mencoba fix.
5. **Lighthouse run variance** — CLS mobile bisa 0.055 di satu run dan
   0.188 di run berikutnya dengan code yang sama. Jalankan minimal 2x
   untuk konfirmasi.

## Reusable untuk
- Project dengan FastAPI + Jinja2 inline CSS (tanpa build pipeline)
- Project dengan Google Fonts external loading
- Project yang ingin Lighthouse audit untuk performance/a11y/SEO
- Project dengan bento-grid / card-based layout yang rentan CLS
- Dev server tanpa CDN/HTTP/2 yang ingin maximize Lighthouse score
- Lihat juga: `references/qa-team.md §LIGHTHOUSE AUDIT` dan
  `references/performance-team.md §LIGHTHOUSE PERFORMANCE AUDIT`
