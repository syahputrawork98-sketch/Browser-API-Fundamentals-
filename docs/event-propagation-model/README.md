# Event Propagation Model (DOM Standard)

## 1. Official Term

Event dispatch and event propagation (capturing phase, target phase, bubbling phase).

## 2. Definisi Formal

Event propagation adalah algoritma dispatch event pada DOM Standard yang menentukan jalur (event path) dan urutan listener invocation dari root ke target lalu kembali ke ancestor (jika event bersifat bubbling).

Model ini adalah bagian dari Web Platform (DOM/HTML integration), bukan ECMAScript language.

## 3. Mental Model

Ketika event terjadi pada suatu node:

1. Browser membentuk event path dari root ke target.
2. Fase capturing: listener dengan `capture: true` berjalan dari ancestor ke arah target.
3. Fase target: listener di target dijalankan.
4. Fase bubbling: jika `event.bubbles === true`, listener non-capturing berjalan dari target ke ancestor.

`stopPropagation()` menghentikan perjalanan ke node berikutnya.  
`stopImmediatePropagation()` juga menghentikan listener berikutnya pada node saat ini.

## 4. Runtime Perspective

- Thread execution:
  - Pada dokumen normal, dispatch event DOM berjalan di main thread renderer.
- Queue:
  - Banyak event pengguna (click, key, pointer) masuk dari task source input sebagai task event loop.
  - Listener sinkron berjalan dalam task yang sama saat dispatch berlangsung.
- Scheduler:
  - Browser scheduler menentukan kapan task input diproses; DOM dispatch algorithm menentukan urutan listener dalam task itu.
- Rendering timing:
  - Perubahan DOM/style oleh listener dapat memengaruhi render pada frame berikutnya.
  - Rendering tidak terjadi di tengah listener sinkron yang sedang berjalan.
- Performance implication:
  - Listener berat pada event frekuensi tinggi (scroll, pointermove) dapat menyebabkan jank.
  - Delegasi event pada ancestor dapat mengurangi jumlah listener dan biaya setup.
- Memory implication:
  - Listener yang menutup closure besar atau tidak dibersihkan saat node tak dipakai dapat menahan objek tetap reachable lebih lama.
- Security implication:
  - Event propagation tidak melanggar same-origin boundary antar dokumen berbeda origin; akses data tetap dibatasi oleh model keamanan DOM/web.

## 5. Kenapa API Ini Ada?

Web butuh model event yang:

- Konsisten untuk berbagai tipe input dan custom event.
- Mendukung komponen bertingkat (nested DOM trees).
- Memberi kontrol interception (capturing), handling lokal (target), dan reaksi ancestor (bubbling).

Tanpa model propagasi standar, interaksi antarkomponen UI akan sulit diprediksi dan sulit dikomposisikan.

## 6. Contoh Minimal

```html
<div id="outer">
  <button id="btn">Click</button>
</div>
<script>
  outer.addEventListener("click", () => console.log("outer bubble"));
  outer.addEventListener("click", () => console.log("outer capture"), true);
  btn.addEventListener("click", () => console.log("button target"));
</script>
```

Urutan log ketika tombol diklik:

1. `outer capture`
2. `button target`
3. `outer bubble`

## 7. Common Misconceptions

- "Event selalu bubbling."
  - Koreksi: tidak semua event `bubbles`; beberapa event tidak naik ke ancestor.
- "capture dan bubble berjalan bersamaan."
  - Koreksi: keduanya fase berbeda dengan urutan deterministik.
- "stopPropagation menghentikan default action."
  - Koreksi: default action dibatalkan dengan `preventDefault()` pada event yang cancelable.

## 8. Pitfall & Best Practices

- Pitfall: memasang listener terlalu banyak di banyak node.
  - Dampak: overhead memori dan maintenance tinggi.
  - Praktik: gunakan event delegation pada ancestor yang stabil.
- Pitfall: memakai `stopPropagation` secara agresif.
  - Dampak: integrasi antar komponen mudah rusak.
  - Praktik: hentikan propagasi hanya jika ada alasan kontrak perilaku yang jelas.
- Pitfall: lupa opsi listener (`passive`, `once`, `capture`) sesuai kebutuhan.
  - Dampak: performa buruk atau perilaku tidak sesuai ekspektasi.
  - Praktik: set opsi eksplisit untuk event kritikal.

## 9. Prerequisite

- DOM tree fundamentals.
- Event loop dasar.
- Task vs microtask basics.

## 10. Next Topics

- Shadow DOM event retargeting and composed path
- Default action and cancelation model
- Pointer and keyboard event specifics

## References

- DOM Standard (events): https://dom.spec.whatwg.org/#events
- DOM Standard (dispatching events): https://dom.spec.whatwg.org/#concept-event-dispatch
- HTML Standard (event handlers integration): https://html.spec.whatwg.org/
