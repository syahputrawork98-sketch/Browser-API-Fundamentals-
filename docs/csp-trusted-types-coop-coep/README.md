# CSP, Trusted Types, and COOP/COEP

## 1. Official Term

Content Security Policy (CSP), Trusted Types, Cross-Origin-Opener-Policy (COOP), and Cross-Origin-Embedder-Policy (COEP).

## 2. Definisi Formal

- CSP adalah kebijakan berbasis header/meta untuk membatasi sumber konten dan perilaku eksekusi tertentu (script, style, frame, connect, dll).
- Trusted Types adalah mekanisme mitigasi DOM XSS sink tertentu dengan memaksa value berasal dari policy yang terkontrol.
- COOP/COEP adalah pasangan header isolasi lintas-origin untuk membentuk cross-origin isolated context.

Semua ini adalah kebijakan keamanan Web Platform, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Lapisan pertahanan:

1. CSP membatasi apa yang boleh dimuat/dieksekusi.
2. Trusted Types memperketat bagaimana string berbahaya masuk ke DOM sinks sensitif.
3. COOP/COEP mengisolasi browsing context dari cross-origin window/resource yang tidak memenuhi syarat.

Gabungan ketiganya mengurangi kelas serangan injeksi dan kebocoran lintas-context.

## 4. Runtime Perspective

- Thread execution:
  - Enforcement policy dilakukan oleh browser subsystems (network/parser/renderer integration), bukan oleh JavaScript engine saja.
- Queue:
  - Pelanggaran policy dapat muncul sebagai error/event report asinkron yang diproses pada event loop context terkait.
  - Script blocked oleh policy tidak masuk jalur eksekusi normal.
- Scheduler:
  - Browser mengevaluasi header policy pada tahap load/navigation/resource fetch sebelum resource dipakai runtime.
- Rendering timing:
  - Resource/script yang diblok policy dapat mengubah jalur render (misalnya script bootstrap tidak jalan, style tidak termuat).
  - Reporting/violation handling biasanya tidak dominan ke frame timing, tetapi dapat menambah overhead observability.
- Performance implication:
  - CSP/COOP/COEP bisa menambah kompleksitas deployment dan potensi preflight/compat checks.
  - Konfigurasi tepat meningkatkan keamanan tanpa overhead signifikan di jalur steady-state.
- Memory implication:
  - Isolasi context (termasuk cross-origin isolation) dapat mengubah model alokasi proses/resource dan menambah overhead tertentu.
  - Instrumentasi violation report berlebihan dapat menambah usage memori/log pipeline.
- Security implication:
  - CSP + Trusted Types menurunkan risiko DOM/script injection.
  - COOP/COEP memperkuat boundary process/context dan membuka fitur yang butuh isolasi kuat (misalnya `SharedArrayBuffer`).

## 5. Kenapa API Ini Ada?

Threat model web modern mencakup XSS, supply chain script risk, dan cross-origin data leakage. Kebijakan ini ada untuk membuat prinsip least-privilege bisa diterapkan di level platform, bukan sekadar konvensi kode aplikasi.

## 6. Contoh Minimal

```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

```http
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types appPolicy
```

Contoh di atas menunjukkan arah kebijakan minimum: membatasi sumber script dan mengaktifkan isolasi yang lebih ketat.

## 7. Common Misconceptions

- "CSP cukup pakai sekali, selesai."
  - Koreksi: policy perlu iterasi, monitoring, dan penyesuaian saat arsitektur berubah.
- "Trusted Types menggantikan sanitasi input."
  - Koreksi: Trusted Types melengkapi, bukan menggantikan validasi/sanitasi.
- "COOP/COEP hanya untuk performa."
  - Koreksi: tujuan utamanya isolasi keamanan; akses fitur tertentu hanyalah konsekuensi capability model.

## 8. Pitfall & Best Practices

- Pitfall: policy terlalu longgar (misalnya wildcard berlebih).
  - Dampak: proteksi melemah.
  - Praktik: mulai strict baseline, tambah exception seminimal mungkin.
- Pitfall: rollout langsung enforcement tanpa report phase.
  - Dampak: breakage produksi sulit dipetakan.
  - Praktik: gunakan report-only terlebih dahulu untuk observasi.
- Pitfall: mengaktifkan COOP/COEP tanpa audit dependency pihak ketiga.
  - Dampak: resource third-party gagal dimuat/berfungsi.
  - Praktik: audit seluruh chain resource dan header compatibility sebelum enforcement.

## 9. Prerequisite

- [Security Boundary](../security-boundary/README.md)
- [Browser Process Model Deep Dive](../browser-process-model-deep-dive/README.md)
- Dasar HTTP headers dan deployment pipeline

## 10. Next Topics

- [Fetch Metadata Request Headers](../next-topics/fetch-metadata-request-headers/README.md)
- [Subresource Integrity (SRI) strategy](../next-topics/subresource-integrity-sri-strategy/README.md)
- [Origin-Agent-Cluster and isolation tuning](../next-topics/origin-agent-cluster-and-isolation-tuning/README.md)

## References

- Content Security Policy Level 3: https://w3c.github.io/webappsec-csp/
- Trusted Types specification: https://w3c.github.io/trusted-types/dist/spec/
- COOP (HTML Standard): https://html.spec.whatwg.org/multipage/origin.html#cross-origin-opener-policies
- COEP (HTML Standard): https://html.spec.whatwg.org/multipage/origin.html#coep


