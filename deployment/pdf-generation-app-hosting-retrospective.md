# PDF generation on Firebase App Hosting — incident retrospective

This document records a multi-stage production failure affecting **quote/invoice PDF generation and document email** (`POST /api/documents/email` returning **HTTP 500**), why each failure mode happened on **Firebase App Hosting** (Google Cloud Run + Next.js standalone), and how the team resolved it **without switching hosting platforms**.

Readers who maintain workflow documents, deployments, or `lib/documents/` should skim at least **Executive summary**, **Constraints**, and **Current architecture**.

---

## Executive summary

| Phase | Symptom (user-visible or log) | Root cause | Resolution |
|-------|------------------------------|------------|------------|
| 1 | `Could not find Chrome` pointing at **`.../.cache/puppeteer`** under `www-data` | Full **`puppeteer`** reads `PUPPETEER_CACHE_DIR` **once at package import**. Setting env later in route code ran **too late**. | **`instrumentation.ts`** to set **`PUPPETEER_CACHE_DIR`** before **`import "puppeteer"`**. (Superseded by Phase 7.) |
| 2 | `libglib-2.0.so.0`: cannot open shared object file | **Puppeteer’s Chrome** expects **glib** and other desktop libraries not present on the **minimal Linux** App Hosting runtime image | Switched runtime path toward **Linux** + **`@sparticuz/chromium`** (still Chromium). Superseded. |
| 3 | Brotli / “input directory `.../@sparticuz/chromium/bin` does not exist” | **Next.js standalone** file tracing omitted **`bin/*.br`** (not visible to static tracer) | **`outputFileTracingIncludes`** for `./node_modules/@sparticuz/chromium/bin/**/*` in **`next.config.ts`**. Superseded. |
| 4 | `libnspr4.so`: cannot open shared object file (**code 127**) | Sparticuz’s extracted Chromium **still dynamically links NSS** libraries the image does not ship | **Removed headless Chromium** from the server PDF path entirely. |
| 5 | `ENOENT`: open `Helvetica.afm` under **`pdfkit/js/data`** | Same standalone tracing gap: **`fs.readFileSync(__dirname + '/data/…')`** is invisible to NFT | **`outputFileTracingIncludes`** for **`./node_modules/pdfkit/js/data/**/*`** + **`pdfkit`** in **`serverExternalPackages`**. |
| 6 | Git **`HTTP 408`** on push | **`.puppeteer-chrome/`** (full Windows Chromium) was accidentally **committed** after `.gitignore` dropped that entry — **~hundreds of MB** push timed out | Restored `.gitignore`, **`git rm --cached .puppeteer-chrome`**, amended commit |

**Stable end state**: PDFs for quotes/invoices/etc. are built with **`pdfkit`** from **`DocumentTemplateData`** (`lib/documents/render-pdf-document.ts` in the app repo). No Puppeteer/Chrome in production. **`next.config.ts`** includes explicit tracing rules for **`pdfkit`** data files.

---

## Product and code context

### What broke

Sending a quote (or generating a workflow PDF attached to email) calls the Next.js route **`/api/documents/email`** (see `app/api/documents/email/route.ts` in the app repo). That flow:

1. Loads team/quote (or invoice) data from Firestore.
2. Builds a **`DocumentTemplateData`** payload.
3. Renders a **PDF buffer** server-side.
4. Sends email (e.g. Resend) with the PDF attachment.

Any uncaught exception in that pipeline becomes **`500`** with `error.message` in the JSON body; the browser shows a failed **`fetch`** on `/api/documents/email`.

### Relevant modules (app repository)

Historical and current hooks (exact filenames may drift; grep if needed):

| Concern | Location (typical) |
|---------|---------------------|
| Email API route | `app/api/documents/email/route.ts` |
| Orchestration + storage upload | `lib/documents/generate.ts` |
| HTML templates (still used where HTML is needed; PDF path may bypass) | `lib/documents/templates/shared.ts`, `templates/*.tsx` |
| Current PDF renderer | `lib/documents/render-pdf-document.ts` |
| Production build entry | `npm run build` → `scripts/run-next-build.cjs` |
| App Hosting rollout config | `apphosting.yaml` |
| Standalone / tracing config | `next.config.ts` (`output: 'standalone'`, **`outputFileTracingIncludes`**) |

---

## Why Firebase App Hosting is “hard mode” for Puppeteer/Chromium

### Runtime model

Firebase App Hosting builds a container (Cloud Native Buildpacks) and runs the app on **Cloud Run**. The filesystem layout often includes a **standalone** bundle such as **`/workspace/.next/standalone`**, mirroring **`output: 'standalone'`** in Next.js.

### Why “Chrome in Node” fights the platform

1. **Thin base image**: The Node runtime image intentionally omits GUI stack libraries (glib, nss, atk, fonts, …) that desktop **Chromium or Chrome-for-Testing** expect.
2. **Bundlers don’t prove runtime assets**: Dependencies that load files via **`path.join(__dirname, 'something')`** or **`fs.readFileSync(dynamicPath)`** are easy to miss in **NFT** tracing — those files never reach **`node_modules`** in the trimmed artifact.
3. **Puppeteer config is import-time frozen**: `puppeteer` (non-core) invokes **`getConfiguration()`** immediately when its entry module loads, binding **`cacheDirectory`** to **`HOME/.cache/puppeteer`** unless **`PUPPETEER_CACHE_DIR`** is already set. Route-level env mutation does not retroactively change that frozen config.

Together, “install Chrome in CI and ship it locally under **`.puppeteer-chrome`**” still failed in production whenever **(a)** the binary expected host libraries the image lacked, or **(b)** Next dropped parts of **`node_modules`** from the standalone output.

---

## Phase-by-phase chronology

### Phase 1 — “Chrome not found” and `/www-data-home/.cache/puppeteer`

**Observed**: Error text referenced **`Could not find Chrome`** and **`/www-data-home/.cache/puppeteer`**.

**Why**: Puppeteer resolves the browser bundle under **`$HOME/.cache/puppeteer`** unless overridden at **import**. The runtime user (`www-data`-style home paths) did not contain a populated cache; our code attempted to assign **`process.env.PUPPETEER_CACHE_DIR`** only inside **`renderPdfFromHtml`**, **after** the static `import "puppeteer"` had already run.

**Direction taken**: **`instrumentation.ts`** with **`register()`** guarded for Edge, setting **`PUPPETEER_CACHE_DIR`** early on Node. **`run-next-build.cjs`** downloaded Chrome into a project-local `.puppeteer-chrome/` on non-Linux builders.

This fixed “wrong HOME” but exposed the next blocker on App Hosting Linux.

---

### Phase 2 — Missing `libglib-2.0.so.0` (stock Puppeteer Chrome)

**Observed**: Browser path pointed at **`.../.puppeteer-chrome/chrome/linux-.../chrome`**, followed by **`error while loading shared libraries: libglib-2.0.so.0`**.

**Why**: The Puppeteer-managed Chrome tarball is built for mainstream Linux desktops. The minimal container does not ship **glib** or the broader dependency set.

**Mitigation explored**: APT packages in Dockerfile — standard App Hosting does **not** expose arbitrary **`apt-get`** unless you operate a fully custom deployment path outside default buildpacks. So patching the OS repeatedly was brittle.

---

### Phase 3 — Sparticuz Chromium + standalone `bin/` omission

**Observed**: “The input directory `.../@sparticuz/chromium/bin` does not exist. Please provide the location of the brotli files.”

**Why**: **`@sparticuz/chromium`** ships **`.br`** archives under **`node_modules/@sparticuz/chromium/bin/`**. Next standalone tracing followed JS imports only; it did **not** copy **`bin/`** into **`.next/standalone`**.

**Fix applied at the time**: In **`next.config.ts`**, **`outputFileTracingIncludes`** with:

```javascript
outputFileTracingIncludes: {
  '/*': ['./node_modules/@sparticuz/chromium/bin/**/*'],
},
```

---

### Phase 4 — Missing `libnspr4.so` (`/tmp/chromium`, exit 127)

**Observed**: `Failed to launch the browser process: Code: 127` with **`libnspr4.so`** (NSS).

**Why**: Sparticuz’s extracted Chromium executable still **ELF-links** core system libraries (**NSS**) that the App Hosting runtime image lacks. Sparticuz is commonly paired with fuller layers (Lambda runtimes tuned for Chromium, Docker images that install NSS, etc.). The default Firebase App Hosting runner is still too thin without **vendor-controlled OS packages**.

**Strategic pivot**: Eliminate Chromium from PDF generation entirely for this codebase.

---

### Phase 7 (final PDF approach) — **PDFKit**, no browser

**Design**: Reimplement server PDF output using **`pdfkit`**, consuming the same **`DocumentTemplateData`** that previously fed HTML templates. Fonts use PDF built-ins (Helvetica family) rendered via PDFKit’s standard font metrics (**AFM**) files shipped in **`node_modules/pdfkit/js/data/`**.

**Why this fits App Hosting**:

- Pure Node buffers; predictable cold starts versus decompressing **`chromium.br`**.
- Network use limited to **`fetch`** for logos/signatures embedded in templates (existing pattern).

Trade-off: Pixel parity with Chrome-printed HTML is **not guaranteed** — layout logic lives in **`render-pdf-document.ts`** and should be updated intentionally when **`shared.ts`** HTML changes materially.

---

### Phase 8 — ENOENT **`Helvetica.afm`** (`/ROOT/node_modules/pdfkit/...`)

**Observed**: `ENOENT ... open '.../node_modules/pdfkit/js/data/Helvetica.afm'`.

**Why**: Same class of bug as Sparticuz: PDFKit resolves metrics with:

```javascript
fs.readFileSync(path.join(__dirname, 'data/Helvetica.afm'), 'utf8');
```

Standalone tracing copied **`pdfkit.js`** without **`js/data/**/*`**.

**Fix**:

```javascript
outputFileTracingIncludes: {
  '/*': ['./node_modules/pdfkit/js/data/**/*'],
},
serverExternalPackages: [
  // ...existing
  'pdfkit',
],
```

After **`npm run build`**, **`Helvetica.afm`** must exist under **`.next/standalone/node_modules/pdfkit/js/data/`** on disk.

Also include **`*.icc`** paths if color profiles hit at runtime (**`sRGB_IEC61966_2_1.icc`** in the same **`data`** tree).

---

## Phase 9 — Git `HTTP 408` (deployment blocked)

**Observed**: `Git: RPC failed; HTTP 408 ... The requested URL returned error: 408`.

**Cause**: Removing **`.puppeteer-chrome/`** from `.gitignore` briefly allowed **`git add -A`** to stage Puppeteer’s full Windows Chromium tree (**`chrome.dll` alone ~280 MiB**, plus hundreds of ancillary files).

**Remediation**:

1. Add **`.puppeteer-chrome/`** back to **`.gitignore`**.
2. `git rm -r --cached .puppeteer-chrome`
3. `git commit --amend` **or** a follow-up cleanup commit restoring small diff only.

Operational note: If a bad commit **had** landed on **`main`**, you would purge history (**`git filter-repo`**, GitHub rewrite) — treat as security/size incident.

---

## Current deployment checklist for PDF touches

Before merging changes that alter PDF output or fonts:

1. **Run `npm run build`** locally or in CI with **`output: 'standalone'`** enabled.
2. **Verify traced assets** exist under `.next/standalone/node_modules/pdfkit/js/data/` (**`Helvetica*.afm`**, ICC if used).
3. **Avoid committing** toolchain caches (**`.puppeteer-chrome`**, `.cache`, Puppeteer remnants) — keep them in `.gitignore`.
4. **Watch App Hosting logs** after rollout for ENOENT paths — any new **`fs.readFileSync` under `node_modules`** likely needs **`outputFileTracingIncludes`** mirroring Sparticuz/PDFKit lessons.
5. **Memory**: Chromium formerly needed headroom; **PDFKit + fetch** uses less RAM, but **`apphosting.yaml`** **`runConfig.memoryMiB`** remains a tuning knob (**1024 MiB** was justified during Chromium experimentation).

---

## Lessons learned

| Lesson | Detail |
|--------|--------|
| **Static import side effects bite** | Never assume **`process.env` in route handlers** affects packages that freeze config **at import**. |
| **Filesystem ≠ graph** | **NFT** skips dynamic relative reads. Maintain explicit **`outputFileTracingIncludes`** for **`node_modules` data subtrees.** |
| **“Serverless Chrome” ≠ “no distro deps”** | Many Chromium builds still **`dlopen`/link NSS/glib**. Verify **minimal** images or avoid browsers. |
| **Watch git scope** | Dropping **`gitignore`** lines on cache directories risked catastrophic repo bloat (`HTTP 408`, storage costs, poor clone UX). |

---

## Related documentation

- [Firebase App Hosting config](./firebase-app-hosting.md) — `apphosting.yaml`, env vars, rollout context.
- [Next.js standalone / output tracing](https://nextjs.org/docs/app/api-reference/config/next-config-js/output) — authoritative reference for **`outputFileTracingIncludes`**.
- Puppeteer troubleshooting (historical reference only): https://pptr.dev/troubleshooting

---

## Changelog cue

Major decisions in **2026 Q2** for **app.groundzy.com** deployments: phased removal of Puppeteer/Sparticuz layers; **`pdfkit`** PDF output; **`next.config`** tracing for **`pdfkit/js/data`**; `.gitignore` restoration for Puppeteer caches; **`apphosting.yaml`** memory budget updated during migration.
