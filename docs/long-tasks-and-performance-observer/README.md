# Long Tasks and PerformanceObserver

## 1. Official Term

Long Tasks API (`PerformanceLongTaskTiming`) and `PerformanceObserver` interface.

## 2. Definisi Formal

Long Tasks API mengekspos entri performa untuk task yang memblokir main thread lebih lama dari ambang tertentu (umumnya >50ms). `PerformanceObserver` adalah mekanisme observasi entry performa asinkron dari Performance Timeline.

Keduanya adalah Web Platform APIs, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Jika main thread sibuk terlalu lama:

1. Input tertunda.
2. Rendering frame tertunda.
3. Browser menandai pekerjaan itu sebagai long task.
4. Aplikasi bisa mengamati sinyal ini via `PerformanceObserver`.

Ini memberi visibilitas konkret terhadap jank yang sering tidak terlihat dari log biasa.

## 4. Runtime Perspective

- Thread execution:
  - Long task terutama relevan untuk main thread dokumen tempat UI/rendering berjalan.
- Queue:
  - Long task terjadi saat satu task di event loop memonopoli waktu eksekusi terlalu lama.
  - Callback observer dipicu asinkron saat performance entries tersedia.
- Scheduler:
  - Browser runtime mengukur durasi task dan menghasilkan performance entries sesuai spesifikasi timeline.
- Rendering timing:
  - Long task mengurangi atau melewati rendering opportunity, menyebabkan dropped frame dan input delay.
- Performance implication:
  - Identifikasi long task membantu menargetkan bottleneck JS, layout, atau kombinasi keduanya.
  - Observasi kontinu memberi sinyal regressi performa saat perubahan kode.
- Memory implication:
  - Menyimpan terlalu banyak performance entries atau payload telemetry tanpa batas dapat menambah konsumsi memori.
- Security implication:
  - Timing data adalah data sensitif performa; browser menerapkan pembatasan tertentu untuk mengurangi potensi side-channel.

## 5. Kenapa API Ini Ada?

Metrik umum seperti total load time tidak cukup menjelaskan responsivitas runtime. Long Tasks API hadir untuk mengukur gangguan interaktivitas di level event loop, sehingga optimasi dapat diarahkan ke pekerjaan yang benar-benar memblokir pengalaman pengguna.

## 6. Contoh Minimal

```html
<script>
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.log("Long task:", Math.round(entry.duration), "ms");
    }
  });

  observer.observe({ type: "longtask", buffered: true });

  // Simulasi blocking.
  const end = performance.now() + 120;
  while (performance.now() < end) {}
</script>
```

## 7. Common Misconceptions

- "Semua lag UI pasti karena network."
  - Koreksi: long task di main thread sering jadi penyebab utama input/render delay.
- "Kalau pakai async/await pasti non-blocking."
  - Koreksi: pekerjaan sinkron berat di dalam callback async tetap memblokir thread.
- "PerformanceObserver hanya untuk analytics akhir."
  - Koreksi: API ini juga cocok untuk diagnosis langsung selama development.

## 8. Pitfall & Best Practices

- Pitfall: menganggap semua long task berasal dari JavaScript murni.
  - Dampak: diagnosis parsial.
  - Praktik: korelasikan long tasks dengan tooling rendering/network timeline.
- Pitfall: observer aktif tanpa lifecycle cleanup.
  - Dampak: overhead observasi tidak perlu.
  - Praktik: `disconnect()` saat observasi tidak dibutuhkan.
- Pitfall: hanya mengukur, tidak menindaklanjuti.
  - Dampak: tidak ada perbaikan UX nyata.
  - Praktik: pecah kerja panjang, gunakan worker, dan sinkronkan visual update dengan rAF.

## 9. Prerequisite

- [Event Loop](../event-loop/README.md)
- [Rendering Pipeline Interaction](../rendering-pipeline-interaction/README.md)
- [requestAnimationFrame](../request-animation-frame/README.md)

## 10. Next Topics

- [Event Timing and INP diagnostics](../next-topics/event-timing-and-inp-diagnostics/README.md)
- [Scheduler API and cooperative yielding](../next-topics/scheduler-api-and-cooperative-yielding/README.md)
- [Web Workers and Message Passing](../web-workers-and-message-passing/README.md)

## References

- Long Tasks API: https://w3c.github.io/longtasks/
- Performance Timeline: https://w3c.github.io/performance-timeline/
- PerformanceObserver (MDN): https://developer.mozilla.org/docs/Web/API/PerformanceObserver


