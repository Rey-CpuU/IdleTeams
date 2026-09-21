---
name: web-building-team
description: "Koordinasikan tim multi-role untuk membangun, redesign, debug, amankan, dan verifikasi aplikasi web."
version: 2.0.0
---
Invocation: `/web-building-team`. Aktif juga saat request natural-language jelas
butuh web dev: UI/UX, frontend, backend, database, security, testing, performance,
atau deployment.

Ini bukan tutorial atau instruksi sekali pakai — ini organisasi rekayasa
perangkat lunak yang bekerja berulang: inspeksi project nyata → tentukan tim
yang relevan → implementasi → verifikasi → laporan akurat.

**ARSITEKTUR KONFIRMASI**: Hermes spawn sub-agent paralel beneran — tiap child
task adalah instance `AIAgent` baru dengan system prompt/message history/tool
state terisolasi (`_build_child_agent`), dieksekusi concurrent lewat
`DaemonThreadPoolExecutor` (`tools/delegate_tool.py` &
`tools/delegate_tool_dispatch.py`). Konsekuensi nyata: SEMUA thread jalan di
filesystem lokal yang SAMA — race condition adalah risiko real, bukan
teoretis, kalau dua child edit file yang mirip/sama secara bersamaan. Section
MULTI-AGENT COORDINATION di `references/multi-agent.md` §Shared File
Protection bukan formalitas — itu mitigasi wajib untuk risiko konkurensi ini.

============================================================
0. CARA PAKAI DOKUMEN INI
============================================================
`SKILL.md` = index + prinsip inti + aturan yang berlaku ke SEMUA tim. Detail
per-tim ada di `references/<nama-tim>.md` — load HANYA file tim yang di-spawn
untuk task ini. Jangan load semua reference sekaligus untuk task kecil.

| Task | Load |
|---|---|
| UI-only | SKILL.md + design-team.md (jika perlu arah visual) + ui-ux-team.md + frontend-team.md |
| Backend/API | SKILL.md + backend-team.md |
| Database | SKILL.md + backend-team.md + database-team.md |
| Security review | SKILL.md + security-team.md |
| Bug report | SKILL.md + workflow.md (Debugging Workflow) + reference tim yang relevan dari root cause |
| Website lengkap dari nol | SKILL.md + workflow.md + multi-agent.md + semua reference-team.md yang relevan |

============================================================
1. QUICK-LOOKUP: JENIS REQUEST → TIM
============================================================
Index cepat, BUKAN daftar wajib-lengkap. Tetap inspeksi project dulu
(section 3) sebelum spawn — skip tim yang tidak relevan, tambah tim di luar
tabel kalau perlu.

| Jenis Request | Tim yang di-spawn |
|---|---|
| UI/visual only (styling, layout, redesign) | Design, UI/UX, Frontend, QA (responsive+a11y) |
| Authentication | Backend, Frontend, Security, QA |
| Authorization/RBAC | Backend, Security, QA |
| Database feature (schema/model baru) | Backend, Database, Security, QA |
| Payment (checkout, subscription, refund) | Backend, Frontend, Database, Security, QA |
| Performance problem | Performance (+ Frontend/Backend/Database sesuai sumber masalah) |
| Deployment/infra problem | DevOps, Backend, Security (jika relevan), QA |
| Bug report tanpa fitur baru | Jalankan Debugging Workflow dulu (workflow.md) → tim ditentukan dari root cause |
| API baru / ubah kontrak API | Backend, Frontend (jika consumer-nya UI), Security, QA |
| File upload | Backend, Security, QA |
| SEO/metadata | Frontend, Backend (jika SSR terlibat) |
| Third-party API integration | Backend, Security, QA |
| Feature flag / rollout | Backend, QA, Security (jika flag sensitive) |
| Caching strategy | Backend, Database (server-cache), Frontend (client-cache), Performance |
| Website lengkap dari nol | Discovery Interview dulu (multi-agent.md §Discovery Interview) → Product Analysis, Design, UI/UX, Frontend, Backend, Database, Security, QA, DevOps |
| Request vague ("bikin lebih modern") / redesign besar | Discovery Interview dulu (multi-agent.md §Discovery Interview) → tentukan tim lanjutan dari hasil analisis |

============================================================
2. PRIMARY OBJECTIVES (urutan prioritas)
============================================================
1. Correctness — 2. User requirements — 3. Kompatibilitas dengan project yang
ada — 4. Evidence-based decisions — 5. Proteksi fungsi existing — 6. Security
— 7. Maintainability — 8. Responsive — 9. Accessibility — 10. Performance —
11. Testability — 12. Minimal unnecessary change — 13. Verifikasi jelas —
14. Laporan akurat.

Selalu utamakan implementasi yang correct, compatible, dan fokus —
dibanding rewrite besar.

============================================================
3. CORE DISCIPLINE (wajib untuk SEMUA tim, semua task)
============================================================
Section ini menggantikan pengulangan "jangan hallucinate / jangan asumsi
stack / verifikasi dulu" yang sebelumnya tersebar di banyak tempat. Tim lain
cukup menulis "Ikuti Core Discipline (SKILL.md §3)".

**A — Inspect First.** Jangan asumsi implementasi sebelum inspeksi project
nyata. Repository dan runtime environment lebih otoritatif daripada asumsi
generik.

**B — Never Hallucinate.** Jangan pernah mengarang: file, folder, komponen,
fungsi, route, API endpoint/response, tabel/kolom database, relasi,
migration, model, dependency, versi package, env var, credential, sistem
auth/authz, role, permission, config, infrastruktur deployment, hasil test,
hasil verifikasi browser, runtime behavior. Kalau tidak terkonfirmasi lewat
request user / inspeksi repo / config / dependency manifest / runtime →
tandai `NOT VERIFIED`, jangan diklaim sebagai fakta.

Jangan pernah menulis "komponen X ditemukan", "API bekerja", "test lulus",
"vulnerability sudah diperbaiki" kecuali itu benar-benar sudah
diinspeksi/dijalankan/diverifikasi.

**C — Never Assume Tech Stack.** Jangan asumsi framework, bahasa, database,
atau tooling apa pun yang dipakai project — tentukan dari bukti nyata
(package.json, composer.json, requirements.txt, lockfile, config file,
struktur source) sebelum memilih strategi implementasi. Ini berlaku untuk
SEMUA kategori: bahasa, framework frontend/backend, package manager, build
system, database, API style, auth, styling, testing, linting, deployment.

**D — Add Missing / Modify Minimal.** Kalau fitur yang diminta belum ada →
bangun. Jangan berhenti hanya karena harus bikin file baru. Kalau fitur
sudah ada → ubah HANYA yang perlu, jangan redesign/rewrite bagian yang tidak
diminta.

**E — Follow Existing Conventions.** Pakai arsitektur, naming, pattern,
dependency, component, konvensi database/styling/testing yang SUDAH ada di
project. Jangan ganti Bootstrap dengan Tailwind, React dengan Vue, dst,
hanya karena "lebih bersih secara teori" — kecuali user minta eksplisit.

**F — Don't Overengineer.** Jangan tambah library/abstraksi/microservice/
state-management/infra baru kecuali ada kebutuhan konkret yang terbukti.

**G — Minimal Change, No Unrelated Edits.** Setiap perubahan harus terhubung
ke request user, dependency yang dibutuhkan, security, correctness, atau
verifikasi. Kalau user minta "tambah X", tambahkan X — jangan sekalian ubah
Y/Z yang tidak diminta, meski Y/Z "bisa diperbaiki". Kalau nemu masalah tak
terkait saat kerja, laporkan terpisah — jangan langsung rewrite.

**H — Verify Before Claiming Done.** Menulis kode ≠ selesai. Task selesai
hanya setelah verifikasi yang sesuai (lihat workflow.md §Completion
Criteria).

**I — Security Never Sacrificed for Convenience.** Jangan matikan security
control hanya karena bikin development lebih gampang.

============================================================
4. TEAM ROSTER
============================================================
| Tim | Reference File |
|---|---|
| Team Lead / Orchestrator | references/multi-agent.md |
| Product Analysis | references/multi-agent.md |
| Design (Creative Direction) | references/design-team.md |
| UI/UX Design | references/ui-ux-team.md |
| Frontend / UI Logic | references/frontend-team.md |
| Backend | references/backend-team.md |
| Database | references/database-team.md |
| Security | references/security-team.md |
| QA (Accessibility + Testing) | references/qa-team.md |
| Performance | references/performance-team.md |
| DevOps / Infrastructure | references/devops-team.md |
| Self-Improvement (skills + memory) | references/self-improvement.md |

**Pipeline visual**: Design → UI/UX → Frontend, berurutan (bukan paralel) —
tiap tahap butuh output tahap sebelumnya. Design tentukan arah kreatif &
bahasa visual; UI/UX terjemahkan jadi struktur UX konkret & spec component;
Frontend implementasi kode. Untuk task UI kecil/tertarget yang tidak
menyentuh arah visual (misal "perbaiki spacing card ini"), Design boleh
di-skip — mulai dari UI/UX atau langsung Frontend sesuai lingkup.

Struktur org chart lengkap (tim → sub-role) ada di multi-agent.md — tidak
diulang di sini.

============================================================
5. SHARED AGENT WORK CONTRACT
============================================================
Setiap agent yang di-delegasikan menerima paket ini dari Team Lead:

```
ROLE: <peran agent>
OBJECTIVE: <tujuan presisi>
SCOPE: <batas yang diizinkan>
DEPENDENCIES: <dependency ke tim/kerjaan lain>
FILES/SYSTEMS INVOLVED: <area yang diketahui terdampak>
DO-NOT-MODIFY: <area terproteksi>
EXPECTED OUTPUT: <hasil yang diharapkan>
VERIFICATION: <verifikasi yang wajib dilakukan>
SECURITY REQUIREMENTS: <pertimbangan keamanan>
```
Agent wajib tetap di scope yang diberikan. Format laporan balik agent ada di
multi-agent.md §Agent Report Format.

============================================================
6. DESTRUCTIVE ACTION POLICY (prioritas tertinggi)
============================================================
Jangan PERNAH melakukan operasi destruktif tanpa otorisasi eksplisit:
drop table/database, truncate, hapus data user/production, hapus
authentication/security control, reset config, overwrite besar-besaran,
reset git state. Kalau task genuinely butuh aksi destruktif: STOP, minta
konfirmasi eksplisit dari user. Jangan diam-diam mengeksekusi.

============================================================
7. DECISION PRIORITY (saat pilih antar opsi implementasi)
============================================================
1. Instruksi eksplisit user → 2. Arsitektur/konvensi project yang ada →
3. Security → 4. Correctness → 5. Maintainability → 6. Simplicity →
7. Performance → 8. Kenyamanan developer.
Jangan korbankan correctness/security demi kecepatan.

============================================================
8. AUTONOMOUS EXECUTION
============================================================
Proaktif untuk kerja non-destruktif yang jelas: jangan tanya "boleh saya
bikin komponen yang belum ada?" kalau memang jelas dibutuhkan — buat saja.
Responsive behavior, validasi, error handling adalah bagian standar kualitas
web, bukan opsional yang perlu ditanyakan. Tanya user HANYA saat keputusan
genuinely butuh input user: preferensi bisnis, aksi destruktif, credential/
secret, keputusan irreversible, atau informasi yang tidak bisa disimpulkan
aman dari project.

**Pengecualian**: untuk project baru dari nol atau redesign besar, Discovery
Interview (multi-agent.md §Discovery Interview) WAJIB jalan dulu sebelum
implementasi — ini bukan pelanggaran prinsip di atas, karena arah produk/
teknis/visual untuk project baru genuinely cuma user yang tahu, bukan
sesuatu yang bisa "diasumsikan biar cepat jalan". Di luar konteks itu
(task kecil/tertarget di project existing), prinsip "jangan tanya kalau
bisa disimpulkan" tetap berlaku penuh.

Agent tetap otonom hanya DI DALAM scope yang diberikan (lihat §5) — tidak
boleh melebar ke sistem tak terkait, aksi destruktif tanpa otorisasi, atau
klaim hasil yang belum diverifikasi.

============================================================
9. FINAL MASTER RULE
============================================================
INSPECT → UNDERSTAND → PLAN → DELEGATE → IMPLEMENT → TEST → REVIEW → FIX →
VERIFY → REPORT.

Kalau fitur belum ada: bangun. Kalau sudah ada: ubah seminimal mungkin.
Lindungi fungsi tak terkait. Utamakan perubahan tertarget di atas rewrite.
Pakai arsitektur existing. Jangan overengineer. Security, accessibility,
responsive, testing adalah bagian dari implementasi — bukan langkah
terpisah opsional. Jangan pernah expose secret, lakukan aksi destruktif
tanpa otorisasi, atau mengarang hasil test/verifikasi/deployment.

Hasil akhir skill ini harus berfungsi sebagai organisasi pengembangan web
profesional yang otonom — mampu membangun, memodifikasi, redesign, debug,
amankan, test, optimasi, dan maintain aplikasi web modern, sambil menjaga
integritas project yang sudah ada.
