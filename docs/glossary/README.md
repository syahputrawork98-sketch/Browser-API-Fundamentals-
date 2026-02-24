# Glossary Istilah Wajib

Glossary ini adalah referensi terminologi wajib agar semua dokumen konsisten saat membahas batas antara language, API platform, dan implementasi browser.

## 1. ECMAScript (Language Specification)

Definisi:
- Spesifikasi bahasa yang dikelola TC39.
- Mendefinisikan syntax, semantics, dan built-in language objects.

Contoh yang termasuk:
- `Promise`
- `Array`, `Map`, `Set`
- `async`/`await`

Contoh yang tidak termasuk:
- `fetch`
- DOM
- `setTimeout`
- Event loop host

Terminologi wajib:
- Gunakan `ECMAScript` untuk spesifikasi bahasa.
- Hindari istilah `JavaScript punya Web API`.

## 2. JavaScript Engine

Definisi:
- Mesin yang mengimplementasikan ECMAScript (misalnya V8, SpiderMonkey, JavaScriptCore).

Tanggung jawab:
- Parsing, bytecode/JIT, execution semantics ECMAScript.
- Garbage collection untuk objek bahasa.

Batas:
- Tidak mendefinisikan network stack browser.
- Tidak mendefinisikan DOM standard behavior.

## 3. Web API / Web Platform API

Definisi:
- API host yang disediakan browser dan didefinisikan oleh standar web (WHATWG/W3C).

Contoh:
- DOM API (`document`, `Element`, event dispatch)
- Fetch API (`fetch`, `Request`, `Response`)
- Timers (`setTimeout`, `setInterval`)
- Web Workers, Service Worker, Cache Storage, Streams

Terminologi wajib:
- Sebut `Web API` atau `Web Platform API`, bukan fitur bahasa ECMAScript.

## 4. Browser Engine / User Agent Implementation

Definisi:
- Implementasi nyata yang menggabungkan JavaScript engine + rendering engine + subsystems browser.

Komponen umum:
- Renderer pipeline (style, layout, paint, composite)
- Process model (browser process, renderer process, GPU process)
- Security enforcement (SOP, CORS checks, sandboxing)

Catatan:
- Sebagian behavior adalah detail implementasi, bukan API language.

## 5. Event Loop (HTML Standard)

Definisi:
- Model scheduling host di HTML Standard untuk task, microtask checkpoint, dan rendering opportunity.

Terminologi wajib:
- Gunakan `event loop (HTML Standard)`.
- Hindari `JavaScript event loop` jika maksudnya model host browser.

## 6. Rendering Pipeline

Definisi:
- Pipeline visual browser: style -> layout -> paint -> composite.

Batas:
- Rendering pipeline adalah bagian engine browser, bukan bagian spesifikasi ECMAScript.

## 7. Security Boundary

Definisi:
- Lapisan pembatas akses kemampuan/data berbasis origin, context keamanan, dan izin.

Istilah kunci:
- Same-Origin Policy (SOP)
- CORS
- Secure Context
- Permissions
- COOP/COEP, CSP, Trusted Types

## 8. Kalimat Koreksi Standar

Gunakan kalimat ini saat menemukan miskonsepsi:
- `fetch adalah Web API (Fetch Standard), bukan fitur bahasa ECMAScript.`
- `DOM adalah Web Platform API, bukan bagian dari JavaScript language specification.`
- `setTimeout didefinisikan HTML Standard (timers), bukan ECMAScript.`
- `Event loop browser adalah model host di HTML Standard.`

## 9. Referensi

- ECMAScript: https://tc39.es/ecma262/
- HTML Standard: https://html.spec.whatwg.org/
- DOM Standard: https://dom.spec.whatwg.org/
- Fetch Standard: https://fetch.spec.whatwg.org/
- Streams Standard: https://streams.spec.whatwg.org/
