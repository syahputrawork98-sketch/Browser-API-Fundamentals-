# Contribution Rules

File ini adalah kontrak instruksi untuk AI yang membantu menulis repository ini.

---

## 1. Arsitektur Wajib Dipatuhi

AI harus selalu membedakan:

- ECMAScript (Language)
- JavaScript Engine
- Web Platform APIs
- Browser implementation

Dilarang menyebut Web APIs sebagai bagian dari JavaScript.

---

## 2. Terminology Enforcement

Jika menemukan istilah berikut:

❌ "JavaScript event loop"  
✔ Event loop (HTML Standard)

❌ "JavaScript fetch"  
✔ Fetch API (Fetch Standard)

❌ "JavaScript timer"  
✔ Timers (HTML Standard)

AI wajib mengoreksi secara eksplisit.

---

## 3. Struktur Jawaban Wajib

Setiap penjelasan API harus mengikuti:

1. Official Term  
2. Definisi Formal  
3. Mental Model  
4. Runtime Perspective  
5. Kenapa Ada  
6. Contoh Minimal  
7. Common Misconceptions  
8. Pitfall & Best Practices  
9. Prerequisite  
10. Next Topics  

---

## 4. Runtime Awareness

AI wajib menjelaskan:

- Thread execution
- Task vs microtask
- Scheduling mechanism
- Rendering timing
- Memory implication
- Security boundary

---

## 5. Dilarang

- Oversimplifikasi yang merusak akurasi
- Lompat ke framework
- Menggunakan analogi menyesatkan
- Copy paste definisi tanpa pemahaman
