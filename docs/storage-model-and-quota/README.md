# Storage Model and Quota

## 1. Official Term

Storage APIs and storage quota management (Web Storage, IndexedDB, Cache Storage, Origin storage bucket model in browser implementations).

## 2. Definisi Formal

Storage model web adalah aturan bagaimana data disimpan, dipartisi, dan dibatasi per origin/site oleh user agent. Quota model menentukan batas kapasitas serta kebijakan eviction ketika ruang penyimpanan terbatas.

Storage APIs adalah Web Platform APIs, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Setiap origin memiliki "ruang data" yang dikelola browser:

1. Aplikasi menulis data via API (misalnya `localStorage`, IndexedDB, Cache Storage).
2. Browser menyimpan data pada storage backend internal.
3. Browser menerapkan quota policy per origin/site/device state.
4. Jika tekanan storage tinggi, data tertentu dapat di-evict sesuai kebijakan browser.

Konsekuensi: persistence bukan jaminan absolut selamanya; aplikasi harus siap menghadapi data hilang/terbatas.

## 4. Runtime Perspective

- Thread execution:
  - `localStorage` bersifat sinkron pada main thread.
  - IndexedDB dan sebagian besar storage modern bersifat asinkron (callback/promise) dan backend I/O berjalan di luar eksekusi sinkron script.
- Queue:
  - Operasi async storage menyelesaikan hasil lewat task/microtask sesuai API masing-masing.
  - Event seperti `storage` (Web Storage) dipublish sebagai event task ke dokumen terkait.
- Scheduler:
  - Browser scheduler mengatur commit I/O, transaction ordering (misalnya IndexedDB), dan eviction timing.
- Rendering timing:
  - Operasi sinkron berat pada main thread (khususnya akses Web Storage besar) dapat mengganggu frame rendering.
  - Operasi async mengurangi blocking, tetapi callback proses data besar tetap bisa membebani frame.
- Performance implication:
  - `localStorage` tidak cocok untuk payload besar/frekuensi tinggi.
  - IndexedDB lebih tepat untuk data terstruktur besar dan transaksi.
- Memory implication:
  - Caching data besar in-memory setelah load dari storage meningkatkan jejak memori runtime.
  - Serialisasi/deserialisasi objek besar memiliki biaya memori sementara.
- Security implication:
  - Isolasi storage mengikuti same-origin boundary dan kebijakan privasi browser (partitioning, third-party restrictions).
  - Data sensitif di storage client dapat terekspos jika origin terkena XSS.

## 5. Kenapa API Ini Ada?

Web app butuh persistence lintas navigasi/sesi:

- Preferensi pengguna.
- State aplikasi offline.
- Cache data untuk latency rendah.

Storage APIs menyediakan opsi dengan tradeoff berbeda: sinkron sederhana (`localStorage`) versus asinkron dan scalable (IndexedDB/Cache Storage).

## 6. Contoh Minimal

```html
<script>
  // Web Storage (sinkron, kecil)
  localStorage.setItem("theme", "dark");
  const theme = localStorage.getItem("theme");
  console.log("theme:", theme);
</script>
```

Untuk data besar/kompleks, gunakan IndexedDB atau Cache Storage, bukan `localStorage`.

## 7. Common Misconceptions

- "Storage browser tidak punya batas."
  - Koreksi: browser menerapkan quota dan dapat melakukan eviction.
- "localStorage cocok untuk semua kebutuhan persistence."
  - Koreksi: `localStorage` sinkron dan tidak ideal untuk data besar/high-frequency access.
- "Data di storage client otomatis aman."
  - Koreksi: keamanan sangat bergantung pada integritas origin; XSS dapat membaca data yang dapat diakses script.

## 8. Pitfall & Best Practices

- Pitfall: menyimpan data besar di `localStorage`.
  - Dampak: blocking main thread, UX menurun.
  - Praktik: gunakan IndexedDB/Cache Storage untuk payload besar.
- Pitfall: asumsi data selalu persisten permanen.
  - Dampak: bug saat data di-evict.
  - Praktik: desain mekanisme recovery/sync ulang dari server.
- Pitfall: menyimpan data rahasia mentah di client storage.
  - Dampak: risiko kebocoran saat XSS/kompromi origin.
  - Praktik: minimalkan data sensitif client-side dan terapkan mitigasi XSS ketat.

## 9. Prerequisite

- Same-Origin Policy fundamentals.
- Event loop and async basics.
- JSON serialization basics.

## 10. Next Topics

- [IndexedDB transaction model deep dive](../next-topics/indexeddb-transaction-model-deep-dive/README.md)
- [Service Worker Lifecycle and Cache Storage](../service-worker-lifecycle-and-cache-storage/README.md)
- [Security Boundary](../security-boundary/README.md)

## References

- Storage Standard: https://storage.spec.whatwg.org/
- HTML Standard (Web Storage): https://html.spec.whatwg.org/multipage/webstorage.html
- Indexed Database API 3.0: https://w3c.github.io/IndexedDB/
- Service Workers / Cache Storage: https://w3c.github.io/ServiceWorker/


