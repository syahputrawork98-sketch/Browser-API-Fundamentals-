Browser API Fundamental

Long-term reference repository for understanding the Web Platform and Browser APIs at architectural depth.

1. Project Vision

Repository ini adalah dokumentasi fundamental tentang Web Platform & Browser APIs, dibangun secara sistematis dan jangka panjang.

Tujuan utama repository ini:

Memahami bahwa JavaScript (ECMAScript) ≠ Web APIs

Memahami arsitektur Web Platform secara utuh

Membangun mental model runtime browser

Menghilangkan miskonsepsi seperti:

“JavaScript bisa melakukan networking”

“JavaScript punya setTimeout”

“JavaScript punya DOM”

Padahal secara spesifikasi:

Fitur	Didefinisikan oleh
Promise	ECMAScript (TC39)
fetch	Fetch Standard (WHATWG)
DOM	DOM Standard
setTimeout	HTML Standard
Event Loop	HTML Standard
Rendering	Rendering Engine (Blink/WebKit/Gecko)
2. Web Platform Architecture (Mental Model)
ECMAScript (Language Spec - TC39)
        ↓
JavaScript Engine (V8 / SpiderMonkey / JSC)
        ↓
Web Platform (WHATWG Standards)
        ↓
Browser Implementation (Blink / WebKit / Gecko)
        ↓
OS & Networking Stack
Penting:

ECMAScript hanya mendefinisikan bahasa.

Web APIs bukan bagian dari ECMAScript.

Rendering tidak dijalankan oleh JS engine.

Event loop bukan milik ECMAScript.

3. Scope & Boundary
Fokus Utama

DOM Standard

Event System

Fetch API

Storage APIs

URL & History

Timers

Web Workers

Service Workers

Cache Storage

Rendering interaction APIs

Security & Permission model

Page Lifecycle

Event Loop & Scheduling

Tidak Termasuk

Node.js APIs

Framework abstraction (React, Next.js, dll)

Tooling / bundler

Build system

4. Documentation Writing Standard (WAJIB DIIKUTI)

Setiap topik HARUS mengikuti struktur berikut:

1️⃣ Official Term

Gunakan nama resmi dari spesifikasi.

Contoh:

EventTarget

Event propagation

Task

Microtask

Update the rendering

Structured Clone

Same-Origin Policy

2️⃣ Definisi Formal

Definisi teknis yang selaras dengan:

HTML Standard

DOM Standard

Fetch Standard

ECMAScript Spec

MDN (terminology alignment)

3️⃣ Mental Model Sederhana

Penjelasan konseptual ringkas tanpa analogi berlebihan.

4️⃣ Runtime Perspective (WAJIB)

Harus menjawab:

Berjalan di thread mana?

Masuk queue apa?

Siapa yang menjadwalkan?

Kapan rendering terjadi?

Dampak ke performa?

Dampak ke keamanan?

5️⃣ Kenapa API Ini Ada?

Masalah apa yang diselesaikan?

Kenapa tidak menjadi bagian ECMAScript?

6️⃣ Contoh Kode Minimal

Runnable

Fokus ke konsep

Tidak kompleks

7️⃣ Common Misconceptions

Contoh:

❌ "setTimeout adalah bagian dari JavaScript"
✔️ Salah. Itu bagian dari HTML Standard.

8️⃣ Pitfall & Best Practices

Bahas:

Memory leak

Detached DOM

Closure retention

Race condition

Forced synchronous layout

Structured clone cost

Security implication

Browser implementation differences

9️⃣ Prerequisite Concepts

Topik yang harus dipahami sebelumnya.

🔟 Next Topics

Relasi ke topik lanjutan.

5. Core Knowledge Areas

Repository ini wajib membahas secara mendalam:

5.1 Web Platform Spec Landscape

ECMAScript → TC39

HTML Standard → WHATWG

DOM Standard

Fetch Standard

Streams Standard

CSSOM

Infra Standard

Web IDL

5.2 Event Loop & Scheduling Model

Task queue

Microtask queue

Rendering opportunity

requestAnimationFrame

Long task

Frame budget (~16.67ms)

5.3 Rendering Pipeline

Style calculation

Layout (reflow)

Paint

Composite

Main thread blocking

Forced synchronous layout

5.4 Memory & Object Lifetime

Reachability

Garbage collection

Detached DOM nodes

Event listener leak

Structured clone vs Transferable

5.5 Concurrency Model

Single-threaded execution

Web Workers

MessageChannel

Structured clone algorithm

SharedArrayBuffer (cross-origin isolation)

5.6 Networking & Security Model

Same-Origin Policy

CORS

Preflight

Credentials mode

Mixed content

Secure Context

Permissions API

Cross-Origin Isolation

5.7 Document & Page Lifecycle

DOMContentLoaded

load

beforeunload

visibilitychange

Page Lifecycle API

bfcache

5.8 Browser Process Model

Browser process

Renderer process

GPU process

Site isolation

6. Repository Structure (Recommended)
/docs
  /architecture
  /event-loop
  /dom
  /fetch
  /storage
  /rendering
  /security
  /workers
  /lifecycle
  /memory
  /spec-reading
README.md
CONTRIBUTING.md
AI_RULES.md
