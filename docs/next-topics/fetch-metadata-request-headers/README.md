# Fetch Metadata Request Headers

## 1. Official Term

Fetch Metadata Request Headers (`Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User`).

## 2. Definisi Formal

Fetch Metadata Request Headers adalah sekumpulan HTTP request headers yang otomatis ditambahkan browser untuk memberi konteks asal dan tujuan request ke server.

Header ini membantu server menerapkan resource isolation policy berbasis konteks request (same-origin/same-site/cross-site, mode request, destination request), sehingga request berisiko dapat ditolak lebih awal.

Ini adalah mekanisme platform/browser + HTTP, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Saat request datang ke backend, server tidak hanya melihat path dan method, tetapi juga "siapa konteks pemanggilnya".

Dengan Fetch Metadata:

1. Browser menambahkan metadata konteks request.
2. Server membaca header `Sec-Fetch-*`.
3. Server mengizinkan request yang sesuai kontrak aplikasi.
4. Server menolak request cross-site yang tidak seharusnya mengakses resource sensitif.

Model ini efektif sebagai lapisan tambahan untuk mitigasi CSRF dan cross-site abuse.

## 4. Runtime Perspective

- Thread execution:
  - Header dibentuk oleh browser networking stack saat request dipersiapkan (main thread/worker pemanggil tetap tidak mengatur header ini secara manual).
- Queue:
  - Request tetap mengikuti alur fetch/event loop normal; metadata header hanya memperkaya informasi HTTP request.
- Scheduler:
  - Browser scheduler/network stack menentukan kapan request dikirim, lalu menempelkan metadata sesuai konteks origin dan destination.
- Rendering timing:
  - Tidak berdampak langsung ke paint/layout; dampaknya muncul jika server menolak request sehingga data untuk render tidak tersedia.
- Performance implication:
  - Overhead parsing header di server kecil, tetapi dapat mengurangi beban backend dari request cross-site yang tidak valid.
- Memory implication:
  - Dampak memori sangat kecil (beberapa header tambahan per request).
- Security implication:
  - Memberi sinyal kuat ke server untuk membedakan traffic same-site vs cross-site dan menurunkan permukaan serangan CSRF/cross-site probing.

## 5. Kenapa API Ini Ada?

SOP dan CORS tidak otomatis menyelesaikan semua abuse scenario di sisi server, terutama request yang tetap "terkirim" walau response-nya dibatasi browser.

Fetch Metadata ada agar server bisa:

- Menegakkan kebijakan resource isolation berbasis konteks request.
- Menolak request cross-site berisiko sebelum menyentuh logic bisnis sensitif.
- Menambah lapisan pertahanan yang melengkapi CSRF token, SameSite cookie, CORS, dan authz backend.

## 6. Contoh Minimal

```js
// Express.js middleware (contoh konsep)
app.use((req, res, next) => {
  const site = req.get("Sec-Fetch-Site"); // same-origin | same-site | cross-site | none
  const mode = req.get("Sec-Fetch-Mode"); // cors | no-cors | navigate | same-origin | websocket
  const dest = req.get("Sec-Fetch-Dest"); // document | image | script | empty | ...

  // Kebijakan sederhana: tolak request cross-site ke endpoint sensitif non-public.
  const isSensitivePath = req.path.startsWith("/api/private");
  if (isSensitivePath && site === "cross-site") {
    return res.status(403).json({ error: "blocked by fetch metadata policy" });
  }

  // Contoh tambahan: endpoint JSON internal tidak menerima navigation/document load.
  if (isSensitivePath && mode === "navigate" && dest === "document") {
    return res.status(403).json({ error: "invalid request context" });
  }

  next();
});
```

## 7. Common Misconceptions

- "Fetch Metadata menggantikan CSRF token."
  - Koreksi: ini lapisan tambahan; CSRF protection utama tetap perlu.
- "Kalau pakai CORS, Fetch Metadata tidak perlu."
  - Koreksi: CORS mengatur akses baca oleh browser; Fetch Metadata membantu decision di server terhadap konteks request.
- "Header `Sec-Fetch-*` bisa di-set bebas oleh JavaScript."
  - Koreksi: ini forbidden/request-controlled headers dari browser, bukan header biasa yang aman diandalkan dari input aplikasi.

## 8. Pitfall & Best Practices

- Pitfall: membuat policy terlalu ketat tanpa inventaris endpoint.
  - Dampak: traffic valid ikut terblokir.
  - Praktik: mulai dari mode observasi/logging, lalu bertahap enforce.
- Pitfall: hanya memeriksa satu header (`Sec-Fetch-Site`) tanpa konteks endpoint.
  - Dampak: false positive/false negative.
  - Praktik: evaluasi kombinasi `site + mode + dest` sesuai jenis resource.
- Pitfall: menganggap semua browser selalu mengirim header ini.
  - Dampak: kebijakan gagal untuk client tertentu.
  - Praktik: buat fallback policy aman (default deny untuk endpoint sensitif + mekanisme autentikasi/CSRF standar).
- Pitfall: menerapkan rule global tanpa pengecualian endpoint publik.
  - Dampak: integrasi third-party sah bisa rusak.
  - Praktik: klasifikasikan endpoint public/static vs sensitive/internal.

## 9. Prerequisite

- [Fetch Lifecycle](../../fetch-lifecycle/README.md)
- [Security Boundary](../../security-boundary/README.md)
- Dasar HTTP request/response headers
- CSRF dan SameSite cookie fundamentals

## 10. Next Topics

- [Content Security Policy (CSP) fundamentals](../content-security-policy-csp-fundamentals/README.md)
- [Subresource Integrity (SRI) strategy](../subresource-integrity-sri-strategy/README.md)
- [Cross-Origin Isolation (COOP/COEP) and SharedArrayBuffer readiness](../cross-origin-isolation-coop-coep-and-sharedarraybuffer-readiness/README.md)

## References

- Fetch Metadata Request Headers specification: https://www.w3.org/TR/fetch-metadata/
- web.dev (Protect your resources with Fetch Metadata): https://web.dev/articles/fetch-metadata
- Fetch Standard (context for request modes/origins): https://fetch.spec.whatwg.org/
