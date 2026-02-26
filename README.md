📘 Browser API Fundamental V1

> Long-term reference repository for understanding the Web Platform & Browser APIs at architectural depth.

---

## Documentation Index

- [Documentation Map](./docs/README.md)
- [Contribution Rules](./RULES.md)
- [Next Topics Index](./docs/next-topics/README.md)
- [Glossary Istilah Wajib](./docs/glossary/README.md)
- [Cross-Topic Architecture Diagram](./docs/cross-topic-architecture/README.md)

---

## Official Learning Path (Urutan Baca Wajib)

1. Foundation Runtime -> [Event Loop](./docs/event-loop/README.md) -> [Timers](./docs/timers/README.md) -> [Microtask and Promise Jobs Integration](./docs/microtask-and-promise-jobs/README.md) -> [requestAnimationFrame](./docs/request-animation-frame/README.md) -> [Rendering Pipeline Interaction](./docs/rendering-pipeline-interaction/README.md)
2. DOM and Interaction -> [DOM Standard Fundamentals](./docs/dom-standard-fundamentals/README.md) -> [Event Propagation Model](./docs/event-propagation-model/README.md)
3. Networking and Security Boundary -> [Fetch Lifecycle](./docs/fetch-lifecycle/README.md) -> [Security Boundary](./docs/security-boundary/README.md)
4. Persistence and Navigation -> [Storage Model and Quota](./docs/storage-model-and-quota/README.md) -> [URL and History Model](./docs/url-and-history-model/README.md) -> [Document Lifecycle Deep Dive](./docs/document-lifecycle-deep-dive/README.md)
5. Concurrency and Platform Architecture -> [Web Workers and Message Passing](./docs/web-workers-and-message-passing/README.md) -> [Service Worker Lifecycle and Cache Storage](./docs/service-worker-lifecycle-and-cache-storage/README.md) -> [Browser Process Model Deep Dive](./docs/browser-process-model-deep-dive/README.md)
6. Advanced Track -> [Long Tasks and PerformanceObserver](./docs/long-tasks-and-performance-observer/README.md) -> [Streams Deep Dive](./docs/streams-deep-dive/README.md) -> [CSP, Trusted Types, and COOP/COEP](./docs/csp-trusted-types-coop-coep/README.md)

---

## 1. Project Vision

Repository ini adalah dokumentasi fundamental tentang **Web Platform & Browser APIs**, dibangun secara sistematis dan jangka panjang.

Tujuan utama:

- Memahami bahwa **ECMAScript (JavaScript language)** ≠ **Web APIs**
- Membangun mental model arsitektur Web Platform
- Memahami runtime model browser
- Menghilangkan miskonsepsi seperti:
  - "JavaScript punya fetch"
  - "JavaScript punya DOM"
  - "JavaScript punya setTimeout"
  - "Event loop adalah bagian dari JavaScript"

Padahal secara spesifikasi:

| Fitur | Didefinisikan oleh |
|--------|-------------------|
| Promise | ECMAScript (TC39) |
| Array / Map / Set | ECMAScript |
| fetch | Fetch Standard (WHATWG) |
| DOM | DOM Standard |
| setTimeout | HTML Standard |
| Event Loop | HTML Standard |
| Rendering | Rendering Engine |

---

## 2. Web Platform Architecture (Mental Model)


ECMAScript (Language Spec - TC39)
↓
JavaScript Engine (V8 / SpiderMonkey / JSC)
↓
Web Platform (WHATWG Standards)
↓
Browser Engine (Blink / WebKit / Gecko)
↓
OS & Networking Stack


Poin penting:

- ECMAScript hanya mendefinisikan bahasa.
- Web APIs bukan bagian dari ECMAScript.
- Rendering tidak dijalankan oleh JavaScript engine.
- Event loop bukan bagian dari ECMAScript.

---

## 3. Scope & Boundary

### Fokus

- DOM Standard
- Event System
- Fetch API
- Storage APIs
- URL & History
- Timers
- Web Workers
- Service Workers
- Cache Storage
- Rendering interaction APIs
- Security model
- Page lifecycle
- Event loop & scheduling

### Tidak Termasuk

- Node.js APIs
- Framework abstraction (React, Next, dll)
- Tooling / bundler
- Build system

---

## 4. Documentation Standard (WAJIB)

Setiap topik HARUS mengikuti struktur berikut:

### 1️⃣ Official Term

Gunakan istilah resmi spesifikasi.

### 2️⃣ Definisi Formal

Definisi teknis selaras dengan HTML / DOM / Fetch / ECMAScript spec.

### 3️⃣ Mental Model

Penjelasan konseptual ringkas dan presisi.

### 4️⃣ Runtime Perspective

Wajib menjawab:

- Berjalan di thread mana?
- Masuk queue apa?
- Siapa yang menjadwalkan?
- Kapan rendering terjadi?
- Dampak performa?
- Dampak keamanan?

### 5️⃣ Kenapa API Ini Ada?

Masalah apa yang diselesaikan?  
Kenapa bukan bagian dari ECMAScript?

### 6️⃣ Contoh Minimal

Kode runnable dan fokus konsep.

### 7️⃣ Common Misconceptions

Koreksi miskonsepsi secara eksplisit.

### 8️⃣ Pitfall & Best Practices

Bahas:

- Memory leak
- Detached DOM
- Closure retention
- Race condition
- Forced synchronous layout
- Structured clone cost
- Security implication

### 9️⃣ Prerequisite

Konsep yang harus dipahami sebelumnya.

### 🔟 Next Topics

Topik lanjutan yang relevan.

---

## 5. Core Knowledge Areas

Repository ini wajib membahas:

### Web Platform Spec Landscape

- ECMAScript (TC39)
- HTML Standard (WHATWG)
- DOM Standard
- Fetch Standard
- Streams Standard
- CSSOM
- Infra Standard
- Web IDL

### Event Loop & Scheduling

- Task
- Microtask
- Rendering opportunity
- requestAnimationFrame
- Long task
- Frame budget (~16.67ms)

### Rendering Pipeline

- Style calculation
- Layout
- Paint
- Composite
- Main thread blocking

### Memory Model

- Reachability
- Garbage collection
- Detached DOM nodes
- Structured clone
- Transferable objects

### Concurrency Model

- Single-threaded JS execution
- Web Workers
- MessageChannel
- SharedArrayBuffer (cross-origin isolation)

### Networking & Security

- Same-Origin Policy
- CORS
- Preflight
- Credentials mode
- Mixed content
- Secure Context
- Permissions API

### Document Lifecycle

- DOMContentLoaded
- load
- beforeunload
- visibilitychange
- bfcache

### Browser Process Model

- Browser process
- Renderer process
- GPU process
- Site isolation

---

## 6. Ultimate Goal

Setelah memahami repository ini, pembaca harus mampu menjelaskan:

- Bagaimana event dipropagasikan
- Bagaimana task & microtask dijadwalkan
- Kapan rendering terjadi
- Bagaimana networking diproses
- Bagaimana storage bekerja
- Bagaimana security boundary membatasi akses
- Bagaimana concurrency dikontrol

Tujuan akhirnya:

Tidak hanya tahu *apa yang terjadi*,  
tetapi tahu *mengapa dan bagaimana browser melakukannya*.

