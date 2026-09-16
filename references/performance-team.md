# Performance Team

Ikuti Core Discipline (SKILL.md §3).

## ROLE
Optimasi performance HARUS evidence-based — jangan optimasi prematur.
Jangan tambah Redis/queue/caching/microservice/infra kompleks kecuali ada
kebutuhan nyata yang terukur.

## FRONTEND CHECKS
Re-render tak perlu, ukuran bundle, ukuran gambar/asset, duplikasi API
call, ukuran DOM, animasi berlebihan, operasi blocking.

## BACKEND CHECKS
Performance query, index, ukuran payload, request tak perlu, peluang
caching, concurrency.

## CONCURRENCY DETAIL (extend dari BACKEND CHECKS — referensi untuk qa-team.md §LOAD/STRESS TESTING)
- Cek apakah operasi yang sensitif terhadap concurrency (checkout,
  update stok, submit form massal) sudah pakai locking/transaction yang
  tepat (row-level lock, optimistic locking dengan version field, atau
  mekanisme sesuai database yang dipakai).
- Cek connection pool & worker/thread limit tidak exhaust saat banyak
  request bersamaan (lihat juga `§SERVER & DATABASE METRICS` di bawah).
- Untuk load testing: kolaborasi dengan QA (`qa-team.md §LOAD/STRESS
  TESTING`) — Performance team fokus analisis root cause kalau ada
  bottleneck yang ketemu saat load test, QA fokus menjalankan test-nya.

## CORE WEB VITALS (extend dari checks di atas dengan target terukur)
Inspeksi baseline existing, device/network yang didukung, route penting,
dan monitoring yang ada sebelum optimasi.

Target "good" di persentil ke-75 real-user measurement (mobile & desktop
dinilai terpisah):
- **LCP** (Largest Contentful Paint) ≤ 2.5 detik
- **CLS** (Cumulative Layout Shift) ≤ 0.1
- **INP** (Interaction to Next Paint) ≤ 200ms

Ini baseline, bukan jaminan tiap visit. Pertahankan budget project yang
lebih ketat kalau sudah ada. Pakai data field (RUM/CrUX) kalau tersedia —
lab tool/Lighthouse berguna untuk diagnosis tapi TIDAK membuktikan
persentil field lulus. Total Blocking Time ≠ INP.

**Implementasi**: perbaiki LCP dengan kurangi latency server & render
blocking, size/compress gambar kritis dengan benar, jangan lazy-load
gambar LCP above-the-fold. Cegah CLS dengan reserve space untuk media/
embed, kontrol perubahan font/layout. Perbaiki INP dengan kurangi
long-task main-thread, handler event mahal, rendering tak perlu.

**Budget**: definisikan angka budget per route/route-class (JS/CSS
terkirim, asset transferred, gambar/font kritis, request count, latency
server) berdasar baseline & konstrain produk aktual — jangan mengarang
target universal. Bandingkan before/after dengan kondisi setara. Jangan
longgarkan budget cuma supaya check lulus.

## CACHING STRATEGY
Inspeksi semua cache relevan: browser HTTP cache, service worker,
frontend query/state cache, CDN/reverse proxy, application cache,
database/query cache. Pakai caching HANYA kalau dijustifikasi behavior
terukur atau requirement konkret — pakai infra existing, jangan
perkenalkan Redis/Memcached hanya karena "ada di daftar".

- **Klasifikasi data**: asset publik non-personalized bisa shared-cache
  dengan freshness/invalidation jelas. HTML & data API mutable butuh
  freshness policy eksplisit — jangan kasih policy immutable secara tidak
  sengaja. Jangan taruh response authenticated/personalized/sensitive di
  shared cache kecuali desainnya sudah eksplisit mengisolasi identity &
  authorization boundary. Pakai `Cache-Control: no-store` untuk response
  berisi secret (token/reset/payment).
- **Browser & CDN**: definisikan `Cache-Control` & validator (ETag/
  Last-Modified) yang benar. Set cache key & `Vary` sesuai perbedaan
  representasi nyata — `Vary: Cookie`/`Vary: Authorization` saja BUKAN
  jaminan personalized caching aman. Cegah cache poisoning dengan
  validasi input routing/host.
- **Server-side cache**: definisikan purpose, owner, key format,
  namespace environment/tenant, TTL, size limit, eviction policy, failure
  behavior untuk tiap cache. Pilih strategi invalidation eksplisit
  (write-through/cache-aside/event-driven/versioned key) — invalidate
  setelah perubahan data berhasil.
- **Privacy & lifecycle**: invalidate cache relevan setelah logout, ganti
  tenant/akun, pencabutan permission, penghapusan privacy. `no-store` saja
  tidak mencegah JavaScript custom menyimpan data — cek behavior service
  worker/offline terpisah.

## LIGHTHOUSE PERFORMANCE AUDIT (collaboration dengan QA)
Lighthouse lab data adalah alat diagnosis utama Performance team —
melengkapi (bukan menggantikan) target Core Web Vitals di atas. QA
menjalankan audit; Performance team menganalisis root cause & menerapkan
fix. Lihat `qa-team.md §LIGHTHOUSE AUDIT` untuk command template,
prasyarat, dan interpretasi score.

### Tanggung Jawab Performance Team
1. **Baseline**: sebelum optimasi, jalankan Lighthouse navigation (desktop
   + mobile) untuk dapatkan skor baseline per route penting.
2. **Root cause analysis**: untuk setiap audit failure (score < 1):
   - Buka `details.items` di JSON output untuk identifikasi elemen/route
     spesifik yang menyebabkan issue.
   - Klasifikasikan: **actionable** (bisa di-fix: render-blocking,
     unminified CSS, no cache, CLS dari dynamic content) vs **inherent**
     (dev server latency, no CDN, no HTTP/2 — hilang di production).
3. **Budget enforcement**: bandingkan skor & metric (FCP, LCP, CLS, SI,
   TTI) dengan budget project. Jika melanggar budget, prioritaskan fix
   sebelum ship.
4. **Post-fix verification**: setelah optimasi, re-run Lighthouse.
   Bandingkan before→after. Jangan klaim fixed tanpa re-run konfirmasi.

### Metric → Fix Mapping
| Metric | Root Cause Umum | Fix |
|---|---|---|
| FCP tinggi | Render-blocking CSS/JS, slow server response | Inline critical CSS, defer non-critical, preload font, gzip |
| LCP tinggi | Large hero image/font, slow TTFB | Preload LCP element, optimize image, reduce TTFB |
| CLS > 0.1 | Font swap, dynamic content, no reserved space | `min-height`, `content-visibility`, `aspect-ratio`, `font-display: swap` + system fallback |
| SI tinggi | Slow visual population | Inline above-fold CSS, reduce blocking requests |
| TTI tinggi | Long JS tasks, heavy main-thread | Code-split, defer non-critical JS, reduce bundle |
| Minify CSS | Unminified inline/external CSS | Minify atau pindah ke build pipeline |
| Document latency | No gzip, slow server | Gzip/Brotli compression, cache headers |

### Gzip & Compression
Cek apakah server mengirim `Content-Encoding: gzip` (atau brotli).
Tanpa compression, HTML/CSS/JS payload 3–10x lebih besar dari perlu.
Tambahkan gzip middleware di application server jika belum ada —
ini single highest-impact fix untuk payload size.

### Cache Headers Audit
Lighthouse men-flag response tanpa `Cache-Control`. Pastikan:
- HTML: `no-cache, must-revalidate` (selalu fresh, tapi revalidation cepat)
- Static assets (CSS/JS/font): `public, max-age=31536000, immutable`
- Images: `public, max-age=86400`
- PDF: `public, max-age=3600` (atau no-cache jika dinamis)
Jangan kasih `immutable` ke response yang bisa berubah (HTML, API mutable).

### Lighthouse ≠ Field Data
Lighthouse = lab simulation (satu device, throttling simulasi, isolated).
Real user data (CrUX, RUM) bisa berbeda. Pakai Lighthouse untuk
diagnosis struktur, bukan untuk klaim "persentil field lulus". Jika field
data tersedia (Page Speed Insights API, CrUX dashboard), bandingkan
dengan lab data untuk validasi.

## OUTPUT KE TEAM LEAD
Metrik yang diukur (sumber & periode data), perubahan yang dilakukan,
hasil before/after, risiko konsistensi/privacy yang tersisa. Sertakan
tabel Lighthouse before→after, klasifikasi actionable vs inherent, dan
rekomendasi fix untuk sisa issue.

## SERVER & DATABASE METRICS (extend dari BACKEND CHECKS)
Selain "performance query" secara umum, cek eksplisit:
- **TTFB** (Time to First Byte): target < 600ms di persentil ke-75 untuk
  route dinamis (sesuaikan dengan baseline & constraint infra aktual —
  jangan mengarang angka universal kalau baseline project berbeda).
- **N+1 query detection**: inspeksi query log/APM untuk pola query
  berulang dalam loop (misal fetch relasi per-item alih-alih eager
  load/batch). Fix dengan eager loading, batching, atau DataLoader
  pattern sesuai stack.
- **Connection pooling**: pastikan pool size sesuai concurrency yang
  diharapkan, tidak exhaust koneksi database saat traffic naik. Cek
  timeout & retry behavior saat pool penuh.

## MONITORING & ALERTING PASCA-DEPLOY
Setelah optimasi di-ship ke production:
- Pastikan ada alerting kalau metric (LCP/INP/CLS, TTFB, error rate)
  regress dari baseline setelah deploy — bukan hanya dicek manual sekali.
- Definisikan threshold alert berdasar budget yang sudah ditetapkan
  (lihat §Budget), bukan angka arbitrary baru.
- Kalau tidak ada infra alerting existing, laporkan sebagai gap ke team
  lead — jangan diam-diam skip.

## THIRD-PARTY SCRIPT IMPACT
Audit dampak script pihak ketiga (analytics, ads, chat widget, font
external, tracking pixel) terhadap main-thread:
- Ukur kontribusi terhadap TBT/INP lewat Lighthouse third-party summary
  atau Performance panel DevTools.
- Klasifikasi: essential (tidak bisa dihapus) vs optional (bisa
  defer/lazy-load/hapus).
- Fix: `async`/`defer` loading, lazy-load setelah interaction, self-host
  kalau memungkinkan & diizinkan, atau facade pattern (misal facade untuk
  embed video/chat sebelum user interaksi).

## MODERN IMAGE FORMAT & RESPONSIVE IMAGES (extend dari implementasi LCP)
Selain "size/compress gambar dengan benar":
- Pakai format modern (WebP/AVIF) dengan fallback untuk browser lama,
  kalau browser support project mengizinkan.
- Pakai `srcset`/`sizes` untuk serve resolusi sesuai viewport — jangan
  kirim gambar desktop-size ke mobile.
- Cek `loading="lazy"` HANYA untuk gambar below-the-fold (ingat: jangan
  lazy-load gambar LCP above-the-fold, sudah disebutkan sebelumnya).

## BUNDLE ANALYSIS TOOLING
Sebelum klaim bundle size sudah optimal, pakai tool analisis (misal
bundle analyzer sesuai bundler project — webpack-bundle-analyzer,
rollup-plugin-visualizer, atau built-in tool framework) untuk:
- Identifikasi dependency terbesar yang berkontribusi ke bundle size.
- Cek duplicate dependency (versi berbeda dari package sama).
- Verifikasi tree-shaking & code-splitting bekerja sesuai harapan
  (dynamic import benar-benar split, bukan ke-bundle jadi satu chunk).

## ROLLBACK PLAN
Untuk optimasi berisiko (caching baru, perubahan infra, index database
baru):
- Definisikan cara rollback sebelum ship (revert config, disable feature
  flag, hapus cache layer) — bukan improvisasi saat production bermasalah.
- Kalau perubahan melibatkan migration/schema, pastikan migration
  reversible atau ada backward-compatible path.
- Monitor metric pasca-deploy untuk window waktu tertentu sebelum
  dianggap stable (lihat §Monitoring & Alerting Pasca-Deploy).
