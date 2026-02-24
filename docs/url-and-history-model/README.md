# URL and History Model

## 1. Official Term

URL Standard parsing/serialization model and History API (`pushState`, `replaceState`, `popstate`).

## 2. Definisi Formal

URL model mendefinisikan cara browser mem-parse, memvalidasi, menormalkan, dan men-serialize URL berdasarkan URL Standard. History model (HTML Standard) mendefinisikan session history entries, navigasi, serta manipulasi state melalui History API.

Keduanya adalah bagian Web Platform, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Ada dua aspek:

1. URL object model: representasi terstruktur (`protocol`, `host`, `pathname`, `search`, `hash`) dengan aturan parse/serialize standar.
2. Session history stack: daftar entri navigasi per browsing context yang bisa ditambah/diubah oleh `pushState`/`replaceState`.

`pushState` menambah entri history baru tanpa reload dokumen. `popstate` terjadi saat pengguna/browser bergerak antar entri.

## 4. Runtime Perspective

- Thread execution:
  - Operasi URL/History API dari script dokumen berjalan di main thread context.
- Queue:
  - Pemanggilan sinkron (`new URL`, `history.pushState`) terjadi langsung pada task saat ini.
  - Event navigasi seperti `popstate` dideliver sebagai task pada event loop dokumen.
- Scheduler:
  - Browser scheduler menangani transisi history, dispatch event navigasi, dan sinkronisasi UI browser chrome (address bar/back-forward state).
- Rendering timing:
  - `pushState`/`replaceState` tidak otomatis trigger full document reload, tetapi perubahan UI aplikasi setelahnya tetap ikut frame scheduling normal.
  - Navigasi full-document mengikuti lifecycle parsing/loading terpisah.
- Performance implication:
  - Manipulasi history berfrekuensi tinggi tanpa kontrol dapat membebani state management aplikasi.
  - Parsing URL berulang untuk data sama menambah overhead tidak perlu.
- Memory implication:
  - State object besar pada `pushState` dapat meningkatkan konsumsi memori session history.
  - Riwayat panjang dengan payload state besar memperbesar footprint proses renderer/browser.
- Security implication:
  - Same-origin restriction berlaku untuk URL tertentu pada History API.
  - URL dapat memuat data sensitif jika query/hash dipakai sembarangan; ini meningkatkan risiko eksposur melalui log/referrer.

## 5. Kenapa API Ini Ada?

Web app modern membutuhkan:

- Navigasi yang tetap sinkron dengan tombol back/forward browser.
- Deep-linking berbasis URL untuk state aplikasi.
- Manipulasi URL terstandar lintas browser.

History API dan URL Standard memberi kontrak konsisten agar aplikasi dapat mengelola navigasi tanpa mengorbankan model browser native.

## 6. Contoh Minimal

```html
<script>
  const url = new URL(location.href);
  url.searchParams.set("tab", "settings");
  history.pushState({ tab: "settings" }, "", url);

  window.addEventListener("popstate", (event) => {
    console.log("restored state:", event.state);
  });
</script>
```

Contoh ini memperbarui URL dan menambah history entry tanpa full reload.

## 7. Common Misconceptions

- "History API adalah fitur framework router."
  - Koreksi: router framework dibangun di atas History API Web Platform.
- "`pushState` memicu `popstate` langsung."
  - Koreksi: `popstate` dipicu saat berpindah ke entri lain (misalnya back/forward), bukan saat `pushState` dipanggil.
- "Semua URL string valid otomatis aman dipakai."
  - Koreksi: parsing/normalisasi penting; data URL tetap harus divalidasi sesuai konteks aplikasi.

## 8. Pitfall & Best Practices

- Pitfall: menyimpan state terlalu besar di `pushState`.
  - Dampak: konsumsi memori naik, potensi ketidakstabilan state restore.
  - Praktik: simpan state minimal; data besar simpan di store terpisah.
- Pitfall: tidak menangani `popstate`.
  - Dampak: UI dan URL/history bisa tidak sinkron.
  - Praktik: definisikan mekanisme restore state yang deterministik.
- Pitfall: menaruh data sensitif di query/hash.
  - Dampak: kebocoran via log, analytics, referrer, screenshot.
  - Praktik: hindari URL sebagai media penyimpanan rahasia.

## 9. Prerequisite

- Event loop basics.
- Same-Origin Policy fundamentals.
- Dasar navigation lifecycle dokumen.

## 10. Next Topics

- [Navigation API and modern navigation interception](../next-topics/navigation-api-and-modern-navigation-interception/README.md)
- [bfcache and lifecycle interaction with history traversal](../next-topics/bfcache-and-lifecycle-interaction-with-history-traversal/README.md)
- [Security Boundary](../security-boundary/README.md)

## References

- URL Standard: https://url.spec.whatwg.org/
- HTML Standard (History API): https://html.spec.whatwg.org/multipage/nav-history-apis.html
- HTML Standard (session history): https://html.spec.whatwg.org/multipage/history.html


