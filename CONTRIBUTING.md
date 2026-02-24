# Contributing Guidelines

Terima kasih atas kontribusinya.

Repository ini memiliki standar teknis yang ketat.

Rujukan aturan kontribusi terpusat:

- [Contribution Rules](./RULES.md)

---

## 1. Spec-Aligned Only

Semua penjelasan harus selaras dengan:

- HTML Standard
- DOM Standard
- Fetch Standard
- ECMAScript Spec

---

## 2. Wajib Runtime Perspective

Setiap topik harus membahas:

- Scheduling
- Event loop
- Rendering
- Memory
- Security

---

## 3. Dilarang

- Framework-first explanation
- Node.js specific APIs
- Istilah non-standar
- Simplifikasi berlebihan

---

## 4. Terminology Consistency (WAJIB)

Semua kontribusi harus konsisten dengan glossary resmi:

- [Glossary Istilah Wajib](./docs/glossary/README.md)

Aturan minimum:

- Wajib membedakan `ECMAScript` vs `Web Platform APIs` vs `browser implementation`.
- Dilarang menulis Web APIs sebagai bagian dari JavaScript language.
- Jika ada istilah ambigu, ikuti istilah dan kalimat koreksi standar di glossary.

---

## 5. Writing Quality

- Sistematis
- Presisi
- Tidak santai
- Tidak motivasional
- Fokus arsitektur

---

## 6. Referensi

Jika memungkinkan, sertakan referensi ke spesifikasi resmi.

---

## 7. Markdown Quality Gate (WAJIB)

Sebelum membuka PR, jalankan markdown lint:

```powershell
npx markdownlint-cli2 "**/*.md"
```

Konfigurasi lint ada di file `.markdownlint.json`.

Jika lint gagal, perbaiki dulu sebelum submit kontribusi.

---

## 8. Checklist Validasi 10-Bagian (WAJIB)

Setiap topik API wajib lolos checklist berikut:

- [ ] 1. Official Term
- [ ] 2. Definisi Formal (spec-aligned)
- [ ] 3. Mental Model (presisi, tidak menyesatkan)
- [ ] 4. Runtime Perspective (thread, queue, scheduler, rendering, performance, memory, security)
- [ ] 5. Kenapa API Ini Ada?
- [ ] 6. Contoh Minimal (runnable, fokus konsep)
- [ ] 7. Common Misconceptions (dengan koreksi eksplisit)
- [ ] 8. Pitfall & Best Practices
- [ ] 9. Prerequisite
- [ ] 10. Next Topics

Tambahan validasi terminologi:

- [ ] Tidak menyebut Web APIs sebagai bagian dari JavaScript language
- [ ] Memisahkan jelas: ECMAScript vs Web Platform APIs vs browser implementation
