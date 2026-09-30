---
name: python-scraping-specialist
description: Stealth web scraping specialist for high-detection sites. Use for anti-bot bypass, TLS/JA3 fingerprint evasion, residential/mobile proxy rotation, Camoufox/Nodriver stealth browsers, curl_cffi, and Rust (spider-rs) for ultimate performance and undetectability.
---

# Python Stealth Scraping Specialist Skill

## When to Activate
- Any site with Cloudflare, DataDome, Imperva, or fingerprint-based blocking.
- High-volume or previously-blocked targets.
- When standard requests or plain Playwright fail.

## Core Principles (Stealth-First)
- Residential or mobile proxies with per-request or sticky rotation (match geo + language headers).
- TLS/JA3 + HTTP/2 fingerprint spoofing mandatory.
- Engine-level fingerprint patching (not JS patches — they are detected in 2026).
- Human-like behavior: random delays, natural mouse movements, scrolling.
- Never reuse fingerprints across sessions.

## Stealth Workflow (2026 Best Stack)
1. **Browser Automation (Dynamic/JS sites)**  
   - Primary: Camoufox (C++ Firefox fork — best against Cloudflare Turnstile & fingerprinting).  
   - Alternative: Nodriver (undetected CDP, removes webdriver flags).  
   - Fallback: SeleniumBase UC mode.

2. **HTTP-Only (Static sites)**  
   - `curl_cffi` with impersonate (chrome101, chrome120, etc.) for perfect TLS signatures.

3. **Rust for Extreme Stealth/Scale**  
   - spider-rs (built-in stealth mode, proxy rotation, CDP support, 200-1000x faster).  
   - Or custom reqwest + scraper with impersonated TLS client.  
   - Use Rust when Python detection risk is high or you need low-level control.

4. **Proxy Rotation**  
   - Services with massive residential/mobile pools.  
   - Per-request rotation + sticky sessions for logins.  
   - Always rotate User-Agent + Accept-Language together with IP.

5. **Implementation Steps**  
   - Install via poetry/uv.  
   - Launch Camoufox/Nodriver with proxy + stealth config.  
   - Add behavioral randomization.  
   - Store data via SQLAlchemy/Postgres or SQLite.  
   - Run headless only — never expose flags.

## Review Checklist (Before Any Run)
- [ ] TLS fingerprint matches real browser (test with curl_cffi or Camoufox)
- [ ] Proxy geo/language headers consistent
- [ ] No automation markers (navigator.webdriver = false, consistent canvas/WebGL/audio)
- [ ] Success rate >95% on target
- [ ] Fingerprints rotated per session
- [ ] Rate limiting respected per IP

Follow these patterns and the scraper will stay undetected on even the most protected sites in 2026.