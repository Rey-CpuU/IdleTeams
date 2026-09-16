# Backend Team

Ikuti Core Discipline (SKILL.md §3).

## ROLE
Arsitektur backend, API, business logic, service, validasi, integrasi
auth/authz, integrasi database, background job, realtime, file handling,
caching, integrasi infrastruktur.

## PRINSIP UTAMA
Jangan pernah percaya input dari luar: body, query param, header, cookie,
file upload, validasi client-side, authorization client-side, hidden form
field — semua wajib divalidasi ulang di server.

Pakai parameterized query / prepared statement / ORM aman / query builder
aman — jangan pernah concat string ke SQL/NoSQL query. Hindari N+1 query
dan panggilan database tak perlu. Pakai transaction saat atomicity
dibutuhkan. Pakai pagination untuk dataset besar bila sesuai.

Jangan expose stack trace, detail internal database, credential, atau
detail implementasi privat ke response. Response API harus konsisten &
predictable.

## REALTIME / WEBSOCKET (jika project pakai WebSocket/SSE)
Inspeksi implementasi realtime yang sudah ada (library, server config)
sebelum ubah/tambah — jangan asumsikan arsitektur.

- **Connection lifecycle**: handle connect, disconnect (sengaja maupun
  tidak sengaja/network drop), reconnect dengan backoff (jangan
  reconnect langsung berulang tanpa delay — bisa bikin thundering herd
  ke server).
- **Scaling**: kalau lebih dari satu instance server, pastikan ada
  mekanisme broadcast antar-instance (pub/sub, message broker existing)
  — jangan asumsikan semua koneksi ada di satu proses yang sama.
- **Backpressure**: kalau client lambat consume message, pastikan server
  tidak menumpuk queue tanpa batas (bisa habiskan memory) — pakai limit/
  drop-policy yang wajar.
- **Auth per-connection**: validasi auth saat koneksi dibuka DAN
  pertimbangkan re-validasi kalau koneksi long-lived (token bisa
  expired di tengah sesi).
- Jangan perkenalkan message broker/pub-sub baru (Redis pub/sub, RabbitMQ,
  dst) kecuali sudah ada kebutuhan nyata yang terukur — ikuti prinsip
  Core Discipline §F (Don't Overengineer).

## SEBELUM TAMBAH DEPENDENCY BARU
Cek apakah sudah ada di project → cek apakah kode existing sudah cukup →
verifikasi kompatibilitas package manager → pertimbangkan security &
maintenance jangka panjang → tambah hanya kalau genuinely perlu. Jangan
perkenalkan microservices/Redis/message queue hanya karena "mungkin
berguna" — hanya untuk kebutuhan nyata yang terukur. Jangan upgrade
dependency atau ganti library existing tanpa alasan.

## THIRD-PARTY API INTEGRATION
Inspeksi client existing, dokumentasi provider, versi SDK, terms, data
flow, limit, dan fasilitas sandbox — jangan mengarang endpoint provider,
response field, algoritma signing, atau jaminan retry.
- **Credential**: secret API key/webhook secret tetap di server (secret
  manager/env config) — jangan pernah di frontend bundle, repo, URL, log,
  atau skill file. Pisahkan akun test/live & secret-nya.
- **Data yang dikirim**: kirim hanya data user yang perlu; review kebutuhan
  privacy (lihat security-team.md §Data Privacy) sebelum menambah
  recipient data baru.
- **Resilience**: set deadline koneksi/request, payload limit, concurrency
  terbatas. Handle rate limit provider dengan retry terbatas + exponential
  backoff + jitter. Retry HANYA untuk failure transient yang terdokumentasi
  atau operasi idempotent — jangan retry input invalid/credential ditolak.
- **Webhook**: verifikasi signature sesuai proses resmi provider, pakai
  raw bytes request sebelum parsing, validasi timestamp tolerance, pakai
  constant-time comparison untuk MAC custom. Cegah replay/duplikasi pakai
  event ID + dedup + transactional state change. Expect retry/duplikasi/
  delivery telat/out-of-order.

## TESTING DENGAN THIRD-PARTY API BERBAYAR/QUOTA-LIMITED
Untuk API eksternal yang berbayar per-call atau punya quota ketat
(contoh: LLM API, Google Maps, SMS gateway):

- Cek dulu apakah provider punya sandbox/test mode gratis — pakai itu
  untuk development & test, bukan production key.
- Untuk unit/integration test: mock response API (jangan panggil API
  asli) — pastikan mock mencerminkan shape response nyata (cek
  dokumentasi provider, jangan mengarang field).
- Untuk E2E test yang genuinely perlu panggil API asli: batasi jumlah
  panggilan (jangan loop test yang manggil API berkali-kali tanpa
  perlu), dan pastikan pakai key test/sandbox kalau tersedia.
- Kalau task butuh verifikasi manual sekali panggil API asli buat
  memastikan integrasi jalan: informasikan ke user dulu kalau ini akan
  makan quota/biaya, terutama kalau key yang dipakai adalah production
  key.

## API VERSIONING (untuk API publik atau consumer yang deploy independen)
Pakai strategi versioning eksplisit hanya kalau memang butuh coexist
kontrak yang incompatible. Jangan versioning tiap refactor internal/bug
fix/perubahan compatible. Identifikasi consumer, bandingkan schema/type/
status code/error format sebelum ubah kontrak — bahkan field additive bisa
break strict client. Pertahankan kontrak yang didukung dalam satu versi;
kalau incompatible terpaksa, isolasi kontrak baru + sediakan migration
path yang teruji.

## KOORDINASI DENGAN TIM LAIN
Skema database baru/berubah → koordinasi Database Team. Endpoint sensitif
(auth, payment, upload) → koordinasi Security Team sebelum dianggap
selesai. Perubahan kontrak API yang dipakai frontend → beri tahu Frontend
Team, cek regresi consumer existing.

## OUTPUT KE TEAM LEAD
Endpoint/service dibuat/diubah, validasi yang diterapkan, dependency baru
(+alasan), hasil verifikasi (test yang benar-benar dijalankan).
