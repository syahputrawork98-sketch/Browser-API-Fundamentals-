# DOM Standard Fundamentals

## 1. Official Term

Document Object Model (DOM) Standard: node tree, document, element, attributes, and mutation model.

## 2. Definisi Formal

DOM Standard mendefinisikan representasi objek terstruktur untuk dokumen (node tree), API manipulasi struktur tersebut, serta algoritma operasi seperti insertion, removal, cloning, dan traversal.

DOM bukan bagian dari ECMAScript language specification. ECMAScript menyediakan bahasa eksekusi, sedangkan DOM menyediakan host objects dan algoritma tree manipulation pada Web Platform.

## 3. Mental Model

DOM adalah graph/tree objek hidup yang merepresentasikan dokumen saat runtime:

1. Parser membangun node tree dari markup.
2. Script membaca/memodifikasi tree melalui API DOM.
3. Perubahan tree bisa memicu invalidasi style/layout pada rendering pipeline.

DOM bukan sekadar "HTML string"; ini model state terstruktur yang dipakai engine browser untuk interaksi, event, dan rendering.

## 4. Runtime Perspective

- Thread execution:
  - Pada dokumen utama, operasi DOM umumnya berjalan di main thread renderer.
- Queue:
  - Operasi DOM sinkron yang dipanggil script berjalan dalam task saat ini.
  - Side effects asinkron tertentu (misalnya observer callback) diantrekan sesuai model scheduling host.
- Scheduler:
  - Event loop host menentukan kapan task dieksekusi; algoritma DOM menentukan efek mutasi dalam task tersebut.
- Rendering timing:
  - Mutasi DOM tidak otomatis langsung paint saat statement itu dieksekusi.
  - Browser melakukan style/layout/paint pada rendering opportunity yang sesuai.
- Performance implication:
  - Mutasi DOM besar dan query layout sinkron berulang dapat menaikkan biaya frame.
  - Batch perubahan mengurangi invalidasi berulang.
- Memory implication:
  - Node yang masih direferensikan (termasuk detached subtree) tetap reachable dan tidak dibersihkan GC.
  - Event listener/closure pada node dapat memperpanjang lifetime objek terkait.
- Security implication:
  - DOM API tunduk pada same-origin policy antar dokumen; akses lintas-origin ke dokumen/iframe dibatasi model keamanan browser.

## 5. Kenapa API Ini Ada?

Web butuh model dokumen yang:

- Terstandar lintas browser.
- Bisa dimanipulasi dinamis oleh script.
- Mendukung event, styling, dan rendering secara konsisten.

DOM menyediakan kontrak interoperabilitas antara parser, script engine, dan rendering engine.

## 6. Contoh Minimal

```html
<ul id="list"></ul>
<script>
  const list = document.getElementById("list");
  const item = document.createElement("li");
  item.textContent = "DOM node baru";
  list.appendChild(item);
</script>
```

Contoh ini memodifikasi node tree runtime, bukan memodifikasi source HTML file.

## 7. Common Misconceptions

- "DOM adalah bagian dari JavaScript."
  - Koreksi: DOM adalah Web API standard, dipakai lewat JavaScript di browser.
- "Mengubah DOM langsung berarti browser langsung paint saat itu juga."
  - Koreksi: paint terjadi sesuai scheduling rendering pipeline.
- "Detached DOM otomatis aman."
  - Koreksi: detached subtree masih bisa menahan memori jika referensinya tetap ada.

## 8. Pitfall & Best Practices

- Pitfall: manipulasi node satu per satu dengan banyak reflow opportunity.
  - Dampak: performa buruk pada tree besar.
  - Praktik: batch update (misalnya `DocumentFragment`) sebelum attach.
- Pitfall: menyimpan referensi node lama tanpa kebutuhan.
  - Dampak: retention memori tidak perlu.
  - Praktik: lepas referensi dan listener saat cleanup.
- Pitfall: membaca layout setelah menulis style berulang-ulang.
  - Dampak: forced synchronous layout.
  - Praktik: pisahkan fase read dan write.

## 9. Prerequisite

- Event loop fundamentals.
- Dasar HTML parsing.
- JavaScript object and reference basics.

## 10. Next Topics

- Event propagation model
- Rendering pipeline interaction
- MutationObserver and structured mutation tracking

## References

- DOM Standard: https://dom.spec.whatwg.org/
- DOM Standard (node tree): https://dom.spec.whatwg.org/#concept-tree
- HTML Standard (parsing and DOM integration): https://html.spec.whatwg.org/
