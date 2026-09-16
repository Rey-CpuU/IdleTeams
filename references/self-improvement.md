# Self-Improvement: Reusable Skills & Persistent Memory

Ikuti Core Discipline (SKILL.md §3). Self-improvement bersifat ADDITIVE —
tidak pernah menghapus, mengganti, atau melemahkan tim/workflow/aturan
security/verifikasi/proteksi project yang sudah ada.

## KAPAN BUAT SKILL BARU
Kalau agent menemukan prosedur yang: berulang berguna, teknis reliable,
compatible dengan project, generalizable, aman dipakai ulang, dipahami
jelas, dan sudah terverifikasi berhasil — boleh disimpan jadi reusable
skill. Contoh: prosedur debugging berulang, pola implementasi
framework-spesifik, workflow project-spesifik, prosedur deployment/
testing/security-hardening/migration yang sudah teruji.

**Jangan** ubah guess yang belum diverifikasi, workaround sementara, atau
eksperimen gagal jadi skill terpercaya.

## KUALITAS SKILL
Skill yang disimpan idealnya mencatat: nama, tujuan, kapan dipakai/tidak
dipakai, precondition, environment/tool/dependency yang dibutuhkan,
prosedur, hasil yang diharapkan, cara verifikasi, pertimbangan security,
limitasi diketahui, kondisi kegagalan, asumsi project/framework/versi
spesifik. **Jangan pernah** simpan password, API key, token, credential,
atau data sensitif lain di dalam skill.

## REUSE SKILL
Sebelum selesaikan masalah berulang dari nol: cek apakah skill relevan
sudah ada → load kalau ada → cek asumsinya masih cocok dengan project saat
ini → pakai hanya kalau compatible → verifikasi hasil → update skill kalau
memang ada perbaikan material. Jangan pakai skill lama secara blind —
framework/dependency/arsitektur/OS/security/versi bisa berubah, selalu
verifikasi kompatibilitas dulu.

## EVOLUSI SKILL
Kalau skill tersimpan ternyata incomplete/outdated/insecure/incompatible:
boleh diperbaiki. TAPI jangan diam-diam ganti skill reliable dengan
prosedur belum terverifikasi — inspeksi skill lama → identifikasi
limitasi → tentukan perbaikan → verifikasi → update setelah cukup
terverifikasi → pertahankan lesson lama yang masih valid.

## PROJECT-SPESIFIK vs GENERALIZED
Skill boleh dibuat project-spesifik (konvensi arsitektur/component/API/
naming project ini) — harus jelas ditandai project-spesifik, jangan
diasumsikan berlaku ke project lain. Kalau lesson bisa digeneralisasi
aman, tulis prinsipnya, bukan detail implementasi project-spesifik.
Contoh: "pakai component TicketCard ini" ≠ skill global; "saat bikin
ticket-list interface, inspeksi component card existing & ikuti pola
project" = lesson yang bisa digeneralisasi.

## SELF-IMPROVEMENT SAFETY
Self-improvement tidak boleh pernah override: inspect-before-modify,
anti-hallucination, jangan asumsi stack, proteksi fitur existing, minimal
change, ikut arsitektur nyata, jaga security, verifikasi implementasi,
hormati requirement user, jangan expose secret, jangan aksi destruktif
tanpa otorisasi, jangan ubah sistem tak terkait, jangan klaim hasil belum
terverifikasi. Kalau nemu lesson reusable saat implementasi fitur X:
catat terpisah — jangan sekalian modifikasi fitur Y/Z yang tak diminta.

## FAILURE HANDLING
Kegagalan boleh menghasilkan lesson berguna, tapi prosedur gagal TIDAK
otomatis jadi skill terpercaya. Boleh dicatat: apa yang dicoba, kenapa
gagal, error yang terlihat, environment, pendekatan yang benar (kalau
ketemu). Jangan simpan prosedur salah sebagai solusi rekomendasi.

## PERSISTENT MEMORY (jika Hermes environment punya backend memory)
Boleh dipakai untuk: preferensi coding/UI/workflow user, konvensi
project, environment yang diketahui, prosedur troubleshooting
terverifikasi, keputusan arsitektur penting, lesson dari task sebelumnya.

**Prioritas saat konflik**: instruksi eksplisit user saat ini > realita
project saat ini > config project saat ini > environment terverifikasi
saat ini > persistent memory relevan > reusable skill umum > asumsi umum.
Realita project & instruksi user SELALU menang atas memory lama.

**Evidence-based**: jangan simpan tebakan sebagai fakta — bedakan info
yang eksplisit diberikan user, hasil observasi project, fakta teknis
terverifikasi, preferensi yang disimpulkan, kondisi sementara.

**Tool awareness**: sebelum pakai memory, tentukan apakah backend memory
benar-benar tersedia & operasi apa yang didukung — jangan mengarang
command/API memory. Kalau tidak tersedia, lanjut normal pakai context
sesi ini saja, jangan berpura-pura ada cross-session memory.

**Security & privacy**: jangan pernah simpan password/API key/token/
credential/secret `.env` di memory. Simpan hanya info yang relevan/
berguna untuk kerja ke depan — hindari detail personal tak perlu.

**Verifikasi ulang**: saat retrieve info dari memory yang menyangkut fakta
teknis (mis. "project pakai framework X"), verifikasi ulang ke environment
saat ini sebelum dipakai — kalau project sudah berubah, pakai realita
saat ini, bukan memory lama.

## LARANGAN KLAIM PALSU
Jangan pernah klaim: "saya belajar ini permanen" kecuali persistent
storage benar-benar berhasil; "saya simpan sebagai skill" kecuali skill
benar-benar dibuat/diupdate; "ini akan diingat selamanya" kecuali
environment benar-benar mendukung persistence itu. Kalau persistence
tidak tersedia, nyatakan limitasinya apa adanya.

## AUTO-LEARNING PROTOCOL (Self-Evolving)
Setiap kali skill web-building-team dipanggil dan menyelesaikan task,
agent WAJIB belajar dari apa yang user suruh lakukan dan hasilnya. Ini
bukan opsional — ini adalah siklus evolusi yang membuat tim makin pintar
setiap session.

### Kapan Auto-Learn Triggered
Auto-learning aktif setelah salah satu kondisi berikut:
1. User mengajarkan teknik/tool baru (misal: "jalankan lighthouse",
   "pakai gzip", "test dengan mode snapshot").
2. Agent menemukan solusi ke bug/issue yang belum pernah di-dokumentasi.
3. User mengoreksi pendekatan agent (misal: "jangan pakai content-visibility
   begini, pakai cara itu").
4. Workflow baru terbentuk yang terbukti efektif dan reusable.
5. Hasil test/audit menghasilkan pattern fix yang bisa digeneralisasi.

### Cara Menyimpan Lesson
1. **Buat file di `references/lessons/`** — direktori khusus untuk
   lesson-lesson yang dipelajari. Nama file: `NNN-short-description.md`
   (misal: `001-lighthouse-audit-workflow.md`).
2. **Format lesson file**:
   ```markdown
   # Lesson: <judul singkat>

   ## Trigger
   <kapan/pada task apa lesson ini ditemukan>

   ## Konteks
   <situasi, project, stack, environment>

   ## Apa yang Dipelajari
   <prinsip, prosedur, fix pattern, atau workflow yang terverifikasi>

   ## Yang Dikerjakan
   <langkah-langkah aktual yang dilakukan dan berhasil>

   ## Hasil / Bukti
   <metrik, skor, test result — bukti bahwa ini bekerja>

   ## Kesalahan yang Dihindari
   <jika ada: pendekatan yang dicoba tapi gagal, dan kenapa>

   ## Reusable untuk
   <kondisi/project type di mana lesson ini berlaku>
   ```

3. **Cross-reference TANPA mengganggu file lain**: Lesson file bersifat
   STANDALONE — agent membaca direktori `lessons/` di awal setiap
   invocation untuk load semua lesson. TIDAK PERLU memodifikasi
   `qa-team.md`, `performance-team.md`, atau file reference lainnya.
   File-file lesson saling melengkapi, bukan menimpa.

   Jika lesson memerlukan integrasi dengan reference file tertentu:
   - Boleh BUAT file baru di `references/lessons/` yang me-reference
     file lain (misal: "lihat qa-team.md §LIGHTHOUSE AUDIT").
   - **DILARANG** mengedit/menambah/menghapus konten dari reference
     file yang sudah ada (qa-team.md, performance-team.md, dll.) untuk
     menyisipkan link ke lesson. Lesson file yang me-reference, bukan
     sebaliknya.

4. **Lesson hanya disimpan jika TERVERIFIKASI**:
   - Fix/prosedur benar-benar berhasil (ada bukti: test pass, skor
     naik, bug resolved).
   - Bukan tebakan, bukan workaround sementara, bukan eksperimen gagal.
   - Jika pendekatan dicoba dan gagal → catat di section "Kesalahan
     yang Dihindari" di lesson file yang sama atau lesson terkait.

### Load Lessons di Awal Invocation
Setiap kali web-building-team di-spawn, agent harus:
1. Cek apakah direktori `references/lessons/` ada.
2. Jika ada, list semua file `.md` di dalamnya.
3. Baca setiap lesson file (atau minimal: judul + "Reusable untuk"
   section untuk quick-scan relevansi).
4. Tentukan lesson mana yang relevan dengan task saat ini.
5. Load full content lesson yang relevan sebelum mulai kerja.

Ini memastikan knowledge dari session sebelumnya langsung tersedia
tanpa perlu user mengulang instruksi.

### Evolusi Lesson
- Lesson bisa di-update jika ditemukan informasi baru yang melengkapi.
- Lesson bisa di-mark `[DEPRECATED]` di judul jika sudah tidak relevan
  (misal: framework diganti, API berubah).
- Jangan hapus lesson — tandai deprecated supaya history tetap ada.
- Lesson baru yang kontradiksi dengan lesson lama: buat lesson baru
  dengan catatan "supersedes lesson NNN".

### Yang TIDAK Boleh Dilakukan
- ❌ Edit/modify/hapus konten dari file reference yang sudah ada
  (qa-team.md, performance-team.md, backend-team.md, dll.) untuk
  menyisipkan link atau referensi ke lesson.
- ❌ Simpan password, API key, token, credential di lesson file.
- ❌ Simpan tebakan/workaround sebagai lesson yang terverifikasi.
- ❌ Buat lesson yang duplikat — cek dulu apakah lesson serupa sudah
  ada sebelum buat baru.
- ❌ Klaim "saya ingat dari session sebelumnya" kecuali lesson benar-
  benar ada di `references/lessons/` dan dibaca ulang.

## PRINSIP INTI (tidak berubah walau ada self-improvement)
INSPECT FIRST → UNDERSTAND → PLAN → DELEGATE → IMPLEMENT → TEST → REVIEW →
FIX → VERIFY → LEARN → SIMPAN PENGETAHUAN TERVERIFIKASI → IMPROVE FUTURE
EXECUTION → REPORT. Self-improvement membuat tim makin capable tanpa
membuatnya kurang predictable, kurang aman, atau kurang menghormati fungsi
project yang sudah ada.
