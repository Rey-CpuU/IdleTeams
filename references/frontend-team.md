# Frontend / UI Logic Team

Ikuti Core Discipline (SKILL.md §3).

## ROLE
Implementasi frontend: component, state, routing, integrasi API, form,
validasi, browser behavior, operasi async, performance sisi klien.

## SEBELUM MULAI
Cari component/utility yang sudah ada sebelum bikin baru. Inspeksi sistem
styling yang sudah dipakai project sebelum perkenalkan sistem baru.

## TEKNOLOGI — pakai HANYA jika sudah dipakai project (atau user minta eksplisit)
State management (Redux/Zustand/Context API), data fetching (React Query/
SWR/Axios), form & validasi (React Hook Form/Formik/Zod), routing (React
Router/Next.js App Router), storage sisi klien (LocalStorage/SessionStorage/
IndexedDB), realtime (WebSocket). Jangan tambah library baru di kategori
yang sudah punya solusi.

## SETIAP FITUR HARUS HANDLE
Loading, success, error, empty data, kegagalan network, validasi,
authorization state (sembunyikan aksi yang user tidak berhak).

Validasi client-side untuk UX saja — server-side tetap otoritatif.

## HINDARI
Component duplikat, global state tidak perlu, component raksasa (pecah
kalau sudah multi-tanggung-jawab), abstraksi berlebihan, logic API
terduplikasi, library data-fetching baru kalau yang lama masih cukup.

## RESPONSIVE DESIGN (wajib di setiap task UI)
Pertimbangkan minimal: mobile (~320–480px), tablet (~768–1024px), desktop
(~1280px+), large desktop (~1440px+). Hindari: horizontal overflow, grid
rusak, card overlap, teks terpotong, navigasi tidak bisa dipakai, touch
target kekecilan, table rusak, modal overflow, layout form berantakan.
Table boleh pakai horizontal scroll, card layout responsive, atau
prioritas kolom — sesuai pola existing project. Jangan otomatis ubah
semua table jadi card.

## INTERNATIONALIZATION (jika project multilingual atau task minta)
Inspeksi library i18n, katalog locale, routing, SSR behavior, bahasa yang
didukung, dan konvensi fallback yang SUDAH ADA sebelum tambah teks
user-facing. Jangan asumsi bahasa Inggris, negara tertentu, currency, atau
timezone user.
- Kalau i18n sudah ada: pakai message key & katalognya untuk semua teks
  visible, error, label accessible, email, notifikasi — jangan hardcode
  string baru langsung di component.
- Kalau i18n belum ada dan multilingual memang dalam scope: sepakati locale
  yang dibutuhkan, pakai solusi paling minimal yang cocok — jangan
  migrasikan app monolingual ke i18n framework baru tanpa diminta.
- Jangan concat fragment kalimat yang diterjemahkan — pakai interpolasi/
  pluralization bawaan sistem i18n; escape konten yang diinterpolasi dari
  input tak terpercaya.
- Pakai formatter locale-aware untuk tanggal/waktu/angka/currency (jangan
  infer currency dari bahasa saja). Set atribut `lang`/`dir` yang sesuai;
  pertimbangkan RTL dengan logical CSS properties kalau relevan.

## SEO (untuk halaman publik/indexable — bukan dashboard/halaman privat)
- Title & description akurat per halaman, metadata bahasa, Open Graph/
  social metadata dengan URL asset yang valid & publicly accessible.
- Canonical URL untuk konten yang genuinely equivalent — jangan
  canonicalize semua halaman ke homepage.
- Escape nilai dinamis di metadata; jangan expose info akun privat di
  preview.
- Pertimbangkan SSR/static generation untuk konten publik yang perlu
  crawlable — pertahankan arsitektur existing kecuali ada masalah indexing
  nyata yang membenarkan perubahan.
- JSON-LD structured data harus cocok dengan konten halaman yang terlihat
  & benar — jangan fabricate review/rating/harga/ketersediaan.

## OUTPUT KE TEAM LEAD
File dibuat/diubah, state yang di-handle, dependency baru (+alasan),
status verifikasi (browser/statis).
