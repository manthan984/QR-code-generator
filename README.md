A zero-overhead, entirely client-side QR code generator utilizing PyScript to execute Python natively in the browser. 

## Architecture
This project eliminates backend dependencies by porting standard Python libraries (`qrcode`, `pillow`) directly to the DOM. It guarantees permanent, static QR code generation without relying on subscription-based API intermediaries.

## Core Mechanics
* **100% Client-Side Execution:** No server processing. All matrix compilation happens in the user's browser.
* **Static Permanence:** Embeds the exact URL into the pixel payload. No dynamic redirects, zero risk of link decay from third-party QR providers.
* **Dynamic Output Sanitization:** Automatically parses the input URL to construct a secure, filesystem-compliant `.png` filename for the download payload.
* **High Error Correction:** Engineered with `ERROR_CORRECT_H` (30% damage recovery) ensuring physical scannability under harsh conditions.

## Tech Stack
* **Engine:** PyScript (Python 3.x in the browser)
* **Python Libraries:** `qrcode`, `Pillow`
* **Frontend:** HTML5, Tailwind CSS

## Deployment Strategy
This system is designed for instant edge deployment.
1. Clone this repository.
2. Host the root directory on **GitHub Pages**, **Vercel**, or any static web server.
3. No build steps. No environment variables. No containerization required.

## Technical Constraints
* **Cold Start:** The initial load requires fetching the PyScript WebAssembly engine. Subsequent loads are cached and instantaneous.
* **Protocol Targeting:** Validates against HTTP/HTTPS structures to prevent malformed code execution.

---
*Built by [Manthan](https://manthank.me) - Engineered for zero maintenance and maximum independence.*
"""

with open("README.md", "w") as f:
    f.write(readme_content)

print("File generated successfully.")
