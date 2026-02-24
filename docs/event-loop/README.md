# Event Loop (HTML Standard)

## 1. Official Term

Event loop.

## 2. Definisi Formal

Event loop adalah mekanisme pemrosesan pekerjaan di HTML Standard yang mengatur urutan eksekusi task, microtask checkpoint, dan rendering opportunity dalam user agent.

Event loop bukan bagian dari ECMAScript language specification. ECMAScript mendefinisikan language semantics (misalnya Promise jobs), sedangkan host environment (browser) mengintegrasikan jobs tersebut ke dalam microtask processing model.

## 3. Mental Model

Bayangkan browser memiliki antrean pekerjaan. Dalam satu putaran:

1. Browser memilih satu task dari task queue.
2. Menjalankan task sampai stack kosong.
3. Menjalankan microtask checkpoint sampai antrean microtask habis.
4. Jika kondisi memungkinkan, browser melakukan rendering update.

Model ini membuat JavaScript tetap single-threaded per execution context, tetapi tetap responsif terhadap input, network completion, timer, dan rendering.

## 4. Runtime Perspective

- Thread execution:
  - Script pada dokumen normal dieksekusi di main thread renderer process.
- Queue:
  - Task queue memuat pekerjaan seperti timer callback, input event dispatch, dan network-related task handoff.
  - Microtask queue memuat Promise reaction jobs dan `queueMicrotask` callbacks.
- Scheduler:
  - Host (browser engine + HTML event loop algorithm) memilih task dan menentukan kapan render opportunity terjadi.
- Rendering timing:
  - Rendering tidak terjadi di tengah eksekusi task sinkron.
  - Setelah task selesai dan microtask checkpoint bersih, browser dapat melakukan style/layout/paint jika dibutuhkan.
- Performance implication:
  - Task panjang memblokir input handling dan menunda frame rendering.
  - Rantai microtask tanpa henti dapat men-starve rendering dan task lain.
- Memory implication:
  - Task atau microtask yang menutup (capture) objek besar lebih lama dari perlu dapat menahan objek tetap reachable lebih lama, menunda garbage collection efektif.
- Security implication:
  - Event loop memisahkan scheduling antar-origin lewat process/site isolation policy browser, tetapi code di dokumen yang sama tetap berbagi main thread execution budget.

## 5. Kenapa API Ini Ada?

Web membutuhkan model konkurensi yang:

- Deterministik untuk script ordering.
- Aman untuk shared UI state (DOM) tanpa data race antar-thread script default.
- Mampu menggabungkan sumber event heterogen: timer, input, networking, parser, lifecycle.

Karena itu, event loop ditempatkan di HTML Standard sebagai host orchestration model, bukan di ECMAScript.

## 6. Contoh Minimal

```html
<script>
  console.log("A: script start");

  setTimeout(() => console.log("D: task (timer)"), 0);

  Promise.resolve().then(() => console.log("C: microtask (promise)"));

  console.log("B: script end");
</script>
```

Urutan output:

1. `A: script start`
2. `B: script end`
3. `C: microtask (promise)`
4. `D: task (timer)`

Penjelasan: script awal berjalan sebagai task. Setelah stack kosong, microtask dijalankan dulu sebelum browser mengambil task berikutnya.

## 7. Common Misconceptions

- "JavaScript punya event loop."
  - Koreksi: event loop adalah model host di HTML Standard.
- "setTimeout(0) langsung jalan."
  - Koreksi: callback timer masuk task queue dan menunggu giliran.
- "Promise callback adalah task."
  - Koreksi: Promise reaction diproses sebagai microtask.
- "Rendering selalu terjadi setelah setiap baris JavaScript."
  - Koreksi: rendering opportunity terjadi di batas loop tertentu, bukan tiap statement.

## 8. Pitfall & Best Practices

- Pitfall: long synchronous task.
  - Dampak: jank, input lag, frame drop.
  - Praktik: pecah kerja berat menjadi chunk (misalnya via timer/task splitting atau worker).
- Pitfall: infinite microtask chaining.
  - Dampak: starvation untuk rendering dan task lain.
  - Praktik: batasi chain dan beri kesempatan kembali ke task queue.
- Pitfall: manipulasi DOM besar dalam satu tick.
  - Dampak: biaya layout/paint tinggi.
  - Praktik: batch update, minimalkan forced sync layout.
- Pitfall: asumsi urutan event lintas API tanpa memahami queue source.
  - Dampak: race condition logika UI/data.
  - Praktik: definisikan ordering secara eksplisit lewat state machine yang jelas.

## 9. Prerequisite

- ECMAScript execution model dasar.
- Call stack dan run-to-completion semantics.
- Promise fundamentals.
- Dasar rendering pipeline (style, layout, paint).

## 10. Next Topics

- Timers (HTML Standard)
- Microtask and Promise jobs integration
- Rendering pipeline and frame budget
- `requestAnimationFrame` and visual scheduling
- Long tasks and responsiveness metrics

## References

- HTML Standard (event loops): https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
- HTML Standard (timers): https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers
- ECMAScript (Jobs and Job Queues): https://tc39.es/ecma262/#sec-jobs-and-job-queues
