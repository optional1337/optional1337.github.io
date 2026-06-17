---
title: "WebVerse: MeltDown"
date: 2026-06-17
weight: 4
tags: ["webverse","LDAP Injection"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "LDAP Injection Authentication Bypass Challenge Lab Writeup."
summary: "LDAP Injection Authentication Bypass Challenge Lab Writeup."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/meltdown_cover.svg"
    alt: "MeltDown challenge"
    caption: "MeltDown challenge (LDAP Injection)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **MeltDown**

**Category:** Web Security / Authentication

**Vulnerability:** **LDAP Injection** (authentication bypass)

**Platform:** webverse

{{< /callout>}}

{{< figure align="center" src="/images/MeltDown_1.png" caption="*MeltDown Challenge*" >}}

## Lab Description

{{< callout insight "Briefing" >}}
*Hadley Ridge has powered the valley since 1971. The customer site is tidy and corporate — but the same login the operators use to reach the control-room portal was wired up years ago and never revisited.*
{{< /callout>}}

---

## Visiting the Site

The target is a fairly standard utility-company marketing site for **Hadley Ridge Power & Light** — services, coverage, leadership, an outage hotline, and a contact form.

{{< figure align="center" src="/images/MeltDown_2.png" caption="*Hadley Ridge Power & Light - Homepage*" >}}

Nothing about the public-facing pages screams "vulnerable" at first glance, so the next step is to poke at the only piece of the site that takes user input: the contact form.

## Testing the Contact Form

We fill out the contact form with a mix of dummy values and common injection probes, looking for anything reflected back to us — an error message, a stray HTML tag, anything that hints at how the backend processes input.

{{< figure align="center" src="/images/MeltDown_3.png" caption="*Testing the contact form with dummy data*" >}}

The form just accepts the submission and shows a generic confirmation message. No reflection, no errors — a dead end.

{{< figure align="center" src="/images/MeltDown_4.png" caption="Contact form accepts the submission with no reflection" >}}

## Discovering an Admin Endpoint

With the contact form giving us nothing, we run a content discovery scan against the site with **dirsearch**.

{{< figure align="center" src="/images/MeltDown_5.png" caption="*dirsearch enumerating the site*" >}}

Buried among the usual marketing pages, one path stands out: {{< badge yellow >}}/admin/login{{< /badge>}} — returning a `200`.


## The Admin Login Portal

Navigating there reveals an **Operator Portal** — described as the login for "authorized operations staff" tied to the company's SCADA gateway. This lines up with the briefing's mention of an operator login that was "wired up years ago and never revisited."

{{< figure align="center" src="/images/MeltDown_6.png" caption="*Operator Portal login - SCADA Gateway*" >}}


## Probing with Default Credentials

Before reaching for anything fancy, we try a few default/common usernames — {{< badge gray >}}admin{{< /badge>}} included — with a placeholder password, just to see how the portal responds.

{{< figure align="center" src="/images/MeltDown_7.png" caption="*Login attempt with admin / placeholder password*" >}}

The login fails, but the error message is the interesting part:

{{< badge red >}}Failed to Authenticate to Directory Server{{< /badge>}}

That single line leaks the authentication backend: this isn't checking credentials against a local database — it's binding against a **directory server**, almost certainly **LDAP**. That immediately raises the question of whether the login form is building an LDAP search filter directly from user input.


## Forming the LDAP Injection Hypothesis

A classic vulnerable LDAP authentication check looks roughly like:

```
(&(uid={username})(userPassword={password}))
```

If the username and password are concatenated into that filter without sanitization, an attacker who can inject LDAP metacharacters (`*`, `(`, `)`, `|`, `&`) can manipulate the boolean logic of the filter itself — similar in spirit to SQL injection, but against a directory service instead of a database.

We first try a bare wildcard in the username field:

{{< badge gray >}}*{{< /badge>}}

This alone doesn't get us in — the filter is malformed enough that the server still rejects it. We turn to the [HackTricks LDAP Injection reference](https://hacktricks.wiki/en/pentesting-web/ldap-injection.html) for a payload that closes out the filter groups correctly while forcing an always-true condition.

## Crafting the Injection Payload

Using the username field, we submit:

{{< badge red >}}*)(|(&{{< /badge>}}

The idea is to close the intended `uid=...` comparison early with `*)`, then reopen a new `OR` group with `(|(&` so that the rest of the filter — including the password check — gets folded into a branch that the directory server evaluates as true, regardless of what password was actually supplied.

{{< figure align="center" src="/images/MeltDown_8.png" caption="*Submitting the LDAP injection payload in the username field*" >}}


## Access Granted

The portal accepts the malformed filter as a successful authentication and drops us straight into the **Control Room Overview** — signed in as an operator, with a "restricted" SCADA gateway integration key visible on the dashboard.

{{< figure align="center" src="/images/MeltDown_9.png" caption="Operator Portal - authenticated via LDAP injection, flag visible on the SCADA Gateway panel" >}}

The authentication bypass is complete — no valid credentials were ever needed.

---

## Why This Works

The login endpoint builds an LDAP search filter by directly concatenating user-supplied input into the query string, instead of treating that input as data. Because special LDAP filter characters (`*`, `(`, `)`, `|`, `&`) aren't escaped or rejected, an attacker can restructure the filter's boolean logic so the authentication check always evaluates to true — bypassing the password check entirely.

The verbose error message (**"Failed to Authenticate to Directory Server"**) made this far easier to find, since it told us exactly which backend technology to target.

---

## Remediation

| Issue | Fix |
| --- | --- |
| User input concatenated directly into the LDAP filter | Use parameterized/escaped LDAP queries; never build filter strings via raw concatenation |
| LDAP metacharacters (`*`, `(`, `)`, `|`, `&`) accepted unescaped | Escape or strip LDAP special characters from all user input before building filters |
| Verbose backend error ("Directory Server") shown to the user | Return generic authentication-failure messages; don't disclose backend technology |
| No apparent lockout/rate limiting on the login form | Add account lockout and rate limiting on repeated failed attempts |

---

{{< callout tip "The takeaway">}}

Whenever a login form's error messages hint at a backend technology — a directory server, a specific database engine, an LDAP bind failure — it's worth testing whether that backend's query language can be injected through unsanitized input. LDAP injection gets far less attention than SQL injection, but the underlying mistake is the same: treating user input as trusted code rather than data.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/events/meltdown

{{< /callout>}}