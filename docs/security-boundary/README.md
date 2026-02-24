# Security Boundary (SOP, CORS, Secure Context, Permissions)

## 1. Official Term

Same-Origin Policy (SOP), Cross-Origin Resource Sharing (CORS), Secure Context, and Permissions API / permission-gated Web Platform APIs.

## 2. Definisi Formal

Security boundary web adalah kumpulan kebijakan platform yang membatasi akses resource dan kemampuan API berdasarkan origin, context keamanan, dan izin pengguna.

- SOP membatasi akses lintas-origin untuk DOM, storage, dan response data.
- CORS adalah mekanisme berbasis HTTP headers untuk mengizinkan pembacaan resource lintas-origin oleh browser secara terkontrol.
- Secure Context mensyaratkan konteks aman (umumnya HTTPS) untuk API sensitif.
- Permissions model mengontrol akses capability tertentu (misalnya geolocation, notifications, camera/microphone) berdasarkan user consent dan policy.

Semua ini adalah kebijakan host environment browser, bukan bagian dari ECMAScript language specification.

## 3. Mental Model

Setiap kali aplikasi web mencoba mengakses data/fitur:

1. Browser mengecek boundary origin dan policy keamanan.
2. Jika lintas-origin, browser menerapkan SOP lalu evaluasi CORS saat relevan.
3. Jika API sensitif, browser cek apakah context aman.
4. Jika API permission-gated, browser cek status izin (granted/prompt/denied).
5. Hanya operasi yang lolos semua batas ini yang bisa lanjut.

Boundary ini adalah "default deny, explicit allow".

## 4. Runtime Perspective

- Thread execution:
  - Enforcement policy terjadi dalam browser networking/security subsystems yang terintegrasi dengan renderer process.
  - Script tetap berjalan di execution context biasa (main thread/worker), tetapi hasil akses ditentukan oleh policy checks host.
- Queue:
  - Request/network events masuk melalui task/event loop; hasil akses (resolve/reject/error surface) diteruskan ke script via callback/promise scheduling.
  - Permission prompts dan hasil interaksi user dipropagasi sebagai event/Promise resolution pada task/microtask yang sesuai.
- Scheduler:
  - Browser scheduler mengelola preflight, permission UI flow, dan delivery hasil policy decision ke context pemanggil.
- Rendering timing:
  - Security checks biasanya tidak langsung terkait paint, tetapi prompt permission/UI interstitial dapat memengaruhi alur interaksi pengguna.
  - Blocking karena network policy failure dapat menunda data yang dibutuhkan sebelum render konten.
- Performance implication:
  - CORS preflight menambah round-trip latency.
  - Permission-gated API flow menambah langkah interaksi pengguna dan potensi penundaan jalur kerja.
- Memory implication:
  - Caching keputusan/policy state dan response metadata menambah overhead internal browser (umumnya kecil dibanding payload aplikasi).
  - Error/retry logic aplikasi yang buruk pada policy denial dapat menumpuk state in-memory tak perlu.
- Security implication:
  - Boundary ini mencegah data exfiltration lintas-origin, menurunkan risiko privilege abuse, dan memaksa explicit trust model.

## 5. Kenapa API Ini Ada?

Web menjalankan code dari origin berbeda dalam lingkungan pengguna yang sama. Tanpa boundary keamanan:

- Situs jahat bisa membaca data situs lain.
- Akses device capability bisa terjadi tanpa kontrol.
- Integritas privasi pengguna runtuh.

SOP, CORS, secure context, dan permissions membentuk lapisan pertahanan inti agar interoperabilitas web tetap aman.

## 6. Contoh Minimal

```html
<script>
  async function demo() {
    try {
      const res = await fetch("https://api.example.com/data", { mode: "cors" });
      console.log("status", res.status);
    } catch (err) {
      // Bisa gagal karena network error atau diblok policy browser.
      console.error("fetch blocked/failed", err);
    }

    if ("permissions" in navigator) {
      const state = await navigator.permissions.query({ name: "geolocation" });
      console.log("geolocation permission:", state.state);
    }
  }

  demo();
</script>
```

Contoh menunjukkan bahwa akses data lintas-origin dan capability device tunduk pada policy browser, bukan hanya logika aplikasi.

## 7. Common Misconceptions

- "CORS melindungi server dari request tidak sah."
  - Koreksi: CORS terutama mengatur apakah browser mengizinkan script membaca response lintas-origin.
- "Kalau request berhasil terkirim, berarti pasti bisa dibaca script."
  - Koreksi: browser tetap bisa memblok akses response karena SOP/CORS.
- "HTTPS hanya untuk enkripsi."
  - Koreksi: HTTPS juga jadi prasyarat banyak API melalui secure context requirement.
- "Permission sekali granted pasti permanen."
  - Koreksi: status izin bisa berubah oleh user/browser policy kapan pun.

## 8. Pitfall & Best Practices

- Pitfall: menganggap CORS sebagai mekanisme autentikasi.
  - Dampak: desain keamanan backend keliru.
  - Praktik: tetap gunakan authn/authz server-side yang benar; CORS hanya salah satu lapisan browser policy.
- Pitfall: wildcard CORS konfigurasi tanpa evaluasi.
  - Dampak: memperluas permukaan akses data lintas-origin.
  - Praktik: batasi origin yang diizinkan sesuai kebutuhan minimum.
- Pitfall: tidak mendesain fallback saat permission denied.
  - Dampak: fitur gagal total dan UX buruk.
  - Praktik: sediakan graceful degradation dan messaging yang jelas.
- Pitfall: menyimpan data sensitif di URL/query saat cross-origin flow.
  - Dampak: risiko kebocoran lewat log/referrer.
  - Praktik: minimalkan data sensitif dan gunakan channel aman yang tepat.

## 9. Prerequisite

- Fetch lifecycle fundamentals.
- Same-origin concept and URL model.
- Dasar HTTP headers dan request/response flow.

## 10. Next Topics

- [Cross-Origin Isolation (COOP/COEP) and SharedArrayBuffer readiness](../next-topics/cross-origin-isolation-coop-coep-and-sharedarraybuffer-readiness/README.md)
- [Content Security Policy (CSP) fundamentals](../next-topics/content-security-policy-csp-fundamentals/README.md)
- [Trusted Types and XSS hardening model](../next-topics/trusted-types-and-xss-hardening-model/README.md)

## References

- Same-Origin Policy (MDN overview): https://developer.mozilla.org/docs/Web/Security/Same-origin_policy
- Fetch Standard (CORS): https://fetch.spec.whatwg.org/
- Secure Contexts specification: https://w3c.github.io/webappsec-secure-contexts/
- Permissions specification: https://w3c.github.io/permissions/


