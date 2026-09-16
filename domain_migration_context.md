# Domain Migration Context

## Current State
- **Current Domain:** `manthank.me`
- **Registrar:** Namecheap (acquired via GitHub Student Developer Pack)
- **Pain Points:** Domain is too long, and the user dislikes the Namecheap environment.
- **Expiration Date:** March 19, 2027
- **Billing Status:** Auto-renew has been successfully turned OFF. The domain will expire naturally without charging the user.
- **Existing Architecture:** 5 to 6 active projects are currently routed to subdomains (e.g., `leetcodeguru.manthank.me`).

## User Goal
The user wants to purchase a permanent, professional "developer domain" for their portfolio, personal work, and real engineering projects. They want to abandon Namecheap and `manthank.me` entirely but need a smooth transition so existing project links don't break immediately.

## Proposed Strategy & Recommendations

### 1. Choosing a Professional TLD
- **Recommended TLD:** `.dev` is the gold standard. It is backed by Google, enforces HTTPS by default, and immediately signals an engineering background.
- **Alternatives:** `.io` (more expensive) or `.co`.
- **Naming:** Keep it short. Examples: `manthan.dev`, `mthn.dev`, or `mk.dev`.

### 2. Choosing a New Registrar
- **Primary Recommendation:** Cloudflare. They sell domains at wholesale cost (no markup or renewal hikes) and provide an incredibly powerful developer environment (free DNS, Cloudflare Workers, KV databases).
- **Secondary Recommendation:** Porkbun (transparent pricing, developer-friendly).

### 3. The Migration Path (Bridging the Domains)
To ensure the 5-6 existing projects (like `leetcodeguru.manthank.me`) do not break immediately while transitioning to the new domain, the user should:
1. Purchase the new domain (e.g., `mthn.dev`) on Cloudflare.
2. Change the Nameservers for `manthank.me` in Namecheap to point to Cloudflare. This allows the user to manage both domains under one dashboard in Cloudflare.
3. Set up a free Cloudflare Page Rule to automatically redirect traffic from the old domain to the new domain.
   - Example rule: `*.manthank.me/*` -> `301 Redirect` -> `$1.mthn.dev/$2`
   - This ensures that anyone visiting `leetcodeguru.manthank.me` is seamlessly sent to `leetcodeguru.mthn.dev`.
4. Over the next year (before March 2027), update resumes, GitHub repositories, and LinkedIn profiles to use the new domain.
5. On March 19, 2027, let `manthank.me` expire safely. The transition will be complete.
