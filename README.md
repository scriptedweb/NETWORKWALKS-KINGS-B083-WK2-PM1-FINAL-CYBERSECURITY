## 🔍 Footprinting, OSINT & Network Scanning — Cybersecurity Internship (Week 2)

**Author:** Kings Ojore Ojorumi
**Program:** Cybersecurity Internship, Networkwalks (B083-Networkwalks)
**Date:** 17 September 2026
**Modules:** W2-PM1 → W2-PM5

---

## ⚠️ Disclaimer

All activity documented here was performed **with explicit authorized permission** against `networkwalks.com`, or against infrastructure I personally own (my own local LAN / VMs). This write-up is for educational and portfolio purposes only. Do not run these techniques against systems you don't have permission to test — unauthorized access is illegal in most jurisdictions.

---

## 📋 Overview

This project covers five hands-on reconnaissance modules completed during Week 2 of my internship:

| # | Module | Tool(s) |
|---|--------|---------|
| 1 | Footprinting with multiple tools | `whois`, `whatweb`, `nslookup`, `curl`, `wafw00f` |
| 2 | Footprinting with GHDB | Google Hacking Database |
| 3 | Footprinting with Maltego | Maltego (OSINT graphing) |
| 4 | Footprinting with theHarvester | theHarvester |
| 5 | Network scanning | Zenmap (Nmap GUI) |

**Target(s):**
- `networkwalks.com` — authorized
- Local LAN `10.0.0.10/24` — own network

---

## 🛠️ Tools Used

- Kali Linux & Windows 10 (VirtualBox)
- WHOIS, WhatWeb, nslookup, curl, wafw00f, dnsrecon
- Google Hacking Database (GHDB)
- Maltego
- theHarvester
- Zenmap (Nmap)

---

## 1️⃣ Footprinting — `networkwalks.com`

### WHOIS
```bash
whois networkwalks.com
```
- Registrar: **GoDaddy.com, LLC**
- Created: `2019-11-06` · Expires: `2027-11-06` · Updated: `2025-11-12`
- Name servers: `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM`
- Registrant identity protected via **Domains By Proxy, LLC** (privacy proxy)
- DNSSEC: **unsigned**

### WhatWeb
```bash
whatweb networkwalks.com
```
- HTTP → 301 redirect to HTTPS
- HTTPS → `200 OK`, **Apache**, IP `192.232.216.135`
- **WordPress 7.1**, **WP Download Manager 3.3.58**, Bootstrap 7.1, JQuery 3.7.1
- Google Tag Manager present
- Contact email exposed: `info@networkwalks.com`
- Page title: *"Networkwalks Academy"*

### nslookup
```bash
nslookup networkwalks.com
```
- Resolves to `192.232.216.135` (via `8.8.8.8`) — consistent with WhatWeb.

### curl -i
```bash
curl -i networkwalks.com
```
- `301 Moved Permanently` → `https://networkwalks.com/`
- `X-Redirect-By: WordPress – Really Simple Security` ← **reveals the specific security plugin**
- `Set-Cookie: __wpdm_client=...` (tied to WP Download Manager)
- `Permissions-Policy` references Google reCAPTCHA, Cloudflare Turnstile, hCaptcha

### wafw00f
```bash
wafw00f networkwalks.com
```
- Site is protected by **ModSecurity (SpiderLabs) WAF**

### dnsrecon
```bash
dnsrecon -d networkwalks.com
```
- SOA/NS on `ns6135`/`ns6136.hostgator.com`
- **BIND version `9.16.23-RH` exposed** on both name servers
- MX: `mail.networkwalks.com` → `192.232.216.135`
- SPF: `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`
- 8× SRV records for `_autodiscover._tcp` → `cpanelemaildiscovery.cpanel.net` (cPanel mail hosting confirmed)

---

## 2️⃣ Search Engine Hacking (GHDB)

Used Google Hacking Database dorks to search for exposed unsecured webcams and publicly indexed mathematics eBooks.

> 🚧 **TODO:** add the exact dork strings used and the results/screenshots for each search here.

---

## 3️⃣ OSINT Graphing with Maltego

Ran the **"To Email Addresses [Search Engine]"** transform against the `networkwalks.com` Domain entity in Maltego (Desktop 4.13.0).

- Result: `info@networkwalks.com` — corroborating the address found via WhatWeb, via a completely independent OSINT method.

---

## 4️⃣ theHarvester (Practice Task — `microsoft.com`)

This was a deliberate syntax/tooling practice run on `microsoft.com`, separate from the `networkwalks.com` target.

```bash
theHarvester -d microsoft.com -l 1000 -b baidu   # completed, 0 results
theHarvester -d microsoft.com -l 50 -b all       # most sources skipped — missing API keys
```

**Takeaway:** theHarvester's usefulness depends heavily on configured API keys for its data sources (bevigil, BuiltWith, Censys, etc.). Confirmed correct command syntax and tool behavior when sources are unavailable.

---

## 5️⃣ Network Scanning — Zenmap (Local LAN)

**ipconfig** (Windows 10 VM):
```
IPv4 Address: 10.0.0.10
Subnet Mask:  255.255.255.0
Gateway:      10.0.0.1
```

**Zenmap Ping Scan:**
```bash
nmap -sn 10.0.0.10/24
```
- 256 IPs scanned in 3.53s → **4 live hosts**:

| Host | MAC Address | Notes |
|------|-------------|-------|
| 10.0.0.1 | 52:55:0A:00:00:01 | Default gateway |
| 10.0.0.2 | 08:00:27:8A:35:D2 | Oracle VirtualBox virtual NIC |
| 10.0.0.3 | 52:55:0A:00:00:03 | Unknown vendor |
| 10.0.0.10 | — | Scanning host (self) |

Topology view confirmed a star topology with all hosts connecting through the local host (no traceroute info — dashed lines).

---

## 🚨 Risk Summary

| Finding | Risk |
|---|:---:|
| WordPress/plugin versions exposed (WhatWeb, curl) | 🟠 Medium |
| Server IP identifiable | 🟢 Low |
| HTTP headers leak plugin/security-tool names | 🟠 Medium |
| WAF vendor identifiable (ModSecurity) | 🟢 Low |
| DNS/BIND version & mail infrastructure exposed | 🟠 Medium |
| Contact email discoverable via OSINT | 🟢 Low |
| 4 live hosts visible on local LAN | 🟠 Medium |

> These are **observations from reconnaissance**, not confirmed vulnerabilities. No exploitation or credential testing was performed.

---

## ✅ Recommendations

- Suppress version-revealing headers (e.g. `X-Redirect-By`)
- Keep WordPress core/plugins patched and current
- Restrict BIND version disclosure on name servers
- Consider enabling DNSSEC
- Keep WAF rules tuned and monitored
- Replace plaintext contact email with a contact form
- Periodically re-scan the LAN and verify all MAC vendors
- Configure API keys for OSINT tooling for complete coverage
- Only test systems with explicit authorization

---

## 🧾 Conclusion

This week reinforced that a lot can be learned about a target *before* any exploitation — through DNS records, HTTP headers, WHOIS data, and OSINT correlation tools like Maltego. It also highlighted the operational reality that OSINT tools are only as good as their configuration (API keys), and that good reporting means being transparent about what worked, what didn't, and why.

All activity was performed within an authorized scope: either against `networkwalks.com` with explicit permission, or against infrastructure I own.

---

📁 *Full evidence screenshots available in `/screenshots` — see the companion PDF/Word report for the complete write-up with all figures.*
