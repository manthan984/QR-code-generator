# QR Code Generator v2

A zero-overhead, entirely client-side QR code generator that produces **compact, poster-friendly** QR codes. Built with pure JavaScript — no server, no signup, no Python runtime.

## What's New in v2

This is a complete rewrite of the [original Python/PyScript-based generator](./V2_PLAN.md). Key improvements:

| Feature              | v1                        | v2                             |
|----------------------|---------------------------|--------------------------------|
| QR Engine            | PyScript + Python qrcode  | **JavaScript (instant)**       |
| Page Load            | 5–15 seconds              | **< 1 second**                 |
| Error Correction     | Fixed at H (30%)          | **User-selectable (L/M/Q/H)**  |
| Module Size          | Fixed at 10px             | **Configurable (3–15px)**      |
| Border               | Fixed at 4                | **Configurable (0–6)**         |
| Output Formats       | PNG only                  | **PNG + SVG**                  |
| URL Shortening       | None                      | **Built-in (via is.gd)**       |
| Live Preview         | No                        | **Real-time as you type**      |
| Custom Colors        | No                        | **Foreground + Background**    |
| Size Comparison      | N/A                       | **Shows % smaller than v1**    |

**Result: QR codes are roughly 50–70% smaller** than v1 output, making them practical for posters, business cards, stickers, and any space-constrained medium.

## Architecture

Single-file (`index.html`), entirely client-side. No build steps, no dependencies beyond two CDN links (Inter font + qrcode-generator library).

```
index.html
├── UI Layer        → HTML + vanilla CSS (glassmorphism dark theme)
├── QR Engine       → qrcode-generator (15 KB, via CDN)
├── URL Shortener   → Fetch API → is.gd (opt-in)
└── Export Module   → Canvas → PNG / SVG download
```

## How to Use

1. Open `index.html` in any modern browser (or host it anywhere).
2. Type or paste a URL/text.
3. The QR code generates in real-time.
4. Adjust error correction, module size, border, and colors to taste.
5. Optionally click **Shorten URL** to shrink the QR further.
6. Download as **PNG** (raster) or **SVG** (vector, infinite scalability).

## Deployment

Host on GitHub Pages, Vercel, Netlify, or any static web server. No build steps, no environment variables, no containerization required.

## Tech Stack

- **QR Engine:** [qrcode-generator](https://github.com/nickel-org/qrcode-generator) (v1.4.4)
- **Rendering:** HTML5 Canvas (PNG) + programmatic SVG
- **URL Shortening:** [is.gd](https://is.gd) API (opt-in, client-side fetch)
- **Typography:** [Inter](https://fonts.google.com/specimen/Inter) via Google Fonts
- **Styling:** Vanilla CSS with glassmorphism, gradients, and micro-animations
