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
├── VISUAL PIPELINE (berurutan, bukan paralel)
│   ├── 1. DESIGN (creative direction) → lihat design-team.md
│   ├── 2. UI/UX DESIGN (UX structure)  → lihat ui-ux-team.md
│   └── 3. FRONTEND / UI LOGIC (impl.)  → lihat frontend-team.md
├── BACKEND              → lihat backend-team.md
│   └── DATABASE         → lihat database-team.md
├── SECURITY             → lihat security-team.md
├── QA (a11y + testing)  → lihat qa-team.md
├── PERFORMANCE          → lihat performance-team.md
└── DEVOPS / INFRA       → lihat devops-team.md
```
Visual Pipeline dijalankan berurutan karena tiap tahap bergantung pada output
tahap sebelumnya (lihat design-team.md §Handoff). Tim lain di luar pipeline
ini boleh paralel sesuai §Speed Through Parallelism di bawah.

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

**Kapan tanya vs kapan langsung jalan** — dua mode:
1. **Task kecil/tertarget** (bug fix, tambah 1 fitur ke project existing,
   perbaikan spesifik): tetap seperti biasa — inspeksi project dulu,
   simpulkan dari bukti yang ada, JANGAN tanya kalau project sudah cukup
   memberi bukti. Tanya user HANYA untuk: preferensi bisnis, aksi
   destruktif, credential/secret, keputusan irreversible, informasi yang
   genuinely tidak bisa disimpulkan aman. Ini konsisten dengan Autonomous
   Execution Policy (SKILL.md §8) — jangan minta izin untuk kerja
   non-destruktif yang jelas.
2. **Project baru dari nol / redesign besar / request masih sangat
   terbuka** ("bikin website buat usaha gw", "bikin aplikasi todo",
   "redesign total dashboard ini"): WAJIB jalankan **Discovery Interview**
   di bawah sebelum planning/spawning tim lain — lihat §Discovery
   Interview.

## DISCOVERY INTERVIEW (wajib untuk project baru & redesign besar)
Tujuannya: jangan mulai coding/planning sampai gambaran web-nya cukup
jelas — apa fungsinya, dibangun pakai apa, dan mau terlihat seperti apa.
Ini di atas §Product Analysis biasa (yang fokus requirement fungsional) —
Discovery Interview lebih luas, mencakup arah teknis & visual dari awal.

**Trigger**: request bikin website/app baru dari nol, atau redesign besar
yang mengubah arah produk secara signifikan (bukan sekadar polish). Kalau
project sudah ada dan request-nya jelas/tertarget (lihat mode 1 di atas),
SKIP Discovery Interview — langsung ke alur normal.

**Prinsip tanya**:
- Kalau project sudah ada (bukan dari nol) dan jawabannya bisa
  disimpulkan dari inspeksi repo (framework, DB yang dipakai, dst) —
  JANGAN tanya itu, konfirmasi singkat saja dari hasil inspeksi. Discovery
  Interview untuk info yang genuinely cuma user yang tahu (tujuan bisnis,
  preferensi visual, dst), bukan pengganti inspeksi.
- Tanya bertahap per kelompok topik (jangan lempar 20 pertanyaan
  sekaligus dalam satu pesan) — mulai dari yang paling menentukan arah
  (tujuan & scope), baru ke teknis, baru ke UI/visual paling akhir &
  paling detail.
- Berhenti tanya begitu cukup jelas untuk mulai PLAN (workflow.md) —
  Discovery Interview bukan interogasi tanpa akhir. Kalau user jawab
  singkat/"terserah kamu aja", itu sinyal berhenti tanya & lanjut dengan
  rekomendasi masuk akal (nyatakan asumsinya eksplisit), bukan alasan
  untuk terus menggali.

**Kelompok pertanyaan** (pakai `ask_user_input_v0`-style choice bila
tersedia, atau pertanyaan terbuka singkat):

1. **Tujuan & scope** — web/app ini buat apa (bisnis/portofolio/tool
   internal/produk SaaS/dst), siapa target usernya, masalah utama apa
   yang mau diselesaikan, fitur inti yang WAJIB ada vs nice-to-have.
2. **Tech stack** — ada preferensi framework frontend/backend, atau
   serahkan ke rekomendasi Team Lead berdasar kebutuhan? Dari nol atau
   ada starting point/boilerplate tertentu?
3. **Database & data** — data apa saja yang perlu disimpan (garis besar),
   ada preferensi SQL vs NoSQL / provider tertentu (Postgres, MySQL,
   MongoDB, Supabase, Firebase, dst), atau serahkan ke rekomendasi.
4. **Auth & role** — perlu sistem login/register? Ada jenis user berbeda
   (admin/member/guest, dst)?
5. **Deployment & constraint** — akan di-deploy ke mana (Vercel, VPS,
   shared hosting, dst — kalau belum tahu, tidak apa, skip), ada
   batasan teknis/budget/timeline yang perlu diperhitungkan dari awal.

6. **UI/UX & Visual Direction** (bagian paling banyak pertanyaannya —
   ini akan langsung jadi input Design Team & UI/UX Team, jadi makin
   jelas di sini makin sedikit revisi nanti):
   - Mood/vibe yang diinginkan: modern-minimalis, bold-energetic,
     playful, premium/elegan, corporate-serius, dark & moody, retro,
     brutalist, dll — atau kombinasi/istilah sendiri.
   - Ada website/produk lain yang jadi referensi visual ("pengen kayak
     X tapi..." )? Boleh kasih link/screenshot.
   - Preferensi warna: ada warna brand yang wajib dipakai, atau bebas
     Design Team tentukan berdasar mood di atas?
   - Light mode, dark mode, atau dua-duanya (toggle)?
   - Tingkat animasi/motion: minim & subtle, sedang (hover/transition
     wajar), atau banyak & expresif (animasi masuk, parallax,
     interactive effects)? (Ini menentukan seberapa berat React Bits
     dipakai — lihat frontend-team.md §Sumber Component.)
   - Gaya layout yang disukai: clean & banyak whitespace, dense/
     information-heavy, card-based, grid-heavy, dst.
   - Tipografi: ada preferensi font/karakter huruf (modern sans-serif,
     serif elegan, monospace/technical, dst), atau serahkan ke Design
     Team?
   - Sudah punya brand asset (logo, palet warna resmi, font resmi)? Kalau
     ada, minta dilampirkan/dijelaskan.
   - Device utama yang dituju: mobile-first, desktop-first, atau
     seimbang keduanya?
   - Ada elemen UI spesifik yang WAJIB ada (dashboard dengan chart,
     tabel data besar, form panjang, galeri gambar, chat, dst) yang
     perlu diperhitungkan dari sisi desain sejak awal?

**Setelah cukup jelas**: rangkum hasil Discovery jadi brief singkat
(tujuan, stack, DB, auth, arah visual) untuk dikonfirmasi user sebelum
Team Lead lanjut ke PLAN & spawn tim (workflow.md §Implementation
Lifecycle). Simpan hasil ini sebagai konteks yang dibagi ke semua tim
(§Shared Project Context) — terutama ke Design Team sebagai brief awal
(lihat design-team.md §Sebelum Mulai).

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

**Additive**: kalau fitur belum ada → bangun (boleh bikin component, page,
route, API, service, hook, utility, migration, validasi, style, config,
test yang perlu) — pakai arsitektur project yang ada.

**Modification**: kalau fitur sudah ada → ubah seminimal mungkin. Contoh:
"ubah desain card dashboard" → cari dashboard → cari component card →
inspeksi styling/behavior → ubah implementasi relevan → jaga
fungsionalitas card, behavior API/data, halaman lain tetap utuh → verifikasi
responsive. Jangan redesign seluruh dashboard kalau tidak diminta.

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

## FALLBACK SAAT SUB-AGENT GAGAL
Sub-agent (level 1, di-spawn tim domain) bisa gagal/blocked (lihat §Failure
Handling & Agent Report Format). Jangan biarkan kegagalan itu diam-diam
retry berulang tanpa batas atau bikin task mandek.

**Trigger fallback** — salah satu dari:
1. Sub-agent lapor `STATUS: BLOCKED` atau `PARTIAL` dan attempt perbaikan
   dalam scope-nya sendiri sudah dicoba tapi tetap gagal.
2. Sub-agent gagal 2x berturut-turut untuk objective yang sama (hitung
   dari REQUIRED yang diminta sub-agent vs hasil retry).
3. Blocker ternyata butuh keputusan/konteks yang cuma dipunyai Team Lead
   (cross-team, prioritas, trade-off) — bukan sesuatu yang bisa
   diselesaikan sub-agent sendirian.
4. Task sisa terlalu kecil/urgent untuk membenarkan delegation round baru.

**Urutan eskalasi** (coba dari level termurah dulu, jangan lompat langsung
ke fallback penuh kecuali blocker jelas butuh itu):

1. **Level 2 sub-agent (bantuan langsung ke Team Lead)** — Team Lead boleh
   spawn sub-agent BARU yang di-spawn langsung oleh Team Lead sendiri
   (bukan oleh tim domain yang gagal), dengan scope SEMPIT: bantu diagnosis
   kenapa level 1 gagal, coba pendekatan alternatif yang spesifik, atau
   verifikasi ulang satu hal spesifik. Level 2 BUKAN tim domain baru —
   perannya cuma membantu Team Lead menyelesaikan blocker yang ada, dengan
   Work Contract (SKILL.md §5) yang ditulis Team Lead sendiri, scope lebih
   sempit dari sub-agent level 1 yang gagal.
2. **Team Lead ambil alih sebagai main worker** — kalau level 2 juga gagal,
   atau blocker jelas butuh keputusan/eksekusi langsung Team Lead (bukan
   didelegasikan lagi): Team Lead kerjakan sendiri bagian yang gagal itu.
   Team Lead tetap wajib ikuti Core Discipline (SKILL.md §3) dan
   COMPLETION CRITERIA (workflow.md) yang sama seperti sub-agent biasa —
   fallback bukan alasan untuk skip verifikasi.
   Saat mengambil alih, Team Lead melapor pakai Agent Report Format
   (§Agent Report Format) dengan `AGENT: TEAM LEAD (fallback)`, dan wajib
   isi field `REASON FOR FALLBACK` singkat: kenapa level 1/level 2 gagal.

**Batas eskalasi**: maksimal 1x putaran level 2 sebelum fallback ke Team
Lead — jangan spawn level 2 berulang-ulang untuk blocker yang sama
(itu tanda masalahnya bukan soal eksekusi, tapi butuh keputusan/informasi
dari user). Kalau Team Lead sendiri juga blocked (butuh keputusan
bisnis, credential, atau otorisasi destruktif), STOP dan laporkan ke user
apa adanya — jangan terus eskalasi ke "level 3" tanpa akhir.

**Jangan**: retry sub-agent yang sama dengan instruksi identik berharap
hasil beda; diam-diam lanjut tanpa lapor kegagalan sebelumnya; klaim task
selesai dari hasil fallback tanpa verifikasi yang sama ketatnya dengan
alur normal.

### FALLBACK DI VISUAL PIPELINE (Design → UI/UX → Frontend)
Pipeline ini sequential — tahap berikutnya butuh output tahap sebelumnya
(lihat design-team.md §Handoff). Ini bikin fallback-nya beda dari tim
independen biasa: kegagalan satu tahap otomatis memblokir tahap
berikutnya, jadi tidak bisa "skip aja, tim lain jalan paralel".

- **Design gagal/blocked** (misal: brief user terlalu vague untuk
  ditentukan arah kreatifnya, atau ada konflik brand yang tidak bisa
  diresolusi sendiri): JANGAN teruskan ke UI/UX dengan asumsi arah visual
  seadanya. Coba Level 2 (bantu Team Lead perjelas brief/batasan) dulu.
  Kalau tetap gagal, Team Lead ambil alih menentukan arah visual minimal
  yang aman (ikuti identitas existing project kalau ada, lihat
  §Existing Design Preservation di ui-ux-team.md) — atau, kalau keputusan
  itu genuinely butuh preferensi user (misal user diminta pilih
  moodboard), STOP dan tanya user. Jangan lanjut ke UI/UX dengan brief
  kosong.
- **UI/UX gagal/blocked** setelah Design selesai: masalahnya biasanya arah
  Design ternyata tidak feasible diterjemahkan ke struktur konkret —
  ini BUKAN kegagalan eksekusi biasa, ini Conflict Resolution
  (§Conflict Resolution) antara Design & UI/UX. Team Lead resolusi dulu
  sebelum eskalasi ke Level 2/fallback biasa.
- **Frontend gagal/blocked** setelah UI/UX selesai: pakai alur fallback
  standar di atas (Level 2 → Team Lead ambil alih) — di titik ini
  masalahnya biasanya teknis (implementasi), bukan lagi soal arah
  kreatif/struktur, jadi tidak perlu balik ke tahap sebelumnya kecuali
  ternyata spec UI/UX-nya sendiri yang tidak bisa diimplementasikan
  (kalau begitu, balik ke Conflict Resolution seperti UI/UX di atas).
- Task kecil yang skip Design/UI/UX (langsung ke Frontend, sesuai SKILL.md
  §1) pakai fallback standar biasa — sub-section ini hanya berlaku kalau
  pipeline penuh dipakai.

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
