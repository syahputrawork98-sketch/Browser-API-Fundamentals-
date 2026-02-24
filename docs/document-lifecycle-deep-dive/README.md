# Document Lifecycle Deep Dive

## 1. Official Term

Document lifecycle events and states (`DOMContentLoaded`, `load`, `pageshow`, `pagehide`, `visibilitychange`, `beforeunload`, `unload`, bfcache-related behavior).

## 2. Definisi Formal

Document lifecycle adalah rangkaian state dan event yang menggambarkan perjalanan dokumen dari parsing, siap-interaksi, aktif/tersembunyi, hingga ditinggalkan atau dipulihkan kembali (termasuk Back/Forward Cache).

Lifecycle behavior ditentukan oleh HTML Standard dan integrasi browser navigation model, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Satu dokumen tidak hanya "loaded atau tidak". Ada fase:

1. Parsing awal dan pembentukan DOM.
2. `DOMContentLoaded` saat DOM selesai dibangun.
3. `load` saat resource subresource utama selesai dimuat.
4. Fase aktif dengan kemungkinan perubahan visibility.
5. Saat navigasi, dokumen bisa dibuang atau dimasukkan ke bfcache.
6. Saat kembali lewat history traversal, dokumen bisa dipulihkan cepat tanpa bootstrap ulang penuh.

Model ini penting agar inisialisasi, cleanup, dan resume aplikasi tepat waktu.

## 4. Runtime Perspective

- Thread execution:
  - Event lifecycle dokumen umumnya diproses di main thread renderer context dokumen tersebut.
- Queue:
  - Event lifecycle (`DOMContentLoaded`, `load`, `visibilitychange`, `pageshow`, `pagehide`) didispatch sebagai task pada event loop dokumen.
  - Callback Promise/async lanjutan tetap dijalankan sebagai microtask sesuai konteks.
- Scheduler:
  - Browser scheduler menentukan kapan event lifecycle dikirim berdasarkan progres parsing, network, dan transisi navigation/history.
- Rendering timing:
  - `DOMContentLoaded` dapat terjadi sebelum seluruh asset selesai, sehingga rendering lanjutan masih bisa berubah.
  - Restorasi dari bfcache dapat menampilkan halaman cepat sebelum jalur bootstrap normal dieksekusi ulang.
- Performance implication:
  - Memindahkan kerja non-kritis dari fase awal load mengurangi Time to Interactive.
  - Dukungan bfcache yang baik mempercepat back/forward navigation secara signifikan.
- Memory implication:
  - Dokumen yang eligible bfcache mempertahankan state in-memory; pengelolaan resource harus memperhitungkan retensi sementara ini.
  - Listener/resource yang tidak dibersihkan dapat memperbesar footprint saat lifecycle berulang.
- Security implication:
  - Lifecycle event dapat memicu logic penyimpanan data sensitif; implementasi harus menghindari eksposur berlebih saat tab background/restore.
  - Beberapa lifecycle hook dibatasi browser untuk mencegah abuse UX/security (misalnya perilaku `beforeunload`).

## 5. Kenapa API Ini Ada?

Browser perlu kontrak standar agar aplikasi dapat:

- Mengetahui kapan DOM siap dipakai.
- Menyesuaikan kerja saat halaman visible/hidden.
- Menangani navigasi history dengan benar.
- Mengoptimalkan UX melalui bfcache-compatible behavior.

Tanpa lifecycle model yang jelas, app cenderung over-initialize, cleanup salah waktu, dan performa navigasi buruk.

## 6. Contoh Minimal

```html
<script>
  document.addEventListener("DOMContentLoaded", () => {
    console.log("DOM ready");
  });

  window.addEventListener("load", () => {
    console.log("all critical resources loaded");
  });

  document.addEventListener("visibilitychange", () => {
    console.log("visibility:", document.visibilityState);
  });

  window.addEventListener("pageshow", (e) => {
    console.log("pageshow, from bfcache:", e.persisted);
  });
</script>
```

## 7. Common Misconceptions

- "`load` selalu event terbaik untuk inisialisasi awal."
  - Koreksi: banyak inisialisasi UI sebaiknya dimulai di `DOMContentLoaded`.
- "Navigasi back selalu reload penuh."
  - Koreksi: browser bisa memulihkan halaman dari bfcache.
- "`beforeunload` bisa diandalkan untuk semua cleanup."
  - Koreksi: eksekusi hook ini dibatasi; jangan jadikan satu-satunya mekanisme cleanup/persistensi.

## 8. Pitfall & Best Practices

- Pitfall: logika startup berat dijalankan terlalu dini.
  - Dampak: first interaction lambat.
  - Praktik: defer kerja non-kritis, gunakan lifecycle event yang tepat.
- Pitfall: tidak bfcache-aware.
  - Dampak: state aplikasi tidak sinkron saat restore history.
  - Praktik: tangani `pageshow`/`pagehide`, dan hindari pola yang membuat halaman tidak eligible bfcache.
- Pitfall: bergantung pada `unload`.
  - Dampak: perilaku tidak konsisten lintas browser/optimisasi modern.
  - Praktik: prioritaskan `visibilitychange`/`pagehide` untuk persistensi ringan.

## 9. Prerequisite

- Event loop fundamentals.
- URL and History model.
- Rendering pipeline basics.

## 10. Next Topics

- [URL and History Model](../url-and-history-model/README.md)
- [Rendering Pipeline Interaction](../rendering-pipeline-interaction/README.md)
- [Page Lifecycle API and advanced background behavior](../next-topics/page-lifecycle-api-and-advanced-background-behavior/README.md)

## References

- HTML Standard (document readiness): https://html.spec.whatwg.org/multipage/dom.html#current-document-readiness
- HTML Standard (event handlers and lifecycle-related events): https://html.spec.whatwg.org/
- web.dev bfcache guide: https://web.dev/bfcache/


