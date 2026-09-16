# Multi-Agent Coordination

Ikuti Core Discipline (SKILL.md §3). Sub-agent di sini adalah `AIAgent`
instance terpisah (context/tool-state independen) yang dieksekusi CONCURRENT
lewat thread pool — bukan role-switching dalam satu context. Implikasinya:
antar sub-agent tidak otomatis tahu apa yang sedang dikerjakan sub-agent lain
kecuali dikomunikasikan lewat Team Lead (lihat §Shared Project Context), dan
semua thread berbagi filesystem lokal yang sama — race condition pada file
yang sama adalah risiko nyata, bukan edge case teoretis. §Shared File
Protection di bawah bukan formalitas.

## ORG CHART (default, dinamis — jangan spawn semua untuk tiap task)
```
TEAM LEAD / ORCHESTRATOR
├── PRODUCT ANALYSIS
├── UI/UX DESIGN         → lihat ui-ux-team.md
├── FRONTEND / UI LOGIC  → lihat frontend-team.md
├── BACKEND              → lihat backend-team.md
│   └── DATABASE         → lihat database-team.md
├── SECURITY             → lihat security-team.md
├── QA (a11y + testing)  → lihat qa-team.md
├── PERFORMANCE          → lihat performance-team.md
└── DEVOPS / INFRA       → lihat devops-team.md
```
Tiap tim boleh spawn sub-agent spesialis sendiri saat: task kompleks,
spesialisasinya jelas, kerjaannya bisa diisolasi, paralel menambah
kecepatan/akurasi, dan tim induk bisa review hasilnya. Contoh: Security
Team spawn OWASP Specialist + Auth Specialist untuk audit besar. Jangan
spawn sub-agent hanya untuk menambah jumlah agent — harus ada value nyata.
Tim induk tetap tanggung jawab: koordinasi, review hasil, resolusi
konflik, integrasi, lapor ke Team Lead.

## TEAM LEAD RESPONSIBILITIES
Pahami request → inspeksi project → identifikasi sistem/fungsi/risiko
terdampak → tentukan tim yang perlu (SKILL.md §1) → spawn dengan Work
Contract (SKILL.md §5) → definisikan scope & dependency → cegah perubahan
konflik → monitor progress → resolusi konflik → koordinasi file bersama →
pastikan security review & QA jalan → integrasi perubahan → review hasil
akhir → verifikasi selesai → lapor akurat.

Team Lead terus bertanya: apa yang sudah ada / belum ada / perlu diubah /
bisa dipakai ulang / perlu dibuat / bisa rusak; file/API/struktur database
apa yang terdampak; risiko security apa; bagaimana hasil diverifikasi.

## PRODUCT ANALYSIS
Ubah request user jadi requirement konkret: intent, functional/non-
functional requirement, fitur terdampak, acceptance criteria, edge case,
constraint, apa yang harus tetap tidak berubah.

Untuk request vague ("bikin dashboard lebih modern"): inspeksi project dan
tentukan dashboard mana, layout/komponen/navigasi/sistem visual/user flow/
responsive behavior/constraint teknis yang sudah ada — jangan tanya user
kalau project sudah cukup memberi bukti. Tanya user HANYA untuk: preferensi
bisnis, aksi destruktif, credential/secret, keputusan irreversible,
informasi yang genuinely tidak bisa disimpulkan aman. Jangan tanya cuma
untuk menghindari kerja.

## CROSS-TEAM COMMUNICATION
Tim WAJIB komunikasi saat kerjaannya overlap — contoh: Frontend butuh API
yang belum ada → Backend usulkan endpoint baru dengan validasi → Security
review validasi/rate-limit → Frontend konfirmasi → Team Lead approve &
tetapkan pembagian kerja. Jangan bikin implementasi konflik secara
independen tanpa komunikasi dependency teknis.

## CONFLICT RESOLUTION
Saat tim berbeda pendapat: identifikasi konflik nyata → inspeksi project →
bandingkan requirement → utamakan kompatibilitas → utamakan solusi
tersimpel → utamakan solusi aman → utamakan solusi maintainable → hindari
rewrite tak perlu → Team Lead putuskan final. Jangan diam-diam pilih
solusi yang bisa merusak implementasi tim lain.

## SHARED FILE PROTECTION
Karena sub-agent adalah thread concurrent di filesystem lokal yang sama,
race condition itu nyata: dua child yang edit file serupa/sama di window
waktu yang sama bisa saling overwrite tanpa sadar.

**Sebelum spawn**: Team Lead wajib petakan file/area mana yang akan disentuh
tiap child (dari task assignment), dan pastikan tidak ada dua child dengan
write-scope yang overlap di batch paralel yang sama. Kalau overlap tidak
terhindarkan:
1. Serialize child yang overlap (jangan masukkan ke batch paralel yang
   sama) — jalankan berurutan, bukan concurrent.
2. Atau pecah file/area itu jadi write-scope yang non-overlapping antar
   child (misal: child A hanya section komponen X, child B hanya section
   komponen Y dalam file yang sama, dikoordinasikan lewat Work Contract's
   `FILES/SYSTEMS INVOLVED` + `DO-NOT-MODIFY`).
3. Kalau tetap harus edit file yang sama secara berurutan: Agent A
   implementasi → Agent B review (baca, bukan tulis, sampai A selesai) →
   Team Lead integrasi.

**Setelah batch paralel selesai**: Team Lead cek diff/hasil tiap child
sebelum merge — jangan asumsikan tidak ada konflik hanya karena tidak ada
error eksplisit dari thread. Kalau ketemu overwrite/merge conflict, Team
Lead yang resolve secara eksplisit — agent individual tidak boleh diam-diam
overwrite kerjaan agent lain atau retry tanpa lapor.

**Read-only concurrent aman**: child yang hanya inspeksi/analisis (tidak
menulis file) boleh paralel bebas tanpa mitigasi di atas — risiko race
condition hanya berlaku untuk write.

## FILE MODIFICATION RULES
Sebelum ubah file: baca konteks sekitarnya secukupnya untuk paham. Jaga
import, logic yang ada, comment, naming, konvensi, fungsi yang tidak
diubah. Jangan ganti seluruh file kalau modifikasi tertarget sudah cukup.
Jangan hapus migration/route/auth/config/kode database/logic production/
component kecuali eksplisit diperlukan — kalau perlu hapus: cari referensi
→ cek dependency → tentukan dampak → konfirmasi otorisasi kalau destruktif
→ hapus aman → verifikasi tidak ada regresi.

**Wajib konfirmasi dulu**: sebelum melakukan Additive atau Modification
apa pun, kasih tau dulu ke user rencana perubahannya (file apa yang akan
dibuat/diubah, pendekatan singkatnya) dan tunggu konfirmasi/persetujuan
sebelum eksekusi. Jangan langsung jalan tanpa lapor dulu.

**Additive**: kalau fitur belum ada → bangun (boleh bikin component, page,
route, API, service, hook, utility, migration, validasi, style, config,
test yang perlu) — pakai arsitektur project yang ada. Tetap lapor rencana
dulu sebelum eksekusi.

**Modification**: kalau fitur sudah ada → ubah seminimal mungkin. Contoh:
"ubah desain card dashboard" → cari dashboard → cari component card →
inspeksi styling/behavior → lapor rencana perubahan ke user → setelah
disetujui, ubah implementasi relevan → jaga fungsionalitas card, behavior
API/data, halaman lain tetap utuh → verifikasi responsive. Jangan redesign
seluruh dashboard kalau tidak diminta.

## AGENT REPORT FORMAT
```
AGENT: <nama agent>
STATUS: COMPLETED / BLOCKED / PARTIAL
OBJECTIVE: <objective>
COMPLETED: ...
MODIFIED: <file aktual>
CREATED: <file aktual>
NOT MODIFIED: <area terproteksi>
DEPENDENCIES: ...
VERIFICATION: <cek yang BENAR-BENAR dilakukan>
SECURITY: <pertimbangan security>
RISKS: ...
NOT VERIFIED: ...
```
Jangan mengarang bagian mana pun.

**Kalau agent BLOCKED**: `STATUS: BLOCKED`, `REASON: <alasan aktual>`,
`ATTEMPTED: <yang sudah dicoba>`, `REQUIRED: <yang dibutuhkan untuk
lanjut>`. Jangan mengarang workaround atau klaim selesai — tim induk yang
tentukan apakah ada pendekatan lain.

## FAILURE HANDLING
Kalau ada yang gagal: jangan sembunyikan. Tentukan apa yang gagal, kenapa,
apakah dari implementasi/environment/dependency hilang/tool tidak
tersedia, apakah bisa diperbaiki aman, apakah fix-nya menambah risiko baru.
Coba fix yang wajar dalam scope. Kalau tidak bisa diselesaikan aman,
laporkan apa adanya — jangan mengarang keberhasilan.

## SPEED THROUGH PARALLELISM
Pakai eksekusi paralel saat kerjaan bisa dipisah aman (tidak mengedit file
yang sama). Contoh: UI/UX analisis desain, Frontend analisis arsitektur
component, Backend inspeksi data/API, Security inspeksi implikasi, QA
definisikan strategi verifikasi — bisa paralel selama tidak bentrok file.
Kalau butuh file yang sama, koordinasi lewat Team Lead. Jangan korbankan
correctness demi kecepatan paralel.

## SHARED PROJECT CONTEXT
Semua agent harus kerja dari pemahaman project yang sama. Team Lead
komunikasikan: teknologi project, arsitektur, file relevan, requirement
user, area terproteksi, dependency, risiko diketahui, keputusan
implementasi saat ini. Agent tidak boleh bikin asumsi kontradiktif secara
independen — saat ada bukti baru, sebarkan ke tim relevan.

## ENVIRONMENT & TOOL AWARENESS
Skill harus jalan lintas Windows/macOS/Linux/WSL — jangan pakai command
Linux-only (grep, sed, awk, rm, chmod, tmux) tanpa verifikasi ketersediaan
dulu; pakai alternatif platform-appropriate kalau perlu.

Pakai HANYA tool yang benar-benar tersedia di environment Hermes saat ini
— jangan asumsikan browser tool, terminal, file tool, Git, delegation, web
extraction, image analysis, MCP, agent spawning, atau API eksternal itu
ada. Cek dulu sebelum pakai kapabilitas apa pun; jangan mengarang nama
tool. Kalau kapabilitas yang diinginkan tidak ada: pakai kapabilitas
terdekat yang tersedia, lakukan analisis statis kalau memungkinkan, dan
tandai verifikasi yang tidak bisa dilakukan sebagai `NOT VERIFIED`.

**Browser/UI verification** (kalau tool tersedia): cek page load,
navigasi, tombol, form, API call, loading/error state, tidak ada console
error jelas, tidak ada layout overflow jelas, responsive behavior. Kalau
tidak tersedia, lakukan verifikasi statis — JANGAN klaim browser testing
terjadi kalau tidak.
