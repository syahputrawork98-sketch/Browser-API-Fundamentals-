# Web Workers and Message Passing

## 1. Official Term

Web Workers API (`Worker`, `DedicatedWorkerGlobalScope`, `postMessage`, structured clone, transferable objects).

## 2. Definisi Formal

Web Worker adalah execution context terpisah yang memungkinkan script berjalan di thread berbeda dari main thread dokumen. Komunikasi antar-context dilakukan melalui message passing, bukan shared DOM access langsung.

Worker APIs adalah Web Platform APIs (HTML + related standards), bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Main thread tetap fokus pada UI/rendering, sementara kerja CPU-heavy dipindahkan ke worker:

1. Main thread membuat worker.
2. Main thread mengirim data via `postMessage`.
3. Worker memproses data di event loop worker-nya sendiri.
4. Worker mengembalikan hasil via `postMessage`.

Tidak ada akses DOM langsung dari dedicated worker, sehingga interaksi UI tetap lewat main thread.

## 4. Runtime Perspective

- Thread execution:
  - Main thread menjalankan DOM/UI logic.
  - Worker berjalan di thread terpisah dengan global scope berbeda.
- Queue:
  - Pesan antar main thread dan worker masuk sebagai task pada event loop masing-masing context.
  - Promise callbacks di masing-masing context tetap berjalan sebagai microtask lokal context tersebut.
- Scheduler:
  - Browser scheduler mengalokasikan eksekusi antar thread/context dan memproses message delivery sesuai antrean event loop masing-masing.
- Rendering timing:
  - Pekerjaan di worker tidak langsung memblokir rendering main thread.
  - Namun handler hasil di main thread tetap bisa menyebabkan jank jika melakukan proses berat sebelum update UI.
- Performance implication:
  - Worker efektif untuk komputasi berat, parsing besar, transformasi data.
  - Ada biaya serialisasi/transfer saat kirim pesan.
- Memory implication:
  - Structured clone menyalin data, sehingga footprint memori sementara bisa meningkat.
  - Transferable objects memindahkan ownership buffer untuk mengurangi copy cost.
- Security implication:
  - Worker tetap terikat origin security model.
  - Beberapa fitur lanjutan (misalnya `SharedArrayBuffer`) memerlukan syarat keamanan tambahan seperti cross-origin isolation.

## 5. Kenapa API Ini Ada?

Model default web menjalankan JavaScript utama di satu thread yang juga kritikal untuk interaksi UI. Untuk menjaga responsivitas, dibutuhkan mekanisme concurrency terisolasi tanpa membuka race condition langsung pada DOM.

Workers memberikan kompromi:

- Paralelisme komputasi.
- Isolasi state antar-context.
- Komunikasi eksplisit via message passing.

## 6. Contoh Minimal

```html
<!-- main.js -->
<script type="module">
  const worker = new Worker("./worker.js", { type: "module" });
  worker.onmessage = (e) => console.log("result:", e.data);
  worker.postMessage({ n: 1_000_000 });
</script>
```

```js
// worker.js
self.onmessage = (e) => {
  let sum = 0;
  for (let i = 0; i < e.data.n; i++) sum += i;
  self.postMessage(sum);
};
```

## 7. Common Misconceptions

- "Worker bisa akses DOM langsung."
  - Koreksi: dedicated worker tidak memiliki akses DOM dokumen.
- "Worker menghilangkan semua bottleneck performa."
  - Koreksi: bottleneck bisa pindah ke serialisasi data, sinkronisasi hasil, atau handler main thread.
- "postMessage selalu murah."
  - Koreksi: structured clone untuk objek besar dapat mahal; pertimbangkan transferable.

## 8. Pitfall & Best Practices

- Pitfall: mengirim objek besar berulang tanpa strategi transfer.
  - Dampak: biaya copy tinggi, memori sementara membengkak.
  - Praktik: gunakan transferable objects untuk buffer besar.
- Pitfall: lifecycle worker tidak dikelola.
  - Dampak: thread idle tetap hidup, resource terbuang.
  - Praktik: `worker.terminate()` saat worker tidak lagi dibutuhkan.
- Pitfall: membebani main thread saat menerima hasil.
  - Dampak: jank tetap terjadi meski komputasi dipindah ke worker.
  - Praktik: lakukan minimal work di handler UI, chunk pekerjaan lanjutan jika perlu.

## 9. Prerequisite

- Event loop and task/microtask basics.
- Structured clone concept.
- Main-thread rendering constraints.

## 10. Next Topics

- [Shared workers and multi-tab coordination](../next-topics/shared-workers-and-multi-tab-coordination/README.md)
- [SharedArrayBuffer and Atomics model](../next-topics/sharedarraybuffer-and-atomics-model/README.md)
- [Service Worker Lifecycle and Cache Storage](../service-worker-lifecycle-and-cache-storage/README.md)

## References

- HTML Standard (workers): https://html.spec.whatwg.org/multipage/workers.html
- MDN Web Workers API: https://developer.mozilla.org/docs/Web/API/Web_Workers_API


