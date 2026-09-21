# Lesson: Test-Review-Report WAJIB Pakai Lessons

## Trigger
User mengoreksi agent karena saat disuruh "test > review > report",
agent tidak mempraktikkan knowledge dari lesson file
`001-lighthouse-audit-workflow.md` yang sudah tersimpan. User bilang
"hukum nya wajibb" — artinya penerapan lesson bukan opsional, tapi
keharusan hukum.

## Konteks
- Project: PayPulse Payroll System
- Skill: web-building-team
- Task sebelumnya: UI redesign (top header bar + tab navigation)
- User suruh: "test > review > report"
- Yang terjadi: Agent test HTML structure + fitur preservation + pytest,
  tapi TIDAK jalankan Lighthouse audit (yang ada di lesson 001)
- Masalah: Lesson file sudah ada, tapi tidak dipraktikkan saat test

## Apa yang Dipelajari
1. **Lesson bukan cuma buat dibaca** — Lesson file di
   `references/lessons/` WAJIB dipraktikkan saat task yang relevan.
   Membaca tanpa mempraktikkan = sama dengan tidak punya lesson.
2. **"test > review > report" = wajib pakai lessons** — Saat user
   minta workflow verifikasi pasca-implementasi, agent WAJIB:
   - Load semua lesson file relevan SEBELUM mulai test
   - Praktikkan prosedur dari lesson (termasuk Lighthouse audit)
   - Laporkan di REPORT lesson mana yang diterapkan + hasilnya
3. **Diam-diam skip = pelanggaran** — Kalau lesson tidak diterapkan,
   WAJIB jelaskan kenapa. Tidak boleh diam.
4. **Aturan ditambahkan ke `self-improvement.md`** sebagai section
   `MANDATORY LESSON APPLICATION (Hukum Wajib)` — inserted before
   PRINSIP INTI, tanpa mengganggu konten existing.

## Yang Dikerjakan
1. Added `## MANDATORY LESSON APPLICATION (Hukum Wajib)` section ke
   `references/self-improvement.md` sebelum `## PRINSIP INTI`.
2. Section berisi: aturan wajib (4 rules), scope kapan wajib (5
   kondisi), scope kapan tidak wajib (3 exceptions).
3. Verified: konten existing di self-improvement.md tidak terganggu
   (only additions, no modifications to existing sections).

## Hasil / Bukti
- Section `MANDATORY LESSON APPLICATION` berhasil ditambahkan
- File `self-improvement.md` sekarang punya 11 section (sebelumnya 10)
- Aturan eksplisit: lesson 001 WAJIB dibaca + dipraktikkan untuk task
  yang melibatkan perubahan frontend/template/UI
- Aturan eksplisit: laporan akhir WAJIB mention "Lesson Applied: ..."

## Kesalahan yang Dihindari
- ❌ Membaca lesson tapi tidak mempraktikkan saat test — ini yang
  dioreksi user. Lesson harus DIPRAKTIKKAN, bukan cuma di-load.
- ❌ Diam-diam skip lesson tanpa menjelaskan kenapa — harus selalu
  lapor apakah lesson diterapkan atau tidak (dan kenapa).
- ❌ Menambahkan aturan dengan mengedit/menghapus konten existing —
  aturan baru ditambah sebagai section baru, tidak mengganggu yang ada.

## Reusable untuk
- Setiap kali user minta "test > review > report" atau variasi
- Setiap kali agent menjalankan workflow TEST → REVIEW → FIX →
  VERIFY → REPORT dari SKILL.md §9
- Setiap task yang melibatkan perubahan frontend/template/UI (WAJIB
  Lighthouse audit minimal desktop + mobile navigation)
- Setiap task yang melibatkan perubahan yang punya lesson relevan
  (cek `references/lessons/` untuk lesson yang match)
