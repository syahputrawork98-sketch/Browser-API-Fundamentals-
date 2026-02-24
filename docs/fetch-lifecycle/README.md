# Fetch Lifecycle (Fetch Standard)

## 1. Official Term

Fetch API and the fetch process (request, response, body streams, and abort).

## 2. Definisi Formal

Fetch adalah algoritma networking di Fetch Standard untuk memproses `Request` menjadi `Response`, termasuk mode keamanan (CORS/same-origin), redirect handling, credentials mode, cache interaction, serta body streaming.

`fetch()` adalah Web API host, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

`fetch()` memulai pipeline asinkron:

1. Bangun `Request` (URL, method, headers, mode, credentials, signal).
2. Browser menjalankan fetch algorithm dan kebijakan keamanan terkait.
3. Promise `fetch` resolve ketika response headers tersedia (atau reject pada kondisi tertentu seperti network error/abort).
4. Body dibaca terpisah sebagai stream (`response.body`) atau helper (`json()`, `text()`, dst).

Ini berarti sukses HTTP (misalnya 404/500) tidak otomatis reject Promise.

## 4. Runtime Perspective

- Thread execution:
  - Pemanggilan `fetch()` berasal dari main thread atau worker thread sesuai context.
  - Operasi jaringan dilakukan oleh stack networking browser di luar eksekusi sinkron JavaScript.
- Queue:
  - Penyelesaian Promise `fetch` dan body read menghasilkan microtask callbacks pada execution context pemanggil.
  - Event lain (input/timer) tetap lewat task queue normal.
- Scheduler:
  - Browser scheduler dan networking stack mengatur koneksi, prioritas, redirect, serta delivery data chunk ke stream consumer.
- Rendering timing:
  - `await fetch(...)` tidak memblokir thread secara sinkron, tetapi callback lanjutan tetap memakai main thread jika dipanggil di dokumen utama.
  - Parsing/transform data berat setelah response diterima tetap dapat mengganggu frame rendering.
- Performance implication:
  - Streaming memungkinkan pemrosesan bertahap tanpa menunggu seluruh body selesai.
  - Overfetching dan parsing berlebih di main thread dapat memperlambat interaksi UI.
- Memory implication:
  - Menahan response besar (misalnya `arrayBuffer()`/`json()` besar) sekaligus meningkatkan tekanan memori dibanding pemrosesan streaming bertahap.
- Security implication:
  - Same-Origin Policy, CORS, preflight, credentials mode, mixed content, dan secure context membatasi apa yang dapat diakses script.

## 5. Kenapa API Ini Ada?

Web memerlukan model networking modern yang:

- Promise-based dan composable.
- Mendukung streaming dan abort.
- Terintegrasi dengan kebijakan keamanan web lintas-origin.

Fetch menggantikan model lama yang lebih terbatas dan menyediakan kontrak yang lebih seragam untuk request/response processing.

## 6. Contoh Minimal

```html
<script>
  const controller = new AbortController();

  async function loadData() {
    try {
      const res = await fetch("/api/data", { signal: controller.signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);

      const data = await res.json();
      console.log("items:", data.length);
    } catch (err) {
      if (err.name === "AbortError") {
        console.log("request aborted");
      } else {
        console.error(err);
      }
    }
  }

  loadData();
  // controller.abort(); // panggil bila perlu membatalkan
</script>
```

## 7. Common Misconceptions

- "fetch adalah fitur JavaScript language."
  - Koreksi: fetch adalah Web API dari Fetch Standard.
- "Promise fetch reject untuk semua HTTP error."
  - Koreksi: HTTP 4xx/5xx tetap resolve; cek `response.ok`/`status`.
- "CORS adalah mekanisme untuk mengizinkan server memanggil client."
  - Koreksi: CORS adalah kebijakan browser untuk membatasi akses response lintas-origin oleh script.

## 8. Pitfall & Best Practices

- Pitfall: tidak memeriksa `response.ok`.
  - Dampak: error HTTP dianggap sukses logika aplikasi.
  - Praktik: validasi status sebelum parsing body.
- Pitfall: tidak memakai abort pada request yang bisa menjadi usang.
  - Dampak: pemborosan network/CPU dan race kondisi UI.
  - Praktik: gunakan `AbortController` saat navigasi/filter berubah cepat.
- Pitfall: parsing payload besar di main thread tanpa strategi.
  - Dampak: jank UI.
  - Praktik: streaming/incremental processing atau pindahkan transform berat ke Worker.

## 9. Prerequisite

- Promise and async/await basics.
- Event loop dan microtask.
- SOP/CORS fundamentals.

## 10. Next Topics

- CORS and preflight deep dive
- Streams API integration
- Service Worker fetch interception
- Cache Storage strategy and consistency

## References

- Fetch Standard: https://fetch.spec.whatwg.org/
- HTML Standard (integration with event loop): https://html.spec.whatwg.org/
- MDN Fetch API overview (practical reference): https://developer.mozilla.org/docs/Web/API/Fetch_API
