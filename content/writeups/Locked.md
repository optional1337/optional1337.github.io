---
title: "WebVerse: Locked"
date: 2026-06-16
weight: 4
tags: ["webverse","Broken Auth"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Broken authentication via exposed .git repository and forged remember_me cookie — Challenge Lab Writeup."
summary: "Broken authentication via exposed .git repository and forged remember_me cookie — Challenge Lab Writeup."
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/locked_cover.svg"
    alt: "Locked challenge"
    caption: "Locked challenge (Broken Auth)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **Locked**

**Category:** Web Security / Broken Authentication

**Vulnerability:** Forged **remember_me cookie** via exposed `.git` repository and unauthenticated API endpoint

**Platform:** webverse

{{< /callout>}}

{{< figure align="center" src="/images/Locked_1.png" caption="*Locked Challenge*" >}}

## Lab Description

{{< callout insight "Briefing" >}}
*There's no sign-up screen worth trying — you're added by someone already inside, and the team treats that as the whole security model. One server, one door, and a lot of trust placed in it. Get yourself in and read the room.*
{{< /callout>}}

---

## Visiting the Site

Navigating to the challenge URL takes us straight to a login page. The form asks for a work email and password — no registration option anywhere in sight.

{{< figure align="center" src="/images/Locked_2.png" caption="*Login page — no self-registration*" >}}

We try a handful of default and weak credentials (`admin:admin`, `admin:password`, and so on) — all rejected. We also attempt a basic SQL injection bypass in the email field:

```
test' or 1=1-- -
```

{{< figure align="center" src="/images/Locked_3.png" caption="*SQLi bypass attempt — credentials not recognised*" >}}

The server returns *"Those credentials were not recognised."* for everything we throw at it. The login logic isn't injectable, so we need a different angle.


## Recon — Directory Fuzzing

With the login locked down, we run {{< badge purple >}}dirsearch{{< /badge >}} against the root to look for anything interesting.

```bash
$ dirsearch -u https://16e6ef18-4575-locked-49562.events.webverselabs-pro.com
```

{{< figure align="center" src="/images/Locked_4.png" caption="*dirsearch — exposed .git directory discovered*" >}}

Two results stand out immediately: a {{< badge yellow >}}301{{< /badge >}} redirect to {{< badge gray >}}/.git/{{< /badge >}} and a {{< badge green >}}200{{< /badge >}} on {{< badge gray >}}/.git/COMMIT_EDITMSG{{< /badge >}}. The `.git` directory is accessible from the web — the whole repository can be pulled.



## Dumping the Git Repository

We use {{< badge purple >}}git-dumper{{< /badge >}} to pull every reachable object from the exposed {{< badge gray >}}/.git{{< /badge >}} endpoint:

```bash
$ git-dumper https://16e6ef18-4575-locked-49562.events.webverselabs-pro.com/ ./web
```

{{< figure align="center" src="/images/Locked_5.png" caption="*git-dumper — fetching repository objects*" >}}

With the repo on disk, we check the commit history:

```bash
$ git log
```

{{< figure align="center" src="/images/Locked_6.png" caption="*git log — commit history*" >}}

One commit message catches the eye: **"Replace members API with v2 version"**. That tells us there was a v1 of the members API that was replaced — and replaced APIs often stick around.


## Reading the Source Code

We dig into the recovered PHP source. First, {{< badge gray >}}login.php{{< /badge >}}:

{{< figure align="center" src="/images/Locked_7.png" caption="*login.php — locked_user_by_email() handles credential lookup*" >}}

The login logic is straightforward — it calls {{< badge blue >}}locked_user_by_email(){{< /badge >}} and checks the password hash with {{< badge blue >}}password_verify(){{< /badge >}}. No injection surface. But there's a second authentication path: the {{< badge yellow >}}remember_me{{< /badge >}} cookie.

Opening {{< badge gray >}}auth.php{{< /badge >}} reveals exactly how it works:

{{< figure align="center" src="/images/Locked_8.png" caption="*auth.php — remember_me cookie decoded as base64(email:uuid)*" >}}

```php
function locked_user_from_remember(): ?array {
    $raw     = $_COOKIE['remember_me'] ?? '';
    $decoded = base64_decode($raw, true);
    [$email, $uuid] = explode(':', $decoded, 2);
    $user = locked_user_by_email($email);
    if (!hash_equals((string) $user['uuid'], $uuid)) {
        return null;
    }
    return $user;
}
```

The cookie is simply {{< badge red >}}base64(email:uuid){{< /badge >}}. If we know a valid user's email **and** their UUID, we can forge it — no signature, no secret. The only question is: where do we get those values?


## Finding the Old Members API

The commit message mentioned a v1 members API. We fuzz the {{< badge gray >}}/api/v2/{{< /badge >}} path to confirm what was replaced:

```bash
$ dirsearch -u https://16e6ef18-4575-locked-49562.events.webverselabs-pro.com/api/v2/
```

{{< figure align="center" src="/images/Locked_9.png" caption="*dirsearch — /api/v2/members returns 401 Unauthorized*" >}}

The v2 endpoint exists at {{< badge gray >}}/api/v2/members{{< /badge >}} but returns {{< badge red >}}401 Unauthorized{{< /badge >}} — authentication is now enforced on the new version.

{{< figure align="center" src="/images/Locked_10.png" caption="*/api/v2/members — authentication required*" >}}

The old version, however, was never taken down. Visiting {{< badge green >}}/api/v1/members{{< /badge >}} returns the full member directory — no authentication required:

{{< figure align="center" src="/images/Locked_11.png" caption="*/api/v1/members — unauthenticated, leaks email and UUID for all members*" >}}

The response lists every member's {{< badge blue >}}display_name{{< /badge >}}, {{< badge blue >}}email{{< /badge >}}, {{< badge blue >}}uuid{{< /badge >}}, and {{< badge blue >}}role{{< /badge >}}. At the top of the list: **Supt. Margaret Whitlock** — {{< badge red >}}role: admin{{< /badge >}}.

```
email: m.whitlock@meadowbank.police.uk
uuid:  fcb95511-94fa-4c0c-bbbf-36fc8dd2c7f4
```


## Forging the Cookie

With the admin's email and UUID in hand, we construct the forged {{< badge yellow >}}remember_me{{< /badge >}} cookie. The format is {{< badge red >}}base64(email:uuid){{< /badge >}}, so we use {{< badge purple >}}CyberChef{{< /badge >}} to encode:

```
m.whitlock@meadowbank.police.uk:fcb95511-94fa-4c0c-bbbf-36fc8dd2c7f4
```

{{< figure align="center" src="/images/Locked_12.png" caption="*CyberChef — base64 encoding email:uuid*" >}}

This gives us:

```
bS53aGl0bG9ja0BtZWFkb3diYW5rLnBvbGljZS51azpmY2I5NTUxMS05NGZhLTRjMGMtYmJiZi0zNmZjOGRkMmM3ZjQ=
```


## Injecting the Cookie & Gaining Access

We open {{< badge purple >}}Cookie-Editor{{< /badge >}} (or DevTools → Application → Cookies) and add a new entry:

- **Name:** {{< badge yellow >}}remember_me{{< /badge >}}
- **Value:** the base64 string above

{{< figure align="center" src="/images/Locked_13.png" caption="*Cookie-Editor — setting the forged remember_me cookie*" >}}

Refreshing the page, the server decodes the cookie, looks up {{< badge blue >}}m.whitlock@meadowbank.police.uk{{< /badge >}}, confirms the UUID matches, and logs us in as Supt. Margaret Whitlock — {{< badge red >}}admin{{< /badge >}}.

We navigate to the {{< badge gray >}}#command{{< /badge >}} channel, restricted to supervisors and command staff, and find the flag waiting in a message from the superintendent herself:

{{< figure align="center" src="/images/Locked_14.png" caption="*#command channel — flag retrieved*" >}}

---

## Why This Works

The vulnerability is a chain of three individually weak decisions that together result in full authentication bypass:

1. **Exposed {{< badge gray >}}.git{{< /badge >}} directory.** The production web server serves the entire `.git` folder, allowing anyone to recover the full application source code with {{< badge purple >}}git-dumper{{< /badge >}}.

2. **Unsigned {{< badge yellow >}}remember_me{{< /badge >}} cookie.** The persistent-login cookie is just {{< badge red >}}base64(email:uuid){{< /badge >}} — no HMAC, no signature, nothing to prevent forgery. Any attacker who knows a valid email/UUID pair can impersonate that user.

3. **Unauthenticated {{< badge green >}}/api/v1/members{{< /badge >}} left running.** When the members API was upgraded to v2 (with authentication), the old endpoint was never removed. It leaks every member's email, UUID, and role to anyone who asks — handing an attacker exactly the values needed to forge the cookie.



## Remediation

| Issue | Fix |
| --- | --- |
| {{< badge gray >}}.git{{< /badge >}} directory served publicly | Add `Deny from all` / `location /.git { deny all; }` in the web server config; verify with directory listing scans before going live |
| {{< badge yellow >}}remember_me{{< /badge >}} cookie has no integrity protection | Sign the cookie with an HMAC keyed on a server-side secret; reject any cookie whose signature doesn't verify |
| UUID as a sole authenticator | Treat UUIDs as opaque identifiers, not secrets; the HMAC above makes UUID exposure irrelevant, but UUIDs should not be considered secret by design |
| Old {{< badge green >}}/api/v1/members{{< /badge >}} left running unauthenticated | Decommission deprecated API versions entirely; if a migration window is needed, gate the old endpoint behind the same auth as the replacement |
| Member directory (email + UUID + role) exposed without auth | Any endpoint returning PII or security-sensitive data must require authentication and respect the principle of least privilege |


{{< callout tip "The takeaway" >}}

An exposed {{< badge gray >}}.git{{< /badge >}} folder turns a black-box target into a white-box one — you hand the attacker your source code, your logic, and every comment your developers left behind. From there, an unsigned {{< badge yellow >}}remember_me{{< /badge >}} token and a forgotten {{< badge green >}}/api/v1/members{{< /badge >}} endpoint are all it takes to walk straight in through the front door as anyone on the list.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/events/locked

{{< /callout>}}