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
