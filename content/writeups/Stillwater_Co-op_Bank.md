---
title: "WebVerse: Stillwater Co-op Bank"
date: 2026-06-14
weight: 5
tags: ["webverse","JWT"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "JWT kid Path Traversal attack Challenge Lab Writeup."
summary: "JWT kid Path Traversal attack Challenge Lab Writeup."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/stillwater_coop_bank_cover.svg"
    alt: "Stillwater Co-op Bank challenge"
    caption: "Stillwater Co-op Bank challenge (JWT)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **Stillwater Co-op Bank**

**Category:** Web Security / JWT

**Vulnerability:** JWT **`kid` Path Traversal** (privilege escalation)

**Platform:** webverse
{{< /callout>}}

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_1.png" caption="*Stillwater Co-op Bank Challenge*" >}}

## Lab Description

{{< callout insight "Briefing" >}}
*Stillwater Co-op Bank has served its community since 1958 — four branches across two counties, twelve thousand members, share-draft accounts and small consumer loans only. The online portal was built one summer by the treasurer's nephew and bolted onto the back-office wire-approvals queue a few years later when remote staff outgrew the paper log. The board's compliance review last quarter flagged the whole stack as "long overdue for outside eyes."*
{{< /callout>}}

---

## Create an Account

We navigate to the registration page and open an account with a dummy name, email, and password.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_2.png" caption="Registering a new member account" >}}


## Inspect the JWT

After registration, the server redirects us to {{< badge yellow >}}/dashboard{{< /badge>}}. Intercepting this in Burp Suite, we can see a cookie named {{< badge blue >}}sw_token{{< /badge>}} being sent. The **Inspector** panel on the right decodes it on the fly, and switching to the **JSON Web Token** tab gives us the full structure.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_3.png" caption="Burp HTTP history — sw_token cookie visible in the /dashboard request" >}}

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_4.png" caption="JWT decoded — Header and Payload" >}}

**Header:**

```json
{
  "alg": "HS256",
  "kid": "2024-q4.pem",
  "typ": "JWT"
}
```

**Payload:**

```json
{
  "role": "member",
  "member_no": "91000-8000",
  "iat": 1781420804,
  "exp": 1781449604
}
```

Two things stand out immediately:

- The {{< badge blue >}}role{{< /badge>}} claim drives access control — changing it to {{< badge red >}}admin{{< /badge>}} is the goal.
- The {{< badge yellow >}}kid{{< /badge>}} header tells the server *which key file to load* when verifying the HMAC signature. If we control this value, we control which file on disk is used as the secret.

---

## The Vulnerability — `kid` Path Traversal

The {{< badge yellow >}}kid{{< /badge>}} (Key ID) parameter is supposed to be an opaque identifier that looks up a key in a safe store. When the server instead passes this value directly to a file-read call without sanitising it, an attacker can supply a path like {{< badge gray >}}../../dev/null{{< /badge>}} to redirect key lookup to any readable file on the system.

With {{< badge purple >}}HS256{{< /badge>}} in play — a symmetric algorithm — whoever knows the secret can produce a valid signature. By pointing {{< badge yellow >}}kid{{< /badge>}} at a file with empty or known content, we become the signer.

We have two exploitation paths.

## Method 1 — `kid` → `/dev/null` (Null Byte Secret)

Reading {{< badge gray >}}/dev/null{{< /badge>}} on Linux always returns an empty byte sequence. This means the HMAC secret the server loads is empty — representable in Base64 as {{< badge green >}}AA=={{< /badge>}}. We craft a symmetric key with that exact value, sign a tampered token, and the server's verification will match.

**Step 1 — Create a symmetric key with a null byte secret**

In Burp's **JWT Editor Keys** tab, click **New Symmetric Key → Generate**, then manually replace the {{< badge purple >}}k{{< /badge>}} value with {{< badge green >}}AA=={{< /badge>}}.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_5.png" caption="New Symmetric Key — k set to AA== (null byte)" >}}

**Step 2 — Modify the token**

In the **JSON Web Token** tab of the intercepted request:

- Set {{< badge yellow >}}kid{{< /badge>}} to {{< badge gray >}}../../dev/null{{< /badge>}}
- Set {{< badge blue >}}role{{< /badge>}} to {{< badge red >}}admin{{< /badge>}}

**Step 3 — Sign**

Click **Sign**, select the symmetric key we just created, choose {{< badge gray >}}Don't modify header{{< /badge>}}, and hit OK. The response panel on the right immediately shows the {{< badge red >}}Admin{{< /badge>}} nav item — the forged token is already accepted.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_6.png" caption="Token signed with null byte key — Admin tab appears in the response" >}}

**Step 4 — Replace the cookie**

Open Cookie Editor, find {{< badge blue >}}sw_token{{< /badge>}}, paste in the forged JWT, and save.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_7.png" caption="Replacing sw_token in Cookie Editor" >}}

Reload the page — the server loads {{< badge gray >}}/dev/null{{< /badge>}} as the key, gets an empty byte, and verifies the HMAC we signed with that same empty byte. Admin access granted.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_8.png" caption="Admin Wire Approvals panel — flag visible" >}}

## Method 2 — `kid` → `/proc/sys/kernel/randomize_va_space` (Known File Content)

Sometimes a target hardens the null-byte path — the server may reject an empty secret or block {{< badge gray >}}/dev/null{{< /badge>}} explicitly. In that case we need a file with *known, predictable content*.

{{< badge gray >}}/proc/sys/kernel/randomize_va_space{{< /badge>}} is a Linux kernel tunable that stores the ASLR randomisation level. On virtually every default Linux system it contains a single character: {{< badge green >}}2{{< /badge>}} followed by a newline ({{< badge green >}}\n{{< /badge>}}).

> **Credit:** Thanks to **ShadowHound** on the WebVerse Discord — signing with just {{< badge green >}}2{{< /badge>}} does not work. You must include the trailing newline, so the actual secret is {{< badge green >}}2\n{{< /badge>}}.

**Step 1 — Find the Base64 of `2\n`**

Open [CyberChef](https://cyberchef.io/#recipe=To_Base64('A-Za-z0-9%2B/%3D')), type {{< badge green >}}2{{< /badge>}} in the Input box, press Enter to add a blank line (the {{< badge green >}}\n{{< /badge>}}), and run **To Base64**. The output is {{< badge green >}}Mgo={{< /badge>}}.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_9.png" caption="CyberChef — Base64 of '2\\n' outputs Mgo=" >}}

**Step 2 — Create a symmetric key with `Mgo=`**

In Burp's **JWT Editor Keys** tab, click **New Symmetric Key → Generate**, then set {{< badge purple >}}k{{< /badge>}} to {{< badge green >}}Mgo={{< /badge>}}.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_10.png" caption="New Symmetric Key — k set to Mgo=" >}}

**Step 3 — Modify and sign the token**

- Set {{< badge yellow >}}kid{{< /badge>}} to {{< badge gray >}}../../proc/sys/kernel/randomize_va_space{{< /badge>}}
- Set {{< badge blue >}}role{{< /badge>}} to {{< badge red >}}admin{{< /badge>}}
- Click **Sign**, select the {{< badge green >}}Mgo={{< /badge>}} key, choose {{< badge gray >}}Don't modify header{{< /badge>}}, and hit OK.

{{< figure align="center" src="/images/Stillwater_Co-op_Bank_11.png" caption="Token signed with Mgo= key — Admin tab appears in the response" >}}

Replace the cookie the same way as Method 1 — admin access granted.

---

## Why This Works

The server treats the {{< badge yellow >}}kid{{< /badge>}} header as a trusted file path, reads whatever file it points to, and uses that content as the HMAC secret. Since {{< badge purple >}}HS256{{< /badge>}} is symmetric, anyone who knows the secret can produce a signature the server will accept.

By redirecting {{< badge yellow >}}kid{{< /badge>}} to a file with empty or known content, we know the secret before the server even reads it. The forged token passes verification because we signed with the exact same bytes the server loads from disk.

A correct implementation should:

- Treat {{< badge yellow >}}kid{{< /badge>}} as an **opaque identifier** looked up in a hardcoded allowlist — never as a filesystem path.
- Sanitise or reject any {{< badge yellow >}}kid{{< /badge>}} containing path separators ({{< badge gray >}}/{{< /badge>}}, {{< badge gray >}}..{{< /badge>}}) or null bytes.
- Store signing secrets in a proper secrets manager, not as files reachable from the app's read path.
- Validate loaded key material meets minimum entropy requirements before using it.


## Remediation

| Issue | Fix |
| --- | --- |
| {{< badge yellow >}}kid{{< /badge>}} passed directly to a file-read operation | Validate {{< badge yellow >}}kid{{< /badge>}} against a strict allowlist; never treat it as a path |
| {{< badge gray >}}/dev/null{{< /badge>}} usable as an HMAC secret | Reject empty or zero-entropy secrets at key-load time |
| Known-content kernel files usable as secrets | Enforce minimum key entropy; reject single-byte secrets |
| {{< badge blue >}}role{{< /badge>}} claim trusted from the JWT payload | Authorise server-side against a database; never rely solely on token claims |


{{< callout tip "The takeaway">}}

The {{< badge yellow >}}kid{{< /badge>}} header is attacker-controlled input. Any server that feeds it directly into a {{< badge gray >}}readFile(){{< /badge>}} call has handed the attacker a path traversal primitive — and with a symmetric algorithm like {{< badge purple >}}HS256{{< /badge>}}, controlling the key file means controlling the signature. Two methods, one root cause: unsanitised {{< badge yellow >}}kid{{< /badge>}}.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/03/safe-deposit

{{< /callout>}}