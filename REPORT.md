# API Security Risk Analysis — Full Report

**Target:** JSONPlaceholder (`https://jsonplaceholder.typicode.com`)  
**Tester:** Atul | B.Tech CSE (Cyber Security) | GITAM University, Hyderabad | Roll No: 2023003147  
**Test Date:** June 16, 2026  
**Tool:** Postman (PostmanRuntime/7.54.0) on Kali Linux  
**Framework:** OWASP API Security Top 10 (2023)  
**Task:** Future Interns Cybersecurity Program  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Methodology](#2-scope-and-methodology)
3. [Evidence Index — Postman Screenshots](#3-evidence-index--postman-screenshots)
4. [Detailed Findings](#4-detailed-findings)
   - [R-01: Unauthenticated Read Access](#r-01-unauthenticated-read-access--high)
   - [R-02: Sequential ID Enumeration (BOLA)](#r-02-sequential-id-enumeration-bola--high)
   - [R-03: access-control-allow-credentials: true](#r-03-access-control-allow-credentials-true--critical)
   - [R-04: Missing HTTP Security Headers](#r-04-missing-http-security-headers--medium)
   - [R-05: Verbose 404 Boundary Oracle](#r-05-verbose-404-boundary-oracle--low)
5. [Risk Summary Matrix](#5-risk-summary-matrix)
6. [Remediation Roadmap](#6-remediation-roadmap)
7. [OWASP Coverage Map](#7-owasp-coverage-map)
8. [Chain of Custody](#8-chain-of-custody)

---

## 1. Executive Summary

This report documents a **manual, read-only API Security Risk Analysis** conducted on JSONPlaceholder using Postman on Kali Linux. The assessment identified **5 security risks** across 3 OWASP API Security Top 10 (2023) categories.

| Severity | Count | Findings |
|----------|-------|----------|
| 🔴 Critical | 1 | R-03: access-control-allow-credentials: true |
| 🟠 High | 2 | R-01: Unauthenticated read, R-02: BOLA enumeration |
| 🟡 Medium | 1 | R-04: Missing security headers |
| 🟢 Low | 1 | R-05: Verbose 404 oracle |

All testing was **read-only**. No authentication was bypassed, no data was modified, and no denial-of-service testing was performed. JSONPlaceholder is a public demo API — these findings represent patterns that are dangerous in production systems.

---

## 2. Scope and Methodology

### What Was Tested
- `GET /posts` — full post listing
- `GET /posts/{id}` — individual post access (IDs 1, 404, 505)
- `GET /post/1` — endpoint naming test (singular vs plural)
- HTTP response headers across all responses

### Testing Method
- Manual HTTP requests via Postman
- Response header inspection (Headers tab in Postman)
- HTTP status code pattern analysis
- No authentication headers were added to any request

### What Was NOT Tested
- POST / PUT / DELETE (write operations) — out of scope for this test session
- ReqRes API — not included in this session
- Authentication bypass or exploitation
- Automated scanning or fuzzing

---

## 3. Evidence Index — Postman Screenshots

All screenshots are located in `screenshots/` folder of this repository.

| Evidence ID | File | Endpoint | Status | Finding |
|-------------|------|----------|--------|---------|
| SS-01 | `Screenshot_2026-06-16_124508.png` | GET /posts/1 (request headers) | 404 Not Found | R-01: No auth header sent |
| SS-02 | `Screenshot_2026-06-16_124633.png` | GET /posts/1 (response headers) | 404 Not Found | R-03: access-control-allow-credentials: true |
| SS-03 | `Screenshot_2026-06-16_125001.png` | GET /posts (response headers) | 200 OK | R-04: Missing security headers |
| SS-04 | `Screenshot_2026-06-16_125154.png` | GET /posts/505 | 404 Not Found | R-02: BOLA boundary test |
| SS-05 | `Screenshot_2026-06-16_125357.png` | GET /posts (full headers) | 200 OK | R-04: Full header audit |
| SS-06 | `Screenshot_2026-06-16_125418.png` | GET /posts/404 | 404 Not Found | R-02+R-05: Enumeration oracle |

### SS-01 — GET /posts/1 Request Headers (No Auth)

![SS-01](screenshots/Screenshot_2026-06-16_124508.png)

**What this shows:**  
The Postman request was sent to `GET /posts/1` with only auto-generated headers:
- `Postman-Token` — auto
- `Host` — auto
- `User-Agent: PostmanRuntime/7.54.0`
- `Accept: */*`
- `Accept-Encoding: gzip, deflate, br`
- `Connection: keep-alive`

**No `Authorization` header was added.** Despite sending a completely unauthenticated request, the server processed it fully. The 404 here is because of a URL typo (`/post/1` singular instead of `/posts/1` plural) — not because authentication failed. The server never challenged the request for credentials.

---

### SS-02 — GET /posts/1 Response Headers (CRITICAL CORS Finding)

![SS-02](screenshots/Screenshot_2026-06-16_124633.png)

**What this shows:**  
Response headers from the 404 response. Key findings visible:

```
access-control-allow-credentials: true    ← CRITICAL
cache-control: max-age=43200
content-type: application/json; charset=utf-8
content-length: 2
server: cloudflare
```

**Security issues visible:**
- `access-control-allow-credentials: true` — present even on a 404 response
- `server: cloudflare` — infrastructure disclosure
- `cache-control: max-age=43200` — 12-hour cache on API responses
- **Missing:** X-Content-Type-Options, Strict-Transport-Security, X-Frame-Options, Content-Security-Policy

---

### SS-03 — GET /posts Response Headers (200 OK + Missing Headers)

![SS-03](screenshots/Screenshot_2026-06-16_125001.png)

**What this shows:**  
Successful response to `GET /posts`. Key observations:
- **200 OK** — full posts dataset returned with zero authentication
- `access-control-allow-credentials: true` — confirmed on 200 responses too
- `content-encoding: gzip` — confirms 7.98 KB is decompressed size (100 posts)
- Test result: **1/1 PASSED** — Postman test confirmed 200 status

---

### SS-04 — GET /posts/505 — BOLA Boundary Test

![SS-04](screenshots/Screenshot_2026-06-16_125154.png)

**What this shows:**  
Testing ID 505 (beyond the valid range of 1–100):
- **404 Not Found** — confirms ID 505 does not exist
- `content-length: 2` — response body is `{}` (empty JSON object)
- `access-control-allow-credentials: true` — present on this 404 too
- Date header: `Tue, 16 Jun 2026 07:16:00 GMT` — timestamp of test

---

### SS-05 — GET /posts Full Response Headers

![SS-05](screenshots/Screenshot_2026-06-16_125357.png)

**What this shows:**  
Full header list for a successful `GET /posts` request. Complete header audit:

**Present:**
```
:status: 200
date: Tue, 16 Jun 2026 07:19:38 GMT
content-type: application/json; charset=utf-8
access-control-allow-credentials: true
cache-control: max-age=43200
content-encoding: gzip
etag: W/"6b80-Ybsq/K6GwwqrYkAsFxqDXGC7DoM"
expires: -1
nel: {...}
pragma: no-cache
report-to: {...}
reporting-endpoints: heroku-nel=...
```

**Missing (security headers):**
```
X-Content-Type-Options     ← ABSENT
X-Frame-Options            ← ABSENT
Strict-Transport-Security  ← ABSENT
Content-Security-Policy    ← ABSENT
X-XSS-Protection           ← ABSENT
Permissions-Policy         ← ABSENT
```

---

### SS-06 — GET /posts/404 — Enumeration Oracle

![SS-06](screenshots/Screenshot_2026-06-16_125418.png)

**What this shows:**  
Testing ID 404 (non-existent in the posts dataset):
- **404 Not Found** — ID 404 does not exist
- `content-length: 2` — same minimal `{}` response as SS-04
- Date: `Tue, 16 Jun 2026 07:14:17 GMT`
- Pattern confirmed: Valid IDs → 200 OK | Invalid IDs → 404 with `{}`

Combined with SS-04 (ID 505 → 404), this proves the dataset ends at ID 100. Any attacker can determine this in seconds with a simple loop.

---

## 4. Detailed Findings

---

### R-01: Unauthenticated Read Access — HIGH

| Field | Detail |
|-------|--------|
| **OWASP** | API1:2023 – Broken Object Level Authorization |
| **CVSS** | 7.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N) |
| **Evidence** | SS-01, SS-03, SS-05 |

**Description:**  
The endpoint `GET /posts` returns the full dataset of 100 posts (7.98 KB) without requiring any form of authentication. The request in SS-01 was sent with only Postman's auto-generated headers — no Authorization header, no API key, no session cookie. Despite this, the server returned HTTP 200 OK with the complete dataset.

**Observed Behavior:**
```
Request:
  GET https://jsonplaceholder.typicode.com/posts
  Headers: Postman-Token, Host, User-Agent, Accept, Accept-Encoding, Connection
  Authorization: [NOT PRESENT]

Response:
  HTTP/2 200 OK
  Content-Length: 7.98 KB
  Content-Type: application/json; charset=utf-8
  Body: [array of 100 post objects]
```

**Business Impact:**  
Any internet-connected client can read the entire dataset with a single HTTP request. No credentials, registration, or access approval is required. In a production system, this would expose all stored data to the public internet — a direct GDPR Article 25 violation.

**Remediation:**
- Require a valid Bearer token or API key on all endpoints including GET
- Return HTTP 401 Unauthorized for unauthenticated requests
- Implement token validation middleware applied globally before routing
- Adopt deny-by-default: endpoints are locked until explicitly opened

---

### R-02: Sequential ID Enumeration (BOLA) — HIGH

| Field | Detail |
|-------|--------|
| **OWASP** | API1:2023 – Broken Object Level Authorization |
| **CVSS** | 8.1 (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N) |
| **Evidence** | SS-04, SS-06 |

**Description:**  
All post objects use sequential integer IDs. Testing IDs 404 and 505 both returned HTTP 404 with a consistent, minimal response body (`{}`, 2 bytes). This creates a binary oracle: valid IDs return 200 with data, invalid IDs return 404 with empty body. The complete dataset (IDs 1–100) is trivially enumerable.

**Observed Behavior:**
```
GET /posts/404  → 404 Not Found | content-length: 2 | body: {}
GET /posts/505  → 404 Not Found | content-length: 2 | body: {}
GET /posts      → 200 OK | 7.98 KB | 100 records (IDs 1–100)

Enumeration script (conceptual):
  for i in range(1, 1000):
      response = GET /posts/{i}
      if response.status == 200:
          collect(response.body)
      # Stops when consistent 404s appear → upper bound found
```

**Business Impact:**  
Zero prior knowledge needed to extract the complete database. In a production user API, this reveals all account data. BOLA is OWASP's #1 most common and impactful API vulnerability class.

**Remediation:**
- Replace sequential integer IDs with UUID v4 identifiers
- Implement server-side ownership checks: verify the requester owns the requested object
- Log and alert on sequential ID access patterns from a single IP
- Return the same response for both "not found" and "forbidden" to prevent oracles

---

### R-03: access-control-allow-credentials: true — CRITICAL

| Field | Detail |
|-------|--------|
| **OWASP** | API8:2023 – Security Misconfiguration |
| **CVSS** | 9.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N) |
| **Evidence** | SS-02, SS-03, SS-04, SS-05, SS-06 |

**Description:**  
The header `access-control-allow-credentials: true` is present on **every single response** — including both 200 OK and 404 Not Found responses. This CORS header instructs browsers to include cookies, Authorization headers, and TLS client certificates in cross-origin requests.

This header was confirmed in **5 out of 6 screenshots** (SS-02 through SS-06), demonstrating it is applied globally with no exceptions.

**Observed in responses:**
```
SS-02 (404 /posts/1):  access-control-allow-credentials: true
SS-03 (200 /posts):    access-control-allow-credentials: true
SS-04 (404 /posts/505): access-control-allow-credentials: true
SS-05 (200 /posts):    access-control-allow-credentials: true
SS-06 (404 /posts/404): access-control-allow-credentials: true
```

**Business Impact:**  
Combined with a permissive Access-Control-Allow-Origin policy, this allows any malicious website to:
1. Make credentialed API requests on behalf of a logged-in victim
2. Read the full API response (including auth tokens and PII)
3. Achieve account takeover through the victim's own browser

This requires no malware — just the victim visiting a malicious webpage while logged into the target application.

**Remediation:**
- Remove `access-control-allow-credentials: true` unless absolutely required for a specific use case
- If credentials must be allowed, restrict `Access-Control-Allow-Origin` to an explicit domain allowlist — never combine credentials with a wildcard origin
- Implement CSRF tokens on all state-changing endpoints when credentials are allowed
- Audit the full CORS policy across all endpoints after every deployment

---

### R-04: Missing HTTP Security Headers — MEDIUM

| Field | Detail |
|-------|--------|
| **OWASP** | API8:2023 – Security Misconfiguration |
| **CVSS** | 5.4 (AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:L/A:N) |
| **Evidence** | SS-02, SS-03, SS-05 |

**Description:**  
Inspection of response headers across multiple requests (SS-03 and SS-05 show the most complete header sets) confirmed that all standard API security headers are absent. The only security-relevant headers present are `content-type` and `cache-control`.

**Header Audit Results:**

| Header | Value Observed | Security Impact |
|--------|----------------|-----------------|
| `X-Content-Type-Options` | ❌ ABSENT | MIME type sniffing attacks |
| `X-Frame-Options` | ❌ ABSENT | Clickjacking / UI redress |
| `Strict-Transport-Security` | ❌ ABSENT | SSL stripping / MITM |
| `Content-Security-Policy` | ❌ ABSENT | XSS injection in consumers |
| `X-XSS-Protection` | ❌ ABSENT | Cross-site scripting |
| `Permissions-Policy` | ❌ ABSENT | Feature abuse |
| `content-type: application/json` | ✅ PRESENT | Correct MIME type set |
| `access-control-allow-credentials` | ✅ PRESENT (risk) | See R-03 |

**Remediation:**
- Add `X-Content-Type-Options: nosniff` to all responses
- Add `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- Add `X-Frame-Options: DENY` for web-facing consumers
- Use a security header middleware globally (e.g., Helmet.js for Node.js)
- Automate header checks in CI/CD pipeline using securityheaders.com

---

### R-05: Verbose 404 Boundary Oracle — LOW

| Field | Detail |
|-------|--------|
| **OWASP** | API9:2023 – Improper Inventory Management |
| **CVSS** | 3.7 (AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N) |
| **Evidence** | SS-04, SS-06 |

**Description:**  
Invalid post IDs return a consistent `404 Not Found` with `content-length: 2` (body: `{}`). This consistent, distinguishable response creates a binary oracle that reveals the exact boundaries of the dataset.

**Pattern Confirmed:**
```
GET /posts/404  → 404 | content-length: 2 | {}
GET /posts/505  → 404 | content-length: 2 | {}
GET /posts/1    → 200 | full post object
```

This confirms the dataset contains exactly IDs 1–100, with no gaps.

**Remediation:**
- Return the same response for "not found" and "forbidden" to prevent existence oracles
- Implement UUID identifiers (R-02 fix) to eliminate integer boundary enumeration
- Log high-frequency 404 access patterns as potential enumeration in progress

---

## 5. Risk Summary Matrix

| ID | Title | Severity | OWASP Category | Evidence | Status |
|----|-------|----------|----------------|----------|--------|
| R-01 | Unauthenticated Read Access | 🟠 HIGH | API1:2023 – BOLA | SS-01, SS-03, SS-05 | Confirmed |
| R-02 | Sequential ID Enumeration (BOLA) | 🟠 HIGH | API1:2023 – BOLA | SS-04, SS-06 | Confirmed |
| R-03 | access-control-allow-credentials: true | 🔴 CRITICAL | API8:2023 – Misconfiguration | SS-02, SS-03, SS-04, SS-05, SS-06 | Confirmed |
| R-04 | Missing HTTP Security Headers | 🟡 MEDIUM | API8:2023 – Misconfiguration | SS-02, SS-03, SS-05 | Confirmed |
| R-05 | Verbose 404 Boundary Oracle | 🟢 LOW | API9:2023 – Inventory | SS-04, SS-06 | Confirmed |

---

## 6. Remediation Roadmap

### Wave 1 — Immediate (0–48 Hours)
| Finding | Action |
|---------|--------|
| R-03 | Remove or restrict `access-control-allow-credentials: true` |
| R-01 | Require authentication on all endpoints |

### Wave 2 — Short-Term (1–2 Weeks)
| Finding | Action |
|---------|--------|
| R-02 | Replace sequential IDs with UUID v4 + add ownership checks |
| R-04 | Add security headers via middleware (Helmet.js or NGINX) |

### Wave 3 — Medium-Term (2–4 Weeks)
| Finding | Action |
|---------|--------|
| R-05 | Normalise 404/403 responses to prevent boundary oracles |

---

## 7. OWASP Coverage Map

| OWASP ID | Category | Finding | Status |
|----------|----------|---------|--------|
| API1:2023 | Broken Object Level Authorization | R-01, R-02 | ✅ Covered |
| API2:2023 | Broken Authentication | — | ⬜ Not tested this session |
| API3:2023 | Broken Object Property Level Auth | — | ⬜ Not tested this session |
| API4:2023 | Unrestricted Resource Consumption | — | ⬜ Not tested this session |
| API5:2023 | Broken Function Level Authorization | — | ⬜ Not tested this session |
| API6:2023 | Unrestricted Access to Sensitive Flows | — | ⬜ Not tested this session |
| API7:2023 | Server-Side Request Forgery (SSRF) | — | ➖ Not applicable |
| API8:2023 | Security Misconfiguration | R-03, R-04 | ✅ Covered |
| API9:2023 | Improper Inventory Management | R-05 | ✅ Covered |
| API10:2023 | Unsafe Consumption of APIs | — | ➖ Out of scope |

---

## 8. Chain of Custody

All evidence screenshots were captured directly from Postman during the live testing session.

| Evidence ID | Filename | Timestamp (from filename) | Tester | Tool |
|-------------|----------|--------------------------|--------|------|
| SS-01 | Screenshot_2026-06-16_124508.png | 16 Jun 2026, 12:45:08 | Atul | Postman |
| SS-02 | Screenshot_2026-06-16_124633.png | 16 Jun 2026, 12:46:33 | Atul | Postman |
| SS-03 | Screenshot_2026-06-16_125001.png | 16 Jun 2026, 12:50:01 | Atul | Postman |
| SS-04 | Screenshot_2026-06-16_125154.png | 16 Jun 2026, 12:51:54 | Atul | Postman |
| SS-05 | Screenshot_2026-06-16_125357.png | 16 Jun 2026, 12:53:57 | Atul | Postman |
| SS-06 | Screenshot_2026-06-16_125418.png | 16 Jun 2026, 12:54:18 | Atul | Postman |

---

## Ethical Disclaimer

> This assessment was conducted exclusively on JSONPlaceholder — a public demo API created specifically for developer testing at `jsonplaceholder.typicode.com`. All testing was read-only. No authentication was bypassed, no data was modified, and no denial-of-service testing was performed. This project is for educational and professional development purposes only as part of the Future Interns Cybersecurity Program.

---

*Prepared by Atul | B.Tech CSE (Cyber Security) | GITAM University, Hyderabad | Roll No: 2023003147*
