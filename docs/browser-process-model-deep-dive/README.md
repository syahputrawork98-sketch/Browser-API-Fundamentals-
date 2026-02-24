# Browser Process Model Deep Dive

## 1. Official Term

Browser process model (browser process, renderer process, GPU process, network/service processes, site isolation).

## 2. Definisi Formal

Browser process model adalah arsitektur multiprocess pada browser modern yang memisahkan komponen utama (UI/browser control, rendering dokumen, GPU compositing, networking/service) ke process berbeda untuk stabilitas, keamanan, dan performa.

Ini adalah detail implementasi browser engine/runtime platform, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Alih-alih satu proses monolitik, browser membagi tanggung jawab:

1. Browser process mengelola tab, navigasi, policy, permission, dan koordinasi global.
2. Renderer process mengeksekusi dokumen web (DOM, JS, layout sebagian besar logic dokumen).
3. GPU process menangani compositing/raster path tertentu.
4. Process lain (tergantung browser) menangani network, storage service, utility tasks.

Site isolation membatasi halaman lintas-site ke process berbeda untuk mengurangi dampak eksploitasi.

## 4. Runtime Perspective

- Thread execution:
  - JavaScript dokumen berjalan di renderer process (umumnya main thread renderer untuk context utama).
  - Browser process dan process lain menangani orkestrasi/navigation/security checks di luar thread JS dokumen.
- Queue:
  - Masing-masing process memiliki event loop/queue internal sendiri; komunikasi antarprocess dilakukan via IPC messages.
  - Event yang terlihat di script tetap muncul sebagai task/microtask di event loop context script.
- Scheduler:
  - OS scheduler + scheduler internal browser menentukan alokasi CPU antar process/thread.
  - Browser scheduler juga menentukan prioritas tab visible/background dan resource scheduling.
- Rendering timing:
  - Renderer menyiapkan update visual, lalu compositing path melibatkan pipeline antar thread/process (termasuk GPU path).
  - Kontensi resource antar process dapat memengaruhi frame delivery.
- Performance implication:
  - Multiprocess meningkatkan isolasi fault dan responsivitas global, tapi menambah overhead IPC/context switching.
  - Site isolation menambah keamanan dengan potensi biaya memori/process count lebih tinggi.
- Memory implication:
  - Tiap process memiliki overhead memori sendiri; banyak tab/site dapat meningkatkan total footprint signifikan.
  - Isolasi process mengurangi shared-state risiko namun tidak gratis dari sisi konsumsi RAM.
- Security implication:
  - Process boundary + sandboxing + site isolation membatasi blast radius ketika renderer dikompromi.
  - Kebijakan origin/site lebih kuat ketika digabung dengan isolasi process dan permission enforcement di browser process.

## 5. Kenapa API Ini Ada?

Ini bukan API tunggal, tetapi fondasi implementasi browser modern untuk memenuhi tiga target:

- Robustness: crash satu renderer tidak menjatuhkan seluruh browser.
- Security: sandbox dan isolation membatasi lateral movement exploit.
- Performance: pemisahan kerja memungkinkan scheduling lebih efektif lintas komponen.

Memahami model ini penting untuk menjelaskan kenapa behavior runtime web tidak cukup dilihat dari JavaScript engine saja.

## 6. Contoh Minimal

```text
User click link
-> Browser process memutuskan navigation policy
-> Renderer process target disiapkan/di-switch
-> Network/service path memuat resource
-> Renderer parse + execute script + layout
-> GPU/compositor path menampilkan frame
```

Contoh alur ini menunjukkan banyak tahap terjadi lintas process, bukan satu thread JavaScript.

## 7. Common Misconceptions

- "Satu tab = satu proses selalu."
  - Koreksi: mapping tab/site/proses bergantung kebijakan browser (site isolation, process reuse, dll).
- "JavaScript engine mengontrol semua bagian browser."
  - Koreksi: JS engine hanya satu komponen di renderer; navigasi/network/security melibatkan komponen lain.
- "Multiprocess hanya soal performa."
  - Koreksi: alasan keamanan dan stabilitas sama pentingnya.

## 8. Pitfall & Best Practices

- Pitfall: analisis performa hanya dari timeline JavaScript.
  - Dampak: bottleneck IPC/network/compositor terlewat.
  - Praktik: gunakan tooling yang menampilkan pipeline lintas komponen.
- Pitfall: mengabaikan efek background throttling/process priority.
  - Dampak: asumsi timing salah pada tab non-visible.
  - Praktik: uji skenario visible vs background secara eksplisit.
- Pitfall: desain aplikasi yang memaksa kerja besar di renderer utama.
  - Dampak: interaksi UI mudah jank.
  - Praktik: gunakan worker, batching, dan strategi scheduling sesuai workload.

## 9. Prerequisite

- Event loop and scheduling fundamentals.
- Rendering pipeline interaction.
- Security boundary fundamentals.

## 10. Next Topics

- [Security Boundary](../security-boundary/README.md)
- [Web Workers and Message Passing](../web-workers-and-message-passing/README.md)
- [Rendering Pipeline Interaction](../rendering-pipeline-interaction/README.md)

## References

- Chromium multi-process architecture overview: https://www.chromium.org/developers/design-documents/multi-process-architecture/
- Site Isolation (Chromium docs): https://www.chromium.org/Home/chromium-security/site-isolation/
- HTML Standard (high-level integration): https://html.spec.whatwg.org/


