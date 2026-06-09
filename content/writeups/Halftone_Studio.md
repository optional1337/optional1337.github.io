---
title: "WebVerse: Halftone Studio"
date: 2026-06-08
weight: 4
tags: ["webverse","JWT"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "JWT Algorithm Confusion (RS256 → HS256) attack Challenge Lab Writeup."
summary: "JWT Algorithm Confusion (RS256 → HS256) attack Challenge Lab Writeup."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/halftone_studio_cover.svg"
    alt: "Halftone Studio challenge"
    caption: "Halftone Studio challenge (JWT)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **Halftone Studio**

**Category:** Web Security / JWT

**Vulnerability:** JWT **Algorithm confusion attacks** (privilege escalation)

**Platform:** webverse
{{< /callout>}}

## Lab Description

{{< callout insight "Briefing" >}}
*Halftone Studio, founded 2021, routes orders from about eight hundred indie merch storefronts to a shifting roster of printers across three continents. The platform migrated off raw API keys to a new in-house auth layer last quarter — a contractor shipped it on a tight cutover window, and the senior engineer who would have read the verifier code closely had already given notice. With a Series A data room opening next month the compliance team wants a third-party look first.*
{{< /callout>}}

---

## Create an Account & Log In

We start by registering a new account on the platform and logging in with those credentials. Nothing unusual here standard signup flow.

{{< figure align="center" src="/images/Halftone_Studio_1.png" caption="Creating an Account" >}}


{{< figure align="center" src="/images/Halftone_Studio_2.png" caption="Logged In" >}}


---

## Inspect the JWT

After logging in, we find a JWT (JSON Web Token) stored as a cookie. Decoding it reveals the following structure:

{{< figure align="center" src="/images/Halftone_Studio_3.png" caption="Inspecting JWT" >}}


**Header:**

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "main"
}
```

**Payload:**

```json
{
  "sub": "optional@gmail.com",
  "name": "optional",
  "role": "customer",
  "brand": "optional brand",
  "iat": 1780938466,
  "exp": 1780967266
}
```

The {{< badge blue >}}role{{< /badge>}} field in the payload immediately stands out. The server appears to be using this claim to control access. The question is: can we tamper with it?

---

## Attempt a Naive Role Change

The first instinct is to simply change {{< badge blue >}}"role": "customer"{{< /badge>}} to {{< badge gray >}}"role": "admin"{{< /badge>}} and re-sign the token. Predictably, this fails - the server validates the RS256 signature and redirects us back to {{< badge yellow >}}/login{{< /badge>}}.

We also tried providing an unsigned JWT with {{< badge gray >}}alg: none{{< /badge>}}, but the server declines unsigned tokens as well.

We need a different approach.

---

## Algorithm Confusion Attack

Checking the API docs reveals two interesting endpoints - a **JWKS endpoint** and a **public key endpoint**.

{{< figure align="center" src="/images/Halftone_Studio_4.png" caption="API Docs - JWKS & Public Key Endpoints" >}}

{{< figure align="center" src="/images/Halftone_Studio_5.png" caption="JWKS Endpoint" >}}


{{< figure align="center" src="/images/Halftone_Studio_6.png" caption="Public Key Endpoint" >}}


Attempting to sign with the public keys exposed at those endpoints doesn't work - they appear to be outdated.

Since the server is using **RS256** (asymmetric signing), there is a well-known attack path: if the server code uses the same key object to verify both RS256 and HS256 tokens without strictly enforcing the algorithm, we can:

1. Derive the RSA public key from two valid JWTs
2. Use that public key as the **HMAC secret** (HS256)
3. Sign a tampered token with {{< badge red >}}alg: HS256{{< /badge>}} - the server will verify it using the public key as the HMAC secret, which we already know

This is the **Algorithm Confusion (RS256 → HS256)** attack.

---

## Deriving the Public Key

To perform this attack we need two valid JWTs signed by the same key. Log out and log back in twice to collect them.

We use the {{< badge purple >}}portswigger/sig2n{{< /badge>}} Docker tool to derive the RSA public key from the two tokens:

```bash
$ docker run --rm -it portswigger/sig2n \
  eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Im1haW4ifQ.eyJzdWIiOiJvcHRpb25hbEBnbWFpbC5jb20iLCJuYW1lIjoib3B0aW9uYWwiLCJyb2xlIjoiY3VzdG9tZXIiLCJicmFuZCI6Im9wdGlvbmFsIGJyYW5kIiwiaWF0IjoxNzgwODU3MjYwLCJleHAiOjE3ODA4ODYwNjB9.OcdYXEkOjlrfdAtBDhiSF4t9U0wBWTLEFiu6AraP5F9y8xRfiEZsOzP5kqPYW4gPYYHH8d5Yh25JOU-XD9tD1Cp7TsHoWdl8GDzPmeccnfC1mv3MTmoNaZNKH3MWDKZsrwHTqiUX1xuH3AncKjbStIUc1ZNj6yqGi6NYVMaNJxhOj_K2rVsq3yEJXfI-1IbC85344PTPSoAqgq1S6adBLkiVlQWPwh3Ag_rbqVwsaB1f5LZx4PjV09vPx9ZnoCcqWVI2IBHtyod-2RhrM9BXA0KeTVXDDGgZ3-yqO9tR0wi3uBthdAurJC-7a3IpcskHYmwO1FKn1eq_y2j66AkqlA \
  eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Im1haW4ifQ.eyJzdWIiOiJvcHRpb25hbEBnbWFpbC5jb20iLCJuYW1lIjoib3B0aW9uYWwiLCJyb2xlIjoiY3VzdG9tZXIiLCJicmFuZCI6Im9wdGlvbmFsIGJyYW5kIiwiaWF0IjoxNzgwODU3OTc0LCJleHAiOjE3ODA4ODY3NzR9.Aahk3gouYU3ZgrdToKfsWkXfLpbdGi5jsERZtHJ2HDpmPYJ-dr9yirX-j7-6nfxT_B6evUJptRB_X8qHkQcC_oKJnd_Ct5RYsQzq53UOqhuls_5M9oxqqPrUO5K5bUXDDkVVOs73pCzKU8QnnXPNYCnPecwj7O9TNcE1ZsbppWmx9no7T8BE9SFcNkYEHjfBLlqy47ppi5sONlFlGZ4wMFCNge0XGnCkBXtr8CLG8S-mEV__8X0-DzAmIr45_2qFU9WkCuZGmsFUNxKvgZLFfmbmoPaf6h6xbtMaUspnLRBplMHXVuxWtfDZsm0-3fv6FwbAC0mH5vRSaRO6OFcrpw
```

The tool outputs two candidate public keys (x509 and PKCS#1 formats) along with a tampered JWT for each. We test both tampered JWTs against the server - the one that keeps us logged in tells us which public key is correct.

{{< figure align="center" src="/images/Halftone_Studio_7.png" caption="sig2n output - derived public key candidates and tampered JWTs" >}}


{{< figure align="center" src="/images/Halftone_Studio_8.png" caption="Testing tampered JWTs to identify the valid public key" >}}


---

## Forge the Token

Once we know which public key is valid, we use **JWT Editor** in Burp Suite to craft our final malicious token:

1. Go to **JWT Editor → New Symmetric Key → Generate**.
2. Replace the value of {{< badge purple >}}k{{< /badge>}} with the **Base64 encoded public key** from the {{< badge gray >}}sig2n{{< /badge>}} output.
3. In the **JSON Web Token** tab, change {{< badge purple >}}alg{{< /badge>}} to {{< badge red >}}HS256{{< /badge>}}.
4. Change {{< badge purple >}}role{{< /badge>}} to {{< badge red >}}admin{{< /badge>}}.
5. Sign the token with the symmetric key you just created.

{{< figure align="center" src="/images/Halftone_Studio_9.png" caption="Creating a Symmetric Key with the derived public key" >}}


{{< figure align="center" src="/images/Halftone_Studio_10.png" caption="Changing alg to HS256 and role to admin, then signing" >}}


---

## Replace the Cookie & Reload

We open the browser's DevTools (or a cookie editor extension), find the JWT cookie, and replace its value with our forged token.

After reloading the page, the server accepts the token and grants us admin access. The role escalation is complete.

{{< figure align="center" src="/images/Halftone_Studio_11.png" caption="Admin Access Granted" >}}


{{< figure align="center" src="/images/Halftone_Studio_12.png" caption="Flag" >}}


---

## Why This Works

The root cause is that the server's JWT verifier does **not strictly enforce the signing algorithm**. When it receives a token claiming {{< badge red >}}alg: HS256{{< /badge>}}, it verifies the HMAC signature using the RSA public key as the secret -- a key the attacker can derive from any two valid tokens. A properly implemented verifier should:

- Accept **only** the expected algorithm (RS256) and reject anything else.
- Never use an asymmetric public key as an HMAC secret.
- Treat the {{< badge green >}}alg{{< /badge>}} header as untrusted input.



## Remediation

| Issue | Fix |
| --- | --- |
| Server accepts {{< badge gray >}}alg: HS256{{< /badge>}} when RS256 is expected | Hardcode and enforce the expected algorithm server-side |
| RSA public key derivable from issued tokens | Not directly preventable, but mitigated by strict algorithm enforcement |
| Role stored in JWT payload | Validate roles server-side against a database, not just the token |
| Exposed JWKS / public key endpoints with stale keys | Rotate keys and audit all exposed key material |

---

{{< callout tip "The takeaway">}}

when an asymmetric signing scheme is in use, the verifier must never accept a symmetric algorithm. An attacker who can observe two tokens can recover the public key -- and if HS256 is accepted, that public key becomes their signing secret.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/challenges/mirrored
{{< /callout>}}