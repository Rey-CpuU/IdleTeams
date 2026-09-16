# Workflow — Reconnaissance, Lifecycle, Verifikasi, Laporan

Ikuti Core Discipline (SKILL.md §3) untuk semua aturan verifikasi/inspeksi —
tidak diulang di sini.

## PROJECT RECONNAISSANCE
Sebelum mulai kerja, inspeksi area yang relevan dengan task:
- **Project**: root dir, source dir, static/public dir, config, docs,
  package manifest.
- **Frontend** (jika relevan): pages/screens, component, layout, routing,
  state management, hooks/utility, API client, form, styling, asset.
- **Backend** (jika relevan): routes, controller, service, middleware,
  model, validasi, format response API, auth, error handling.
- **Database** (jika relevan): schema, migration, model, relasi, seeder,
  index, query yang sudah ada.
- **Quality**: test, lint config, format config, build script, CI config.
- **Infrastructure**: Docker, Nginx/Apache, env config, deploy config,
  monitoring.
Jangan langsung coding sebelum paham sistemnya.

## GIT AWARENESS
Kalau Git tersedia, cek state sebelum perubahan signifikan: branch aktif,
file modified/untracked/staged, perubahan recent yang relevan. Jangan
overwrite kerjaan uncommitted user. Jangan jalankan command destruktif
(`git reset --hard`, `git clean -fd`, force checkout, operasi branch
destruktif) kecuali diminta eksplisit dan jelas diotorisasi.

## MULTI-REPO AWARENESS
Kalau frontend, backend, dan/atau service lain ada di repo terpisah
(bukan monorepo) — inspeksi dulu struktur project untuk pastikan ini
sebelum asumsi:

- Perubahan kontrak API yang menyentuh dua repo (misal backend ubah
  response shape, frontend jadi consumer-nya): kedua sisi perubahan
  harus dikoordinasikan, idealnya backward-compatible dulu di repo
  provider sebelum repo consumer diupdate (hindari breaking deploy
  order).
- Kalau tidak bisa akses langsung ke repo lain (misal frontend task tapi
  backend ada di repo terpisah yang tidak ter-mount): laporkan sebagai
  dependency eksternal ke Team Lead — jangan menebak shape API dari repo
  yang tidak bisa diinspeksi.
- Dokumentasikan kontrak API yang dipakai lintas repo di satu tempat
  yang bisa diakses kedua sisi (OpenAPI spec, shared type definition,
  dst) kalau project sudah punya konvensi ini — jangan buat sistem
  dokumentasi baru kalau belum ada kebutuhan konkret.

## IMPLEMENTATION LIFECYCLE
1. **UNDERSTAND** — ekstrak objective, fitur yang diminta, area terdampak,
   constraint, requirement visual/teknis, fungsi yang harus dilindungi.
2. **INSPECT** — inspeksi repo & environment nyata (lihat Reconnaissance).
3. **PLAN** — rencana internal: file terdampak/dibuat/diubah, data flow, UI
   flow, API flow, dampak database, dampak security, test, risiko,
   dependency.
4. **DELEGATE** — spawn tim/sub-agent yang relevan saja (lihat SKILL.md §1).
5. **IMPLEMENT** — perubahan minimal yang koheren.
6. **INTEGRATE** — selesaikan dependency & konflik antar tim.
7. **VERIFY** — jalankan test/check yang sesuai.
8. **REVIEW** — review implementasi seolah-olah ditulis engineer lain.
9. **FIX** — perbaiki temuan review.
10. **FINAL VERIFY** — jalankan ulang check yang relevan. Task baru dianggap
    selesai setelah tahap ini.

## DEBUGGING WORKFLOW
Saat user laporkan bug:
1. Reproduksi/inspeksi error.
2. Identifikasi sumber sebenarnya.
3. Trace kode yang relevan.
4. Inspeksi dependency terkait.
5. Tentukan root cause.
6. Buat fix seminimal mungkin yang sesuai.
7. Test fix.
8. Cek regresi.
9. Review implikasi security terkait.
10. Laporkan apa yang benar-benar diverifikasi.
Jangan rewrite sistem tak terkait hanya karena "secara teori bisa
diperbaiki".

## COMPLETE WEBSITE WORKFLOW
Untuk request "bikin website/app lengkap": pahami requirement → analisis
user flow → tentukan information architecture → inspeksi/tentukan
arsitektur → desain UI/UX → tentukan struktur component → arsitektur
frontend/backend/database → auth/authz (jika perlu) → kontrak API →
implementasi frontend/backend/database/security/responsive/accessibility →
test → review → fix → re-test → verifikasi final.

## SAAT USER MEMBERIKAN SCREENSHOT
Perlakukan sebagai bukti visual saja. Analisis: struktur, spacing,
hierarchy, komponen, warna, tipografi, gaya visual, pola interaksi yang
terlihat. JANGAN menyimpulkan dari screenshot: framework, arsitektur
backend, database, nilai CSS eksak, nama komponen, logic API, fungsi
tersembunyi. Setelah analisis screenshot, tetap inspeksi source code nyata
sebelum implementasi.

## SAAT USER MEMBERIKAN REFERENCE WEBSITE
Kalau ada kapabilitas browsing, inspeksi situsnya sebagai inspirasi —
jangan copy code/desain berhak cipta secara verbatim. Ambil pola yang
berguna (layout, navigasi, hierarchy informasi, pola interaksi, konsep
spacing/responsive), lalu buat implementasi orisinal yang kompatibel dengan
project user.

## COMPLETION CRITERIA
Task BUKAN selesai hanya karena kode sudah ditulis. Selesai kalau: fitur
yang diminta ada dan sesuai requirement, fungsi terkait yang lama tetap
jalan, arsitektur project dihormati, error handling ada, loading/empty
state ada bila relevan, responsive & accessibility dipertimbangkan, dampak
security direview, test/check relevan dijalankan, perubahan final
direview, tidak ada perubahan tak terkait yang nyelip.

## COMPLETION CRITERIA — TASK ANALISIS/DOKUMENTASI MURNI
Berlaku HANYA untuk task yang eksplisit diminta sebagai analisis/
dokumentasi saja (tanpa implementasi kode) — misal "analisis requirement
dulu", "buatkan rencana teknis", "review arsitektur, jangan diubah
dulu".

Task jenis ini selesai kalau:
- Requirement/pertanyaan user sudah dijawab lengkap berdasar inspeksi
  project nyata (bukan asumsi).
- Semua klaim tentang project (struktur, fungsi existing, dependency)
  sudah terverifikasi lewat inspeksi — tidak ada yang diarang (Core
  Discipline §B).
- Rekomendasi/rencana yang diberikan jelas menyebutkan trade-off, risiko,
  dan area yang butuh keputusan user.
- Kalau ada bagian yang tidak bisa dianalisis (misal butuh akses yang
  tidak tersedia), ditandai eksplisit sebagai gap — bukan diabaikan.
- TIDAK perlu menjalankan test/build/lint (karena tidak ada kode yang
  diubah) — kriteria `§FINAL QUALITY GATE` yang menyebut verifikasi
  kode tidak berlaku untuk task jenis ini.

## FINAL QUALITY GATE
Cek sebelum menyatakan selesai (skip baris yang tidak relevan ke task):
- **Functionality**: fitur ada & jalan, fungsi terkait lama tetap jalan,
  error/loading/empty state ada bila perlu.
- **Frontend**: component maintainable, state management benar, API call
  ter-handle, form tervalidasi, routing jalan, tidak ada re-render
  berlebihan baru.
- **UI/UX**: desktop/tablet/mobile/large-screen jalan, tidak ada horizontal
  overflow, tipografi terbaca, hierarchy jelas, interactive/loading/error/
  empty state ada.
- **Accessibility**: semantic HTML, keyboard nav, focus state, label form,
  error message accessible, contrast, alt text, reduced motion bila
  relevan.
- **Backend**: validasi input, authorization, error handling, query aman,
  pagination bila perlu, logging wajar, kontrak API terverifikasi bila
  memungkinkan.
- **Security**: OWASP risk dipertimbangkan, XSS/SQLi/NoSQLi/CSRF
  dipertimbangkan bila relevan, CORS/CSP dikonfigurasi wajar, auth/authz
  aman, rate limiting bila perlu, cookie aman, password hashing aman,
  secret terlindungi, SSRF/clickjacking dipertimbangkan bila relevan,
  dependency dicek bila relevan.
- **Quality**: type-check/lint/test/build dijalankan bila tersedia,
  verifikasi browser bila tersedia, code review final selesai, tidak ada
  perubahan tak terkait.

## FINAL RESPONSE FORMAT
```
Implemented: apa yang ditambah/diubah/diperbaiki
Files Changed: path file + ringkasan perubahan tiap file
Features: fitur ditambah/diubah
Security: fix/review security yang BENAR-BENAR dilakukan
UI/UX: perbaikan UI/UX yang BENAR-BENAR dilakukan
Backend/API/Database: perubahan aktual
Verification: command/test yang BENAR-BENAR dijalankan, hasil build/lint/
  type-check, hasil cek browser jika benar-benar dilakukan
Not Verified: apa yang tidak sempat/tidak bisa diverifikasi
Notes: keputusan implementasi penting, limitasi tersisa, risiko relevan
```
Jangan pernah klaim: test dijalankan padahal tidak, browser testing terjadi
padahal tidak, API bekerja padahal belum diverifikasi, file ada padahal
belum diinspeksi, vulnerability sudah diperbaiki padahal belum benar-benar
diperbaiki & diverifikasi.

## DOCUMENTATION STANDARDS
Jaga dokumentasi tetap selaras dengan implementasi & konvensi repo yang
ada. Tambahkan info yang berguna, bukan boilerplate yang cuma mengulang
kode yang sudah jelas.
- **Code docs**: pakai JSDoc/PHPDoc/docstring (sesuai ekosistem) untuk
  interface publik, kontrak non-obvious, asumsi security, side effect
  penting, exception, invariant kompleks. Jelaskan KENAPA constraint aneh
  itu ada — jangan duplikasi syntax yang sudah jelas dari kode sendiri.
- **API docs**: untuk REST, jaga OpenAPI/Swagger project (jika dipakai)
  tetap sinkron — cover endpoint, auth/scope, schema request/response,
  validasi/error, pagination, versioning, rate limit. Update README &
  runbook saat arsitektur/dependency/env var/setup/migration/deploy
  berubah. Jangan taruh topologi internal atau credential di dokumentasi
  publik.
- **Architecture Decision Record (ADR)**: buat/update untuk keputusan
  berdampak jangka panjang (pilihan data store, batas auth, kebijakan
  versi API, integrasi besar, model caching, arsitektur deployment).
  Include: context, constraint, alternatif yang dipertimbangkan, keputusan,
  rationale, consequence, status. Jangan buat ADR untuk refactor kecil.
