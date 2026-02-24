# requestAnimationFrame (HTML Standard)

## 1. Official Term

`AnimationFrameProvider.requestAnimationFrame()`.

## 2. Definisi Formal

`requestAnimationFrame` adalah API host (Web Platform) yang menjadwalkan callback untuk dijalankan sebelum repaint berikutnya pada dokumen.

API ini bukan bagian dari ECMAScript language specification. Perilaku scheduling callback ditentukan oleh HTML Standard dan terintegrasi dengan event loop serta rendering update steps user agent.

## 3. Mental Model

`requestAnimationFrame` adalah sinkronisasi kerja visual dengan siklus frame browser:

1. Aplikasi meminta callback frame.
2. Browser menempatkan callback untuk fase sebelum paint.
3. Callback menerima `DOMHighResTimeStamp`.
4. Setelah callback selesai, browser dapat melanjutkan style/layout/paint sesuai kebutuhan.

Gunakan API ini untuk update visual per-frame, bukan `setInterval` untuk animasi UI.

## 4. Runtime Perspective

- Thread execution:
  - Pada dokumen utama, callback berjalan di main thread renderer process.
- Queue:
  - Callback rAF bukan timer task biasa; callback dipanggil pada tahap animation frame callbacks dalam rendering cycle.
  - Microtask checkpoint tetap relevan: microtask yang dibuat di callback harus selesai sebelum loop lanjut.
- Scheduler:
  - Browser menentukan kapan frame berikutnya tersedia dan callback mana yang dipanggil pada frame tersebut.
  - Jika tab/background state berubah, callback bisa ditunda atau di-throttle.
- Rendering timing:
  - Callback dipanggil sebelum repaint berikutnya, sehingga cocok untuk membaca waktu frame dan menulis perubahan visual.
  - Callback berat tetap bisa melewati frame budget dan menyebabkan dropped frames.
- Performance implication:
  - Menyamakan update visual dengan refresh cycle mengurangi jank dibanding timer periodik buta.
  - Perhitungan/layout berat di callback dapat melampaui budget frame (~16.67ms pada 60Hz).
- Memory implication:
  - Loop animasi berantai yang menyimpan closure/state besar tanpa penghentian dapat memperpanjang lifetime objek dan menambah tekanan GC.
- Security implication:
  - Browser dapat membatasi frekuensi callback pada context non-visible untuk efisiensi dan mitigasi abuse timing behavior.

## 5. Kenapa API Ini Ada?

Kebutuhan utama web UI adalah sinkronisasi animasi dengan pipeline rendering browser. Timer generik tidak memiliki kontrak "sebelum repaint", sehingga dapat menghasilkan update yang tidak selaras dengan frame.

`requestAnimationFrame` menyediakan kontrak visual scheduling yang lebih tepat untuk animasi, scrolling effect, dan interpolasi state berbasis timestamp frame.

## 6. Contoh Minimal

```html
<script>
  const box = document.getElementById("box");
  let start = null;

  function tick(ts) {
    if (start === null) start = ts;
    const elapsed = ts - start;
    box.style.transform = `translateX(${Math.min(elapsed * 0.1, 300)}px)`;

    if (elapsed < 3000) {
      requestAnimationFrame(tick);
    }
  }

  requestAnimationFrame(tick);
</script>
```

Contoh di atas menggerakkan elemen berdasarkan timestamp frame, bukan asumsi interval tetap.

## 7. Common Misconceptions

- "requestAnimationFrame adalah fitur JavaScript language."
  - Koreksi: ini Web API pada host environment browser.
- "rAF selalu 60 FPS."
  - Koreksi: frekuensi mengikuti kondisi runtime (refresh rate, throttling, visibilitas tab, beban thread).
- "rAF cocok untuk polling data network."
  - Koreksi: rAF ditujukan untuk pekerjaan visual yang terikat repaint.

## 8. Pitfall & Best Practices

- Pitfall: animasi pakai `setInterval` untuk update DOM per-frame.
  - Dampak: desinkronisasi dengan repaint, jank lebih tinggi.
  - Praktik: gunakan rAF untuk visual updates.
- Pitfall: lupa menghentikan loop rAF.
  - Dampak: kerja terus berjalan saat tidak dibutuhkan, boros CPU/memori.
  - Praktik: simpan id callback dan hentikan via `cancelAnimationFrame` saat cleanup.
- Pitfall: baca-tulis layout berulang dalam satu callback.
  - Dampak: forced synchronous layout dan biaya frame besar.
  - Praktik: batch reads lalu writes, minimalkan layout thrashing.

## 9. Prerequisite

- Event loop fundamentals.
- Rendering pipeline dasar (style, layout, paint, composite).
- Task vs microtask basics.

## 10. Next Topics

- Rendering pipeline interaction details
- Long Tasks API
- Page Visibility and lifecycle-driven scheduling
- Performance timeline and frame diagnostics

## References

- HTML Standard (`requestAnimationFrame`): https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames
- HTML Standard (event loops): https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
