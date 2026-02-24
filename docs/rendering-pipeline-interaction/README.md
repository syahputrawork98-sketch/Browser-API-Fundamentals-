# Rendering Pipeline Interaction

## 1. Official Term

Rendering pipeline stages: style calculation, layout, paint, and compositing.

## 2. Definisi Formal

Rendering pipeline interaction menjelaskan bagaimana perubahan DOM/CSSOM dari script memengaruhi tahapan engine rendering sampai frame akhir tampil di layar.

Pipeline rendering adalah bagian implementasi browser engine (Blink/WebKit/Gecko) yang terintegrasi dengan event loop HTML Standard, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Ketika script mengubah DOM/style:

1. Browser menandai bagian dokumen yang invalid.
2. Pada waktu yang tepat, browser menghitung style.
3. Browser menghitung layout (geometri).
4. Browser melakukan paint (rasterization per layer/area).
5. Browser melakukan compositing untuk menghasilkan frame final.

Tidak semua perubahan melewati semua tahap dengan biaya sama; jenis properti/style memengaruhi tahap mana yang invalid.

## 4. Runtime Perspective

- Thread execution:
  - JavaScript dan banyak operasi DOM berjalan di main thread.
  - Sebagian proses raster/composite dapat melibatkan thread/process lain tergantung engine.
- Queue:
  - Perubahan dari script terjadi dalam task/microtask tempat script dieksekusi.
  - Frame rendering diproses pada rendering opportunity yang diselaraskan event loop.
- Scheduler:
  - Browser menentukan kapan flush style/layout/paint berdasarkan invalidation state, frame cadence, dan prioritas internal.
- Rendering timing:
  - Rendering umumnya terjadi setelah task selesai dan microtask checkpoint bersih.
  - Operasi baca layout sinkron (misalnya `getBoundingClientRect`) setelah write style dapat memaksa layout dihitung lebih awal.
- Performance implication:
  - Layout thrashing (write-read-write berulang) meningkatkan biaya frame dan memicu jank.
  - Properti transform/opacity sering lebih murah untuk animasi karena dapat mengurangi kerja layout/paint penuh.
- Memory implication:
  - Layer tambahan, cache raster, dan objek DOM/CSS besar meningkatkan jejak memori rendering.
  - Animasi/asset besar tanpa manajemen dapat menambah tekanan memori GPU/renderer.
- Security implication:
  - Informasi timing rendering dibatasi oleh kebijakan platform tertentu (coarsening/throttling di konteks tertentu) untuk mengurangi potensi side-channel.

## 5. Kenapa API Ini Ada?

Meski bukan satu API tunggal, memahami interaksi pipeline rendering penting agar:

- Perubahan UI diprediksi dampaknya.
- Animasi tetap halus dalam frame budget.
- Keputusan arsitektur UI mempertimbangkan biaya style/layout/paint/composite.

Tanpa model ini, optimasi performa frontend cenderung trial-and-error.

## 6. Contoh Minimal

```html
<div id="box"></div>
<script>
  const box = document.getElementById("box");

  // Write
  box.style.width = "200px";

  // Read yang bisa memaksa layout flush jika ada invalidasi.
  const w = box.getBoundingClientRect().width;
  console.log(w);
</script>
```

Pola write lalu read layout dalam tick yang sama dapat memicu forced synchronous layout.

## 7. Common Misconceptions

- "Semua perubahan style biayanya sama."
  - Koreksi: biaya tergantung apakah perubahan memicu style-only, layout, paint, atau composite.
- "Rendering berjalan terus menerus walau tidak ada perubahan."
  - Koreksi: browser umumnya merender saat diperlukan atau saat frame scheduling relevan.
- "Jank hanya terjadi kalau JavaScript berat."
  - Koreksi: layout/paint mahal juga dapat menyebabkan frame drop.

## 8. Pitfall & Best Practices

- Pitfall: interleave read/write layout dalam loop.
  - Dampak: layout thrashing.
  - Praktik: kelompokkan reads terlebih dulu, lalu writes.
- Pitfall: animasi properti yang memicu layout besar.
  - Dampak: frame budget mudah terlampaui.
  - Praktik: prioritaskan transform/opacity untuk animasi umum.
- Pitfall: tidak mengukur.
  - Dampak: optimasi asumtif, sulit verifikasi.
  - Praktik: gunakan browser performance tooling untuk identifikasi bottleneck aktual.

## 9. Prerequisite

- Event loop and rendering opportunity.
- DOM and CSSOM basics.
- `requestAnimationFrame` fundamentals.

## 10. Next Topics

- [Frame budget and long tasks](../next-topics/frame-budget-and-long-tasks/README.md)
- [CSS containment and rendering isolation](../next-topics/css-containment-and-rendering-isolation/README.md)
- [PerformanceObserver and rendering diagnostics](../next-topics/performanceobserver-and-rendering-diagnostics/README.md)

## References

- HTML Standard (event loop rendering integration): https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
- CSSOM specification: https://drafts.csswg.org/cssom/
- Rendering performance fundamentals (web.dev): https://web.dev/rendering-performance/


