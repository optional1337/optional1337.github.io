---
title: "WebVerse: Access Control - IDOR & BOLA"
date: 2026-06-12
weight: 5
tags: ["webverse","IDOR","BOLA","Access Control"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
draft: true
description: "Broken Access Control: IDOR & BOLA — Challenge Lab Writeups."
summary: "Broken Access Control: IDOR & BOLA — Challenge Lab Writeups."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
# cover:
#     image: "/images/access_control_cover.svg"
#     alt: "Access Control challenges"
#     caption: "IDOR & BOLA challenges (Access Control)"
#     relative: false
#     hidden: false
---

{{< callout warning >}}

**Lab:** **Coltsfoot Community Center**

**Category:** Web Security / Broken Access Control

**Vulnerability:** Missing Function-Level Access Control (forced browsing to a hidden admin path)

**Platform:** webverse
{{< /callout>}}

## Lab Description

{{< callout insight "Briefing" >}}
*Coltsfoot is run by a three-member board of retirees who took over the operations site from a volunteer web-developer in 2019. Classes start at $12, the board meets the first Tuesday of every month, and the daily totals are reconciled on a Square reader in the back office. The "staff side" of the site, the board insists, has always been hidden — no one outside the building has ever needed it.*
{{< /callout>}}

## Recon: Browsing the Site

The Coltsfoot site is a small community-center page — opening hours, a mission statement, a few links to classes, the board, volunteering, and donations. Nothing here screams "vulnerable", which is exactly the point.

{{< figure align="center" src="/images/Coltsfoot_Community_Center_1.png" caption="Browsing the Coltsfoot homepage" >}}

When testing for {{< badge purple >}}Broken Access Control{{< /badge>}}, one of the first habits worth building is checking the boring, overlooked files every site ships with — {{< badge blue >}}robots.txt{{< /badge>}}, {{< badge blue >}}sitemap.xml{{< /badge>}}, {{< badge blue >}}.well-known/{{< /badge>}}, and so on.

## Checking `robots.txt`

Sure enough, {{< badge blue >}}robots.txt{{< /badge>}} contains a hint the developers probably never meant to ship:

{{< figure align="center" src="/images/Coltsfoot_Community_Center_2.png" caption="Disallow entry pointing at /staff/" >}}

```text
User-agent: *
Disallow: /staff/
```

A {{< badge purple >}}Disallow{{< /badge>}} directive only tells **search engine crawlers** not to index a path — it is **not** an access control mechanism. The browser (and the attacker) is free to ignore it completely.

## Forced Browsing to `/staff/dashboard`

Following the breadcrumb, we navigate directly to {{< badge yellow >}}/staff/dashboard{{< /badge>}} — with **no session, no login, no cookie** at all.

The page loads without any redirect to a login screen, exposing the daily reconciliation totals, card payment figures, the cash drawer, refund counts, and upcoming bookings — and, sitting right inside the "reconciliation reference" box, our flag.

{{< figure align="center" src="/images/Coltsfoot_Community_Center_3.png" caption="Unauthenticated access to /staff/dashboard" >}}

The server never checks **who** is asking for {{< badge yellow >}}/staff/dashboard{{< /badge>}}, only that the URL was requested. Because the only thing "protecting" the staff side was the assumption that nobody would ever guess (or be told) the path, this is a textbook **Broken Access Control** issue — specifically, a missing function-level access control check.

## Why This Works

The root cause is that the {{< badge purple >}}/staff/{{< /badge>}} routes were never wired up to an authentication/authorization middleware. The server **trusts obscurity** instead of **enforcing access**:

- `robots.txt` is a convention for crawlers, not a security boundary.
- Sensitive routes were assumed to be "hidden" simply because no link pointed to them from the public site.
- No session, role, or permission check is performed before rendering the staff dashboard.

## Remediation

| Issue | Fix |
| --- | --- |
| `/staff/*` routes reachable without authentication | Require a valid authenticated session before rendering any staff route |
| Sensitive paths "hidden" only via `robots.txt` | Never rely on `robots.txt` or obscurity to protect sensitive functionality |
| No role check on staff dashboard | Enforce server-side role/permission checks (e.g. `role == staff`) on every privileged endpoint |

---

{{< callout warning >}}

**Lab:** **Remittance**

**Category:** Web Security / IDOR (Insecure Direct Object Reference)

**Vulnerability:** IDOR via user-supplied `user_id` on a data-export endpoint (BOLA)

**Platform:** webverse
{{< /callout>}}

## Lab Description

{{< callout insight "Briefing" >}}
*InvoiceVault is a six-person SaaS shop that's grown to around four thousand freelancer accounts since 2020 — $9/mo, simple invoice CRUD, one feature a quarter. The new account-export endpoint was scaffolded out of an internal admin tool during a frantic week before a customer data-portability deadline. The engineer who wrote it filed a follow-up ticket the same afternoon; it's been sitting at the bottom of the backlog ever since.*
{{< /callout>}}

## Create an Account & Explore

We start the same way as always — register a fresh InvoiceVault account and sign in.

{{< figure align="center" src="/images/Remittance_1.png" caption="InvoiceVault landing page" >}}

{{< figure align="center" src="/images/Remittance_2.png" caption="Creating an account" >}}

Once logged in, we land on an empty {{< badge blue >}}Invoices{{< /badge>}} dashboard — fresh account, nothing to see yet.

{{< figure align="center" src="/images/Remittance_3.png" caption="Empty invoices dashboard for a fresh account" >}}

## Locating the Export Feature

Poking around {{< badge purple >}}Settings{{< /badge>}}, there's a {{< badge green >}}"Full account export"{{< /badge>}} section with a {{< badge yellow >}}Download export{{< /badge>}} button. The description says it bundles {{< badge blue >}}invoices.csv{{< /badge>}} and {{< badge blue >}}account_info.txt{{< /badge>}} into a ZIP — exactly the kind of "data portability" feature mentioned in the briefing.

{{< figure align="center" src="/images/Remittance_4.png" caption="Export account data option in Settings" >}}

## Intercepting the Export Request

With Burp Suite proxying the traffic, clicking {{< badge yellow >}}Download export{{< /badge>}} fires a request to {{< badge purple >}}POST /api/account/export{{< /badge>}}:

```json
{
  "user_id": 3
}
```

The response comes back as a {{< badge green >}}application/zip{{< /badge>}} attachment named {{< badge blue >}}invoicevault-export-2026.zip{{< /badge>}}.

{{< figure align="center" src="/images/Remittance_5.png" caption="Export request sending our own user_id" >}}

The {{< badge purple >}}user_id{{< /badge>}} field immediately stands out. The endpoint isn't deriving the account to export from the **session** — it's trusting a value the **client** sent in the request body. The question is: what happens if we change it?

## Tampering with `user_id`

We turn on intercept, resend the export request, and edit the body so that:

```json
{
  "user_id": 2
}
```

{{< badge red >}}user_id: 3{{< /badge>}} (us) becomes {{< badge red >}}user_id: 2{{< /badge>}} (someone else's account), and we forward the tampered request.

{{< figure align="center" src="/images/Remittance_6.png" caption="Forwarding a request with a tampered user_id" >}}

The server doesn't validate that {{< badge purple >}}user_id{{< /badge>}} belongs to the authenticated session at all — it happily builds and returns an export ZIP for **user 2** instead of our own account.

## Extracting the Flag

Unzipping the downloaded archive and opening {{< badge blue >}}invoices.csv{{< /badge>}}, one of user 2's invoice rows contains a {{< badge green >}}Notes{{< /badge>}} field with our flag sitting right inside it.

{{< figure align="center" src="/images/Remittance_7.png" caption="Flag recovered from another user's exported invoice data" >}}

This is a classic **IDOR / BOLA**: the endpoint authenticates *that* you're logged in, but never authorizes *which* object (account) you're allowed to act on — letting any authenticated user pull **any other user's** full data export simply by changing an ID.

## Why This Works

The export endpoint was lifted from an internal admin tool where any staff member could legitimately export any account by ID. When it was repurposed for end users, the **authorization check never followed it over** — the server still trusts the client-supplied {{< badge purple >}}user_id{{< /badge>}} rather than deriving it from the session.

- The session cookie identifies *who* is logged in.
- The {{< badge purple >}}user_id{{< /badge>}} in the request body decides *whose data* gets exported.
- Nothing ties the two together.

## Remediation

| Issue | Fix |
| --- | --- |
| `user_id` taken from client-supplied request body | Derive the target account from the authenticated session, never from client input |
| No ownership check before export | Verify the requested object belongs to the authenticated user before processing |
| Admin-tool endpoint reused for end users without re-review | Re-audit authorization logic whenever internal tooling is exposed to regular users |

---

{{< callout tip "The takeaway">}}

Authentication answers *"who are you?"* — authorization has to separately answer *"are you allowed to touch **this** object?"*. Skipping either one is what turns a normal feature into IDOR/BOLA.

*Do these labs on webverse Now:*

- Coltsfoot Community Center: https://dashboard.webverselabs-pro.com/challenges/coltsfoot-community-center
- Remittance: https://dashboard.webverselabs-pro.com/challenges/remittance

{{< /callout>}}