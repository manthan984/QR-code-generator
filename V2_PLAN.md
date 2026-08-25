# QR Code Generator v2 — Compact QR Plan

## Problem Statement

The current QR code generator (v1) produces large, dense QR codes that are difficult — often impossible — to use on posters, business cards, flyers, stickers, and other physical media where space is limited.

---

## Root-Cause Analysis (Why v1 Codes Are So Big)

After auditing the v1 source code, there are **three compounding factors** that inflate every QR code the app generates:

### 1. `ERROR_CORRECT_H` — Maximum Redundancy (Line 97)

```python
error_correction=qrcode.constants.ERROR_CORRECT_H   # 30% damage recovery
```

Level **H** embeds the most redundancy data of any error-correction tier. It exists so QR codes survive physical damage (scratches, dirt, partial obstruction). For a digital-first or clean-print use case like a poster, this is massive overkill. The extra parity modules push the QR version (grid size) up, making the code significantly denser.

| Level | Recovery | Modules Added | Best For |
|-------|----------|---------------|----------|
| L     | ~7%      | Minimal       | Screens, clean prints |
| M     | ~15%     | Moderate      | General use (default) |
| Q     | ~25%     | High          | Industrial labels |
| **H** | **~30%** | **Maximum**   | **Damaged/dirty surfaces** ← v1 uses this |

### 2. `box_size=10` — Oversized Pixel Modules (Line 98)

```python
box_size=10   # Each QR module = 10×10 pixels
```

Every tiny square ("module") in the QR grid is rendered as a 10×10 pixel block. For a Version 7 QR code (45×45 modules), this produces a **450×450 px** image *before* adding the border. Combined with `border=4`, the final image balloons to **530×530 px** — far larger than needed for most use cases.

### 3. Long Raw URLs — No Data Optimization

The app encodes the full, raw URL byte-for-byte. A URL like:

```
https://photos.app.goo.gl/abc123XYZlongAlbumIdentifier
```

forces the encoder into **Byte mode** (8 bits per character) and pushes the QR version up. Shorter data → smaller grid → smaller code. The current app has zero mechanisms to shorten or optimize the input data.

---

## v2 Solution — Full Plan

### Goal

Build a **v2 QR Code Generator** that produces compact, poster-friendly QR codes while keeping the app 100% client-side (no backend).

---

### Change 1: Let the User Pick an Error-Correction Level

**What:** Add a dropdown selector for error-correction level (L / M / Q / H) with **M** as the default.

**Why:** Most users print QR codes on clean paper or display them on screens. Level M (15% recovery) is the industry standard default — it's a strong balance between compactness and durability. Giving users the choice lets power users pick H for industrial use or L for maximum compactness.

**Impact:** Switching from H → M alone can **reduce the QR version by 1–3 levels**, shrinking the module grid substantially (e.g., from 45×45 down to 37×37 for the same data).

---

### Change 2: Configurable `box_size` with Smart Defaults

**What:** Add a slider or select control for `box_size` (range: 4–15, default: **6**).

**Why:** A `box_size` of 6 produces crisp, scannable images that are ~60% smaller in pixel dimensions than the current `box_size=10`. The slider lets users scale up for print or down for digital use.

**Impact on Image Size (Version 5 QR, 37×37 modules, border=4):**

| box_size | Image Dimensions | File Size (approx.) |
|----------|-----------------|---------------------|
| 4        | 180×180 px      | ~1 KB               |
| **6**    | **270×270 px**  | **~2 KB**           |
| 10 (v1)  | 450×450 px      | ~5 KB               |

---

### Change 3: Built-in URL Shortening (Client-Side)

**What:** Integrate a **free, no-signup URL shortening API** (e.g., CleanURI or TinyURL API) that the user can optionally trigger with a button/toggle before generating the QR code.

**Why:** This is the **single most impactful change**. A shortened URL like `https://tinyurl.com/y6abc12` is ~30 characters vs. a raw URL that can be 80–150+ characters. Less data = lower QR version = fewer modules = dramatically smaller code.

**Example impact:**

| Input                                          | Characters | QR Version (ECL M) | Grid Size  |
|------------------------------------------------|------------|---------------------|------------|
| `https://photos.app.goo.gl/abc123XYZlongId`   | 50         | Version 4           | 33×33      |
| `https://tinyurl.com/y6abc12`                  | 30         | Version 2           | 25×25      |

**Privacy Note:** The URL is sent to the shortening service's API. We will clearly label this and make it opt-in.

**Fallback:** If the API is unreachable, the app will gracefully fall back to encoding the original URL with a user notification.

---

### Change 4: Reduce Default Border

**What:** Change `border` from `4` to `2`.

**Why:** The QR spec *recommends* a 4-module "quiet zone," but virtually all modern scanners work perfectly with a 2-module border. This trims 4 modules off each side (total grid shrinks by 8 modules in each dimension × box_size pixels).

---

### Change 5: Ditch PyScript — Use a Pure JavaScript QR Library

**What:** Replace PyScript + `qrcode` + `Pillow` with a lightweight JavaScript QR library like `qr-code-styling` or `qrcode-generator`.

**Why:**
- **Cold start elimination:** PyScript downloads an entire WebAssembly Python runtime (~15 MB) on first load. The README itself acknowledges this as a "Technical Constraint." A JS library loads in milliseconds.
- **Instant generation:** No waiting for a Python interpreter to boot.
- **Smaller bundle:** A JS QR library is typically 20–50 KB vs. PyScript's multi-megabyte payload.
- **Better control:** JS libraries like `qr-code-styling` offer direct canvas/SVG rendering with fine-grained control over module shape, colors, gradients, and embedded logos.

**This is not just an optimization — it fundamentally improves the user experience.**

---

### Change 6: SVG Output Option

**What:** Offer QR output as **SVG** in addition to PNG.

**Why:** SVG QR codes are:
- **Resolution-independent** — they scale to any size without pixelation (perfect for posters).
- **Tiny file size** — a typical QR SVG is 2–5 KB regardless of display size.
- **Editable** — designers can open them in Illustrator/Figma and customize colors.

---

### Change 7: Redesigned UI with Real-Time Preview

**What:** Modern, responsive UI with:
- Real-time QR preview as the user types (debounced).
- Side-by-side controls panel and QR preview.
- Light/dark mode toggle.
- Size comparison indicator showing how much smaller the v2 QR is vs. v1 defaults.

**Why:** The current UI works but is basic. A real-time preview eliminates the generate-check-tweak cycle and makes the app feel instant.

---

## Implementation Architecture

```
index.html          ← Single-file app (HTML + CSS + JS)
├── UI Layer        ← HTML structure + CSS styling (vanilla CSS or Tailwind)
├── QR Engine       ← JS QR library (loaded via CDN)
├── URL Shortener   ← Fetch API calls to TinyURL/CleanURI (opt-in)
└── Export Module   ← Canvas → PNG download / SVG download
```

**Tech Stack Comparison (v1 → v2):**

| Component       | v1                     | v2                          |
|----------------|------------------------|-----------------------------|
| QR Engine      | PyScript + qrcode lib  | JavaScript QR library       |
| Rendering      | Pillow (Python)        | Canvas API / SVG            |
| Styling        | Tailwind CDN           | Vanilla CSS (or Tailwind)   |
| Load Time      | ~5-15 seconds          | **< 1 second**              |
| Dependencies   | WebAssembly runtime    | **None** (single CDN link)  |

---

## Summary of Expected Impact

| Metric                    | v1 (Current)       | v2 (Planned)            |
|--------------------------|--------------------|-----------------------|
| QR grid (typical URL)     | ~45×45 modules     | **~25×25 modules**      |
| Image dimensions          | 530×530 px         | **~170×170 px**         |
| File size                 | ~5–8 KB (PNG)      | **~1–2 KB (PNG) / ~3 KB (SVG)** |
| Page load time            | 5–15 seconds       | **< 1 second**          |
| Error correction default  | H (30%)            | **M (15%)**             |
| URL optimization          | None               | **Optional shortening** |
| Output formats            | PNG only           | **PNG + SVG**           |

---

## Is It Possible?

**Yes, absolutely.** Every change listed above is achievable within a single `index.html` file with no backend. The biggest gains come from:

1. **URL shortening** (fewer data modules → smaller grid) — *highest impact*
2. **Dropping error correction from H to M** — *high impact, zero effort*
3. **Switching to a JS QR library** — *eliminates cold start, enables SVG*
4. **Reducing box_size and border** — *immediate pixel-dimension reduction*

The v2 app will produce QR codes that are **roughly 50–70% smaller** than v1 output, making them practical for posters, business cards, stickers, and any space-constrained medium.
