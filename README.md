# Deciq — First-Principles Problem Breakdown

**Deciq** adalah tool AI yang membedah suatu masalah menjadi bagian-bagian terdalamnya memakai
*first-principles thinking* ala Elon Musk: bongkar sampai ke fakta paling dasar
(bukan analogi, bukan "karena biasanya begitu"), hapus yang tidak esensial,
sederhanakan, lalu **bangun ulang solusinya dari nol**.

Aplikasi web statis satu file (`index.html`) — tanpa build step, tanpa backend.
Bisa langsung dibuka di browser, di-host di Vercel / GitHub Pages / Netlify.

## Cara pakai

1. Buka `index.html` di browser (atau deploy).
2. Klik **⚙️ Pengaturan** (pojok kanan atas), isi:
   - **URL Endpoint** — API yang kompatibel dengan *OpenAI Chat Completions*,
     mis. `https://api.openai.com/v1/chat/completions`
     atau OpenRouter `https://openrouter.ai/api/v1/chat/completions`,
     atau server lokal (LM Studio / Ollama via proxy yang mendukung CORS).
   - **API Key** — disimpan hanya di `localStorage` browser, tidak dikirim ke mana pun
     selain endpoint di atas. Kosongkan jika endpoint tidak butuh key.
   - **Nama Model** — mis. `gpt-4o-mini`, `openai/gpt-4o-mini`, `llama-3.1-8b`.
3. Klik **Test Koneksi** untuk memastikan endpoint merespons.
4. Isi **Masalah** (wajib) + konteks: untuk siapa, hasil yang diinginkan,
   yang sudah dicoba, kendala.
5. Klik **⚡ Breakdown dengan First Principles**.

Jika rumusan masalah terlalu kabur, Deciq akan meminta klarifikasi singkat
dulu sebelum menganalisis.

## Alur 10 langkah output

1. **Pahami Masalah** — problem reframe + visualisasi pohon masalah tiap layer sampai akar.
2. **Audit Asumsi** — tabel fakta vs opini + *Assumption Risk Score* (1–10).
3. **Requirement Interrogation** — tiap requirement ditanya: pemilik, tujuan, bukti, risiko jika dihapus.
4. **Fakta Dasar** — dekomposisi (tujuan, constraint, komponen, bukti, alternatif) + rantai **5 Whys**
   yang berhenti saat penjelasan benar-benar fundamental (bukan otomatis di angka 5).
5. **Daftar Hapus** — kandidat hapus + alasan, risiko, *Deletion Score*, rekomendasi
   (hapus sekarang / opsional / tunda ke v2 / pertahankan).
6. **Solusi Sederhana** — before → after untuk semua yang tidak dihapus.
7. **Bangun Ulang** — solusi baru dari *fundamental truths*, bebas pola industri lama + alasan kenapa berbeda.
8. **Otomasi dan Percepatan** — evaluasi *accelerate* dulu, *automate* **selalu paling akhir**.
9. **Rekomendasi MVP** — core user, core problem, core promise, core input, core output, fitur yang ditunda.
10. **Validasi** — rencana eksperimen (hipotesis, metode, metrik, durasi) untuk asumsi paling berisiko.

Hasil bisa diunduh sebagai laporan Markdown atau dicetak ke PDF.

## Voice input (mic per field)

Setiap field input punya tombol **🎤**. Klik untuk merekam suara, klik lagi
untuk berhenti — hasil rekaman otomatis ditranskripsi dan dimasukkan ke field
(ditambahkan di akhir jika field sudah berisi teks).

- Transkripsi memakai **Groq STT** (Whisper, `whisper-large-v3-turbo` default),
  dengan `language: id` untuk akurasi Bahasa Indonesia.
- Isi **Groq API Key** di ⚙️ Pengaturan → bagian "Groq STT" (gratis di
  `console.groq.com`). Tanpa key, tombol mic akan membuka halaman pengaturan.
- Browser harus mendukung `MediaRecorder` + izin mikrofon (Chrome/Edge/Safari
  modern OK).

## Struktur repo

```
deciq/
├── index.html   # seluruh aplikasi (HTML + CSS + JS satu file)
└── README.md
```

## Catatan teknis

- Request memakai format `POST {endpoint}` dengan body `{model, messages, temperature, max_tokens}`
  dan header `Authorization: Bearer <api-key>` (jika key diisi).
- Respons AI wajib JSON sesuai skema di system prompt; parser toleran terhadap
  code fence markdown.

## Batasan

- Membutuhkan endpoint AI yang bisa dijangkau browser (perhatikan CORS untuk server lokal).
- Kualitas breakdown mengikuti kualitas model & kejelasan input.

## Database Supabase (login + riwayat)

Aplikasi memakai **Supabase Auth** dengan login **username + password**
(username dipetakan otomatis ke email internal `username@deciq.internal`)
dan menyimpan tiap hasil breakdown ke tabel `analyses`. Anon key sudah
tertanam di `index.html` (public by design — datanya dilindungi Row Level
Security per user).

### Setup sekali saja (di dashboard Supabase)

1. Buka **SQL Editor → New query**, jalankan:
   ```sql
   create table if not exists public.analyses (
     id uuid primary key default gen_random_uuid(),
     user_id uuid not null references auth.users(id) on delete cascade,
     created_at timestamptz not null default now(),
     problem text not null,
     target_user text,
     desired_outcome text,
     current_solution text,
     "constraint" text,
     result jsonb not null
   );

   alter table public.analyses enable row level security;

   drop policy if exists "Users manage own analyses" on public.analyses;
   create policy "Users manage own analyses"
     on public.analyses for all
     using (auth.uid() = user_id)
     with check (auth.uid() = user_id);
   ```
   Tabel kedua — `user_settings` — untuk sinkronisasi kredensial
   (endpoint, API key, nama model AI + Groq) antar perangkat:
   ```sql
   create table if not exists public.user_settings (
     user_id uuid primary key references auth.users(id) on delete cascade,
     updated_at timestamptz not null default now(),
     ai_endpoint text,
     ai_apikey text,
     ai_model text,
     groq_key text,
     groq_model text
   );

   alter table public.user_settings enable row level security;

   drop policy if exists "Users manage own settings" on public.user_settings;
   create policy "Users manage own settings"
     on public.user_settings for all
     using (auth.uid() = user_id)
     with check (auth.uid() = user_id);
   ```
2. **Authentication → Providers → Email**: pastikan aktif, dan **matikan
   *Confirm email*** (wajib — username login tidak punya inbox untuk verifikasi).
3. (Opsional) Di **Authentication → Settings**, matikan *Allow new users to sign up*
   jika hanya kamu yang boleh punya akun — buat akunmu sekali via form Daftar di aplikasi.

Setelah itu buka aplikasi, daftar/masuk dengan username, dan tiap hasil
breakdown otomatis tersimpan ke menu **🕘 Riwayat** (buka ulang / hapus per item).
Pengaturan kredensial (endpoint, API key, model AI & Groq) juga otomatis
tersinkron ke cloud: ditarik saat login, dikirim tiap klik **Simpan** —
jadi tetap sama di semua perangkat.
