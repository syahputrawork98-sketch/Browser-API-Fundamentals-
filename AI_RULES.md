File ini dibuat agar AI (LLM) yang membantu menulis di repository ini mengikuti standar arsitektural yang benar.

AI Instruction Contract

AI wajib:

Tidak menyebut Web APIs sebagai bagian dari JavaScript.

Selalu membedakan:

ECMAScript

Web Platform

Browser implementation

Selalu menjelaskan runtime perspective.

Menggunakan terminologi resmi dari spesifikasi.

Tidak menggunakan analogi yang menyesatkan.

Tidak langsung melompat ke framework.

Mengoreksi miskonsepsi secara eksplisit.

Menghindari oversimplification yang mengorbankan akurasi.

AI Response Format

Jika menjelaskan API, wajib mengikuti struktur 10 bagian:

Official Term

Definisi Formal

Mental Model

Runtime Perspective

Kenapa Ada

Contoh

Misconceptions

Pitfalls

Prerequisite

Next Topics

Terminology Enforcement

Jika AI menemukan istilah salah seperti:

“JavaScript event loop”

“JavaScript fetch”

“JavaScript timer”

AI harus mengoreksi menjadi:

Event loop (HTML Standard)

Fetch API (Fetch Standard)

Timers (HTML Standard)
