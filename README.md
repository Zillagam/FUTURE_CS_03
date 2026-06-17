# 🔐 API Security Risk Analysis — JSONPlaceholder

> **Future Interns Cybersecurity Task | Read-Only API Security Assessment**  
> Conducted by: **Atul** | B.Tech CSE (Cyber Security) | GITAM University, Hyderabad  
> Date: June 16, 2026 | Methodology: OWASP API Security Top 10 (2023)

---

## 📌 Project Overview

This project is a **professional read-only API Security Risk Analysis** conducted on the public demo API [JSONPlaceholder](https://jsonplaceholder.typicode.com) using **Postman**.

The goal was to identify common API security vulnerabilities as a security consultant would — without exploiting, modifying, or attacking any real system.

---

## 🎯 Target API

| Property | Details |
|----------|---------|
| **API Name** | JSONPlaceholder |
| **Base URL** | `https://jsonplaceholder.typicode.com` |
| **Type** | Public REST API (Demo/Test) |
| **Purpose** | Free fake API for testing and prototyping |
| **Testing Tool** | Postman |
| **Framework** | OWASP API Security |

---

## 🧪 Tests Performed

All tests were **read-only, unauthenticated GET requests** using Postman. No data was exploited, modified, or destroyed.

| # | Endpoint Tested | Method | Result | Risk Found |
|---|----------------|--------|--------|------------|
| 1 | `/posts` | GET | ✅ 200 OK | Unauthenticated read access |
| 2 | `/posts/1` | GET | ✅ 200 OK | Sequential ID enumeration (BOLA) |
| 3 | `/posts/404` | GET | ❌ 404 Not Found | ID boundary confirmed by response |
| 4 | `/posts/505` | GET | ❌ 404 Not Found | Upper bound enumerable |
| 5 | Response Headers | — | Inspected | Missing security headers |
| 6 | `access-control-allow-credentials` | — | `true` | Dangerous CORS misconfiguration |

---

## 🔍 Key Findings from Postman Screenshots

### Finding 1 — Unauthenticated Read Access (HIGH)
- **Endpoint:** `GET /posts`
- **Status:** `200 OK` — 7.98 KB of data returned
- **No Authorization header was sent**
- **Evidence:** Screenshot shows 200 OK with 24 response headers and full data returned

### Finding 2 — Sequential Integer ID Enumeration / BOLA (HIGH)
- **Endpoint:** `GET /posts/1`, `/posts/404`, `/posts/505`
- **Observation:** Valid IDs return 200, invalid IDs return 404
- **Risk:** An attacker can iterate integers to enumerate all valid resources
- **Evidence:** Screenshots show predictable 404 vs 200 pattern

### Finding 3 — Dangerous CORS Misconfiguration (CRITICAL)
- **Header observed:** `access-control-allow-credentials: true`
- **Risk:** When `allow-credentials: true` is set, combining it with a permissive CORS origin allows any malicious website to make authenticated cross-origin requests and read responses
- **This is MORE dangerous than a standard wildcard CORS issue**
- **Evidence:** Header visible in both 200 and 404 response screenshots

### Finding 4 — Missing Security Headers (MEDIUM)
From response headers inspection, the following **critical security headers are absent**:

| Header | Status | Risk |
|--------|--------|------|
| `X-Content-Type-Options` | ❌ MISSING | MIME sniffing attacks |
| `X-Frame-Options` | ❌ MISSING | Clickjacking |
| `Strict-Transport-Security` | ❌ MISSING | SSL stripping |
| `Content-Security-Policy` | ❌ MISSING | XSS injection |
| `X-XSS-Protection` | ❌ MISSING | Cross-site scripting |

### Finding 5 — URL Typo Exposes Endpoint Naming Pattern (INFO)
- Tested `GET /post/1` (singular) → `404 Not Found`
- Correct endpoint is `GET /posts/1` (plural)
- The API returns different content-length (2 bytes = `{}`) confirming the endpoint structure
- **Risk:** Error responses confirm resource naming patterns to attackers

---

## 📊 Risk Summary

| Risk ID | Title | Severity | OWASP Category |
|---------|-------|----------|----------------|
| R-01 | Unauthenticated Read Access | 🟠 HIGH | API1:2023 – BOLA |
| R-02 | Sequential Integer ID Enumeration | 🟠 HIGH | API1:2023 – BOLA |
| R-03 | `access-control-allow-credentials: true` CORS | 🔴 CRITICAL | API8:2023 – Misconfiguration |
| R-04 | Missing Security Headers | 🟡 MEDIUM | API8:2023 – Misconfiguration |
| R-05 | Verbose 404 Responses (Boundary Oracle) | 🟢 LOW | API9:2023 – Inventory |

---

## 🛠️ Tools Used

```
API Client:  Postman (PostmanRuntime/7.54.0)
Target API:  JSONPlaceholder (jsonplaceholder.typicode.com)
Framework:   OWASP API Security Top 10 (2023)
```

---

## 📁 Repository Structure

```
api-security-audit/
│
├── README.md                          ← This file
├── report/
│   └── API_Security_Risk_Analysis.pdf ← Full professional report
├── screenshots/
│   ├── 01_get_posts_200_ok.png        ← GET /posts → 200 OK
│   ├── 02_get_post1_404.png           ← GET /post/1 → 404 (typo test)
│   ├── 03_posts_headers_200.png       ← Response headers on 200
│   ├── 04_posts_404_headers.png       ← Response headers on 404
│   ├── 05_posts505_404.png            ← Boundary enumeration test
│   └── 06_posts404_404.png            ← ID 404 not found
└── postman/
    └── api-security-audit.json        ← Exported Postman collection
```

---

## ⚠️ Ethical Disclaimer

> This assessment was conducted **exclusively on a public demo API** (JSONPlaceholder) designed for testing purposes.  
> All testing was **read-only**. No authentication was bypassed, no data was modified, and no denial-of-service testing was performed.  
> This project is for **educational and professional development purposes only**.

---

## 👤 Author

**Atul**  
B.Tech CSE (Cyber Security) — GITAM University, Hyderabad  
Future Interns Cybersecurity Program

---

## 📚 References

- [OWASP API Security Top 10 (2023)](https://owasp.org/API-Security/editions/2023/en/0x00-header/)
- [JSONPlaceholder Documentation](https://jsonplaceholder.typicode.com/guide/)
- [Postman Documentation](https://learning.postman.com/docs/getting-started/introduction/)
- [MDN CORS Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
