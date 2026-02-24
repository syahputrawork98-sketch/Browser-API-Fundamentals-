# Service Worker Lifecycle and Cache Storage

## 1. Official Term

Service Worker lifecycle (`register`, `install`, `activate`, `fetch` event) and Cache Storage API.

## 2. Definisi Formal

Service Worker adalah worker khusus yang bertindak sebagai programmable network proxy untuk origin/scope tertentu, berjalan terpisah dari halaman, dan dapat menangani event seperti `install`, `activate`, serta `fetch`.

Cache Storage adalah Web Platform storage API untuk menyimpan pasangan `Request`/`Response` yang dapat diakses dari Service Worker maupun window context (sesuai kebijakan). Keduanya bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Alur utama:

1. Halaman mendaftarkan service worker script.
2. Browser menginstal worker versi baru (`install`).
3. Worker diaktifkan (`activate`) saat siap mengambil alih klien.
4. Pada request dalam scope, worker dapat intercept `fetch` dan memutuskan strategi (cache-first, network-first, stale-while-revalidate, dsb).
5. Cache Storage menyimpan aset/data response untuk reuse dan offline path.

Service Worker adalah lapisan kontrol request-response, bukan thread yang terus hidup tanpa henti.

## 4. Runtime Perspective

- Thread execution:
  - Service Worker berjalan di worker context terpisah dari main thread dokumen.
  - Tidak memiliki akses DOM langsung.
- Queue:
  - Event lifecycle (`install`, `activate`, `fetch`, `message`) masuk ke event loop Service Worker sebagai task.
  - Promise reactions di handler event berjalan sebagai microtask dalam context Service Worker.
- Scheduler:
  - Browser menentukan kapan Service Worker process dibangunkan/disuspend berdasarkan event dan resource policy.
  - Update lifecycle dikontrol oleh browser agar transisi versi aman.
- Rendering timing:
  - Operasi di Service Worker tidak langsung memblokir rendering thread.
  - Namun latensi di `fetch` handler dapat menunda data yang dibutuhkan UI.
- Performance implication:
  - Strategi cache tepat dapat menurunkan latency dan meningkatkan resilience offline.
  - Strategi salah (cache miss berulang, blocking fetch chain) dapat memperlambat navigasi/resource loading.
- Memory implication:
  - Cache Storage besar meningkatkan jejak storage dan metadata in-memory saat indexing/query.
  - Menyimpan response berlebihan tanpa eviction policy aplikasi menambah tekanan quota.
- Security implication:
  - Service Worker hanya tersedia pada secure context (HTTPS, kecuali localhost).
  - Scope registration membatasi area kontrol request; desain scope yang terlalu luas meningkatkan blast radius saat bug/kompromi.

## 5. Kenapa API Ini Ada?

Web membutuhkan mekanisme:

- Offline capability yang terkontrol.
- Caching strategi tingkat aplikasi.
- Interception request tanpa extension/proxy eksternal.

Service Worker + Cache Storage menyediakan fondasi Progressive Web App dengan kontrol network behavior per-origin.

## 6. Contoh Minimal

```js
// sw.js
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open("app-v1").then((cache) => cache.addAll(["/", "/index.css", "/main.js"]))
  );
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => cached || fetch(event.request))
  );
});
```

```js
// main thread
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("/sw.js");
}
```

## 7. Common Misconceptions

- "Service Worker selalu aktif seperti daemon."
  - Koreksi: lifecycle dikelola browser; worker bisa dihentikan saat idle.
- "Cache Storage sama dengan HTTP cache browser."
  - Koreksi: Cache Storage adalah API terpisah dengan kontrol eksplisit di aplikasi.
- "Service Worker bisa akses DOM halaman."
  - Koreksi: service worker tidak punya akses DOM langsung.

## 8. Pitfall & Best Practices

- Pitfall: tidak mengelola versi cache.
  - Dampak: aset stale menumpuk dan perilaku inkonsisten.
  - Praktik: versioned cache key + cleanup cache lama di `activate`.
- Pitfall: fetch handler yang tidak punya fallback network/error jelas.
  - Dampak: request bisa gagal tanpa recovery.
  - Praktik: tetapkan strategi per-resource type dan fallback deterministic.
- Pitfall: scope terlalu luas tanpa alasan.
  - Dampak: bug/service worker error memengaruhi area aplikasi lebih besar.
  - Praktik: batasi scope sesuai kebutuhan arsitektur.

## 9. Prerequisite

- Fetch lifecycle fundamentals.
- Storage model and quota.
- Web Workers basics.

## 10. Next Topics

- [Cache invalidation strategy design](../next-topics/cache-invalidation-strategy-design/README.md)
- [Background Sync and periodic sync](../next-topics/background-sync-and-periodic-sync/README.md)
- [Security Boundary](../security-boundary/README.md)

## References

- Service Workers specification: https://w3c.github.io/ServiceWorker/
- Cache API (Service Workers spec): https://w3c.github.io/ServiceWorker/#cache-interface
- HTML Standard (workers integration): https://html.spec.whatwg.org/multipage/workers.html


