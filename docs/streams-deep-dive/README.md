# Streams Deep Dive

## 1. Official Term

Streams API (`ReadableStream`, `WritableStream`, `TransformStream`, backpressure, locking, piping).

## 2. Definisi Formal

Streams API mendefinisikan model pemrosesan data bertahap (chunk-by-chunk) dengan kontrol aliran (backpressure) untuk membaca, mentransformasi, dan menulis data tanpa menunggu keseluruhan payload selesai.

Streams adalah Web Platform APIs (WHATWG Streams Standard integration), bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Daripada memuat semua data sekaligus:

1. Producer menghasilkan chunk.
2. Consumer memproses chunk secara bertahap.
3. Backpressure mengatur agar producer tidak melampaui kapasitas consumer.

Model ini menurunkan latency awal dan mengurangi kebutuhan memori puncak.

## 4. Runtime Perspective

- Thread execution:
  - Stream callbacks dieksekusi pada context pemanggil (main thread atau worker).
- Queue:
  - Operasi stream bersifat asinkron dan interaksinya terlihat melalui Promise/microtask.
  - Data flow internal stream dikelola oleh antrean internal controller.
- Scheduler:
  - Browser runtime mengatur delivery chunk berdasarkan readiness consumer dan mekanisme backpressure.
- Rendering timing:
  - Processing chunk berat di main thread tetap dapat mengganggu frame.
  - Streaming di worker dapat membantu menjaga UI responsif.
- Performance implication:
  - Streaming mengurangi time-to-first-byte-to-processing dibanding menunggu payload penuh.
  - Piping antar stream menghindari buffer perantara besar.
- Memory implication:
  - Pemrosesan bertahap menurunkan peak memory dibanding `await response.arrayBuffer()` untuk payload besar.
  - Buffer internal tetap perlu batas yang baik untuk mencegah bloat.
- Security implication:
  - Data stream lintas-origin tetap tunduk SOP/CORS.
  - Parsing incremental harus tetap validasi input untuk menghindari konsumsi resource berlebih dari input berbahaya.

## 5. Kenapa API Ini Ada?

Aplikasi web modern memproses file/video/network payload besar. Model all-at-once tidak efisien untuk latensi dan memori. Streams API hadir agar pipeline data bisa incremental, composable, dan efisien lintas use-case (network, compression, transformasi data).

## 6. Contoh Minimal

```html
<script>
  async function readAsTextStream(url) {
    const res = await fetch(url);
    const reader = res.body
      .pipeThrough(new TextDecoderStream())
      .getReader();

    while (true) {
      const { value, done } = await reader.read();
      if (done) break;
      console.log("chunk:", value.slice(0, 40));
    }
  }
</script>
```

## 7. Common Misconceptions

- "fetch().json() selalu pilihan terbaik."
  - Koreksi: untuk payload besar atau processing progresif, stream sering lebih tepat.
- "Streams otomatis lebih cepat di semua kasus."
  - Koreksi: overhead pipeline bisa tidak sepadan untuk payload kecil.
- "Stream berarti non-blocking total."
  - Koreksi: kerja sinkron berat saat memproses chunk tetap bisa memblokir thread tempat callback berjalan.

## 8. Pitfall & Best Practices

- Pitfall: tidak menangani cancel/error propagation.
  - Dampak: resource leak atau pipeline menggantung.
  - Praktik: tangani `abort`, `cancel`, dan `catch` secara eksplisit.
- Pitfall: transform chunk terlalu berat di main thread.
  - Dampak: jank UI.
  - Praktik: pindahkan transformasi berat ke worker.
- Pitfall: salah kelola locking reader/writer.
  - Dampak: stream tidak bisa dipakai ulang atau dead-end pipeline.
  - Praktik: pahami lifecycle lock dan lepaskan reader/writer dengan benar.

## 9. Prerequisite

- [Fetch Lifecycle](../fetch-lifecycle/README.md)
- [Web Workers and Message Passing](../web-workers-and-message-passing/README.md)
- Promise and async/await basics

## 10. Next Topics

- [BYOB readers and binary protocol parsing](../next-topics/byob-readers-and-binary-protocol-parsing/README.md)
- [Compression Streams integration](../next-topics/compression-streams-integration/README.md)
- [Service Worker Lifecycle and Cache Storage](../service-worker-lifecycle-and-cache-storage/README.md)

## References

- Streams Standard: https://streams.spec.whatwg.org/
- Fetch Standard (body streaming): https://fetch.spec.whatwg.org/
- Streams API (MDN): https://developer.mozilla.org/docs/Web/API/Streams_API


