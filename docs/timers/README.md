# Timers (HTML Standard)

## 1. Official Term

Timers (`setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`).

## 2. Definisi Formal

Timers adalah mekanisme scheduling di HTML Standard untuk menjadwalkan callback sebagai task setelah delay minimum tertentu.

Timer callback tidak dieksekusi oleh ECMAScript specification secara langsung. Callback dieksekusi ketika event loop memilih task yang berasal dari timer task source.

## 3. Mental Model

`setTimeout(fn, delay)` bukan "tidur lalu langsung jalankan `fn`". Yang terjadi:

1. Browser mencatat timer dengan target waktu tertentu.
2. Saat waktunya tiba, browser membuat task untuk callback.
3. Callback baru berjalan ketika event loop mengambil task tersebut.

Artinya delay adalah waktu minimum sebelum callback bisa diantrekan, bukan jaminan waktu eksekusi persis.

## 4. Runtime Perspective

- Thread execution:
  - Pada dokumen biasa, callback timer berjalan di main thread renderer ketika task-nya dipilih.
- Queue:
  - Callback timer masuk ke task queue (timer task source).
  - Sebelum task berikutnya, microtask checkpoint tetap diproses terlebih dahulu.
- Scheduler:
  - Host scheduler menentukan kapan timer dianggap matang (eligible), lalu kapan task-nya dieksekusi sesuai tekanan antrean.
- Rendering timing:
  - Rendering dapat terjadi di antara task-task, bukan di tengah callback sinkron.
  - Callback timer panjang dapat menunda frame berikutnya.
- Performance implication:
  - Delay kecil (`0`/`1` ms) tidak berarti realtime; tetap bisa terlambat saat main thread sibuk.
  - `setInterval` dapat menumpuk callback jika pekerjaan lebih lambat dari periodenya.
- Memory implication:
  - Timer aktif mempertahankan referensi callback dan nilai yang tertutup closure; tanpa cleanup referensi ini dapat memperpanjang lifetime objek.
- Security implication:
  - Browser dapat menerapkan timer clamping/throttling (misalnya tab background) untuk mitigasi abuse dan efisiensi resource.

## 5. Kenapa API Ini Ada?

Web membutuhkan primitive scheduling sederhana untuk:

- Menunda kerja non-kritis.
- Melakukan polling ringan.
- Memecah pekerjaan sinkron besar menjadi beberapa task.

Primitive ini berada di HTML Standard karena terkait langsung dengan event loop host dan lifecycle dokumen.

## 6. Contoh Minimal

```html
<script>
  const startedAt = performance.now();

  setTimeout(() => {
    const elapsed = performance.now() - startedAt;
    console.log(`Timer fired after ~${Math.round(elapsed)}ms`);
  }, 0);

  // Simulasi blocking main thread.
  const blockUntil = performance.now() + 80;
  while (performance.now() < blockUntil) {}
</script>
```

Walau delay `0`, callback tetap menunggu task saat ini selesai, sehingga hasil nyata > `0` ms.

## 7. Common Misconceptions

- "setTimeout adalah fitur JavaScript language."
  - Koreksi: timers adalah Web API (HTML Standard).
- "setTimeout(fn, 0) berarti langsung jalan."
  - Koreksi: callback dijadwalkan sebagai task, bukan dieksekusi instan.
- "setInterval selalu presisi periodik."
  - Koreksi: eksekusi bisa drift karena beban main thread dan kebijakan throttling.

## 8. Pitfall & Best Practices

- Pitfall: memakai `setInterval` untuk kerja yang bisa overlap.
  - Dampak: backlog callback dan state race.
  - Praktik: gunakan pola recursive `setTimeout` agar jadwal berikutnya dibuat setelah kerja selesai.
- Pitfall: asumsi delay akurat untuk metrik performa.
  - Dampak: hasil observasi bias.
  - Praktik: gunakan `performance.now()` untuk ukur real elapsed time.
- Pitfall: lupa membersihkan timer pada lifecycle tertentu.
  - Dampak: memory leak/logika berjalan saat context tidak relevan.
  - Praktik: simpan id timer dan `clearTimeout`/`clearInterval` saat cleanup.

## 9. Prerequisite

- Event loop fundamentals.
- Task vs microtask.
- Dasar main-thread rendering cost.

## 10. Next Topics

- Microtask and Promise jobs integration
- `requestAnimationFrame` for visual work
- Page Visibility and background throttling
- Long tasks and responsiveness

## References

- HTML Standard (timers): https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers
- HTML Standard (event loops): https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
