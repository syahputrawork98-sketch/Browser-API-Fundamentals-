# Microtask and Promise Jobs Integration

## 1. Official Term

Microtask checkpoint (HTML Standard) and Promise Jobs (ECMAScript).

## 2. Definisi Formal

ECMAScript mendefinisikan Promise reactions sebagai jobs pada job queue model bahasa. Browser sebagai host environment menghubungkan jobs tersebut ke microtask processing melalui mekanisme microtask checkpoint di event loop HTML Standard.

Secara praktis di browser:

- `Promise.then/catch/finally` reaction callbacks diproses sebagai microtask.
- `queueMicrotask` juga menambahkan callback ke antrean microtask host.

## 3. Mental Model

Setelah satu task selesai, browser menjalankan "drain microtasks":

1. Ambil microtask pertama, jalankan.
2. Jika microtask membuat microtask baru, antrekan di belakang.
3. Ulangi sampai antrean microtask kosong.
4. Lanjut ke fase event loop berikutnya (termasuk kemungkinan rendering).

Karena seluruh antrean harus habis dulu, microtask dapat "mendahului" task lain yang sudah menunggu.

## 4. Runtime Perspective

- Thread execution:
  - Pada konteks dokumen utama, microtask callback berjalan di main thread yang sama dengan script lain.
- Queue:
  - Task queue berisi callback dari timers/input/network.
  - Microtask queue berisi Promise reactions dan callback `queueMicrotask`.
- Scheduler:
  - Host menjalankan microtask checkpoint pada titik sinkronisasi event loop (misalnya setelah task selesai).
- Rendering timing:
  - Rendering opportunity menunggu microtask queue kosong.
  - Rantai microtask panjang bisa menunda paint.
- Performance implication:
  - Microtask cocok untuk pekerjaan kecil yang butuh ordering segera setelah task saat ini.
  - Penyalahgunaan microtask chain dapat membuat UI terasa freeze walau tanpa loop sinkron panjang tradisional.
- Memory implication:
  - Rantai Promise/microtask yang menahan closure besar dapat memperpanjang reachability objek, terutama saat antrean terus bertambah sebelum sempat idle.
- Security implication:
  - Walau scheduling cepat, microtask tetap tunduk pada boundary origin/process isolation browser; tidak memberi privilege baru lintas origin.

## 5. Kenapa API Ini Ada?

Promise membutuhkan mekanisme asinkron yang konsisten dan deterministik untuk callback ordering. Microtask checkpoint memberikan jaminan:

- Callback reaction tidak berjalan sinkron di tengah eksekusi saat ini.
- Callback reaction berjalan sebelum task eksternal berikutnya, sehingga dependency chain tetap terurut.

Ini penting untuk komposisi async yang prediktif.

## 6. Contoh Minimal

```html
<script>
  console.log("1: start task");

  setTimeout(() => console.log("4: timer task"), 0);

  Promise.resolve()
    .then(() => {
      console.log("2: promise microtask");
      queueMicrotask(() => console.log("3: nested microtask"));
    });

  console.log("end: task body");
</script>
```

Urutan utama: log sinkron task -> semua microtask -> task timer.

## 7. Common Misconceptions

- "Promise callback adalah setara setTimeout."
  - Koreksi: Promise reaction adalah microtask, `setTimeout` adalah task.
- "queueMicrotask sama dengan setTimeout(..., 0)."
  - Koreksi: `queueMicrotask` diproses sebelum task berikutnya.
- "Microtask selalu aman untuk pekerjaan banyak."
  - Koreksi: chain berlebihan bisa menunda rendering dan event handling.

## 8. Pitfall & Best Practices

- Pitfall: recursive microtask tanpa batas.
  - Dampak: starvation task/rendering.
  - Praktik: untuk kerja besar, beri yield ke task queue secara periodik.
- Pitfall: campur task dan microtask tanpa kontrak ordering.
  - Dampak: bug urutan yang sulit direproduksi.
  - Praktik: dokumentasikan urutan yang diharapkan dan uji explicit ordering.
- Pitfall: kerja berat di microtask.
  - Dampak: blocking tetap terjadi di main thread.
  - Praktik: pindahkan kerja CPU-heavy ke Worker.

## 9. Prerequisite

- Event loop fundamentals.
- Promise lifecycle dasar.
- Task vs microtask semantics.

## 10. Next Topics

- [requestAnimationFrame](../request-animation-frame/README.md)
- [Long tasks API and responsiveness diagnostics](../next-topics/long-tasks-api-and-responsiveness-diagnostics/README.md)
- [Web Workers and Message Passing](../web-workers-and-message-passing/README.md)

## References

- HTML Standard (microtask queuing): https://html.spec.whatwg.org/multipage/webappapis.html#microtask-queuing
- HTML Standard (event loops): https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
- ECMAScript (Jobs and Job Queues): https://tc39.es/ecma262/#sec-jobs-and-job-queues


