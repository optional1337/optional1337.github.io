---
title: "WebVerse: Junior Web Hacker (Web Foundations)"
date: 2026-06-09
weight: 4
tags: ["webverse","Web Foundations","Junior Web Hacker"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Web Foundations:  Ten beginner web security challenge labs writeup."
summary: "Web Foundations:  Ten beginner web security challenge labs writeup."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/web_foundations_cover.svg"
    alt: "Web Foundations"
    caption: "WebVerse: Junior Web Hacker (Web Foundations)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Web Foundations**:  Ten beginner web security challenge labs writeup.

**Platform:** webverse

{{< /callout>}}

---

## #1: Header Hunt

{{< callout info >}}
**Category:** Web Security / HTTP Headers

**Vulnerability:** Sensitive data exposed in HTTP response headers
{{< /callout>}}

{{< figure align="center" src="/images/Header_Hunt_1.png" caption="Header Hunt challenge" >}}

### Lab Description
{{< callout insight "Briefing" >}}

*Arc Logistics — a mid-sized regional freight carrier — just launched their public shipment tracking site. The build team rushed to hit the Q2 deadline and merged a debugging branch the night before launch. Nobody asked what got left behind.*

{{< /callout>}}

### Visit the Site

We start by visiting the webpage normally. The UI looks like a standard shipment tracking page nothing immediately suspicious on the surface.

{{< figure align="center" src="/images/Header_Hunt_2.png" caption="Visiting the Webpage" >}}


### Inspect the Response Headers

We open the browser's DevTools (or intercept the request in Burp Suite) and check the HTTP response headers for the {{< badge blue >}}/{{< /badge>}} request.

There we find a non-standard header: {{< badge purple >}}X-Internal-Order-Ref{{< /badge>}} that the debugging branch left behind. Its value is our flag.

{{< figure align="center" src="/images/Header_Hunt_3.png" caption="X-Internal-Order-Ref Header containing the Flag" >}}



### Why This Works

A development/debugging branch was merged into production without review. HTTP response headers are visible to anyone who inspects network traffic - they are not a safe place to store internal references, tokens, or any sensitive data.

### Remediation

| Issue | Fix |
| --- | --- |
| Debug headers left in production | Audit response headers before deploying; strip internal headers at the reverse proxy |
| No pre-launch header review | Add a security checklist item to verify headers in staging |

{{< callout tip "The takeaway" >}}
HTTP headers are public. Whatever you put in them, assume anyone can read it.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/header-hunt

{{< /callout>}}

---

## #2: Cookie Cutter

{{< callout info >}}
**Category:** Web Security / Cookies

**Vulnerability:** Sensitive data stored in a Base64-encoded cookie (no encryption)

{{< /callout>}}

{{< figure align="center" src="/images/Cookie_Cutter_1.png" caption="Cookie Cutter Challenge" >}}

### Lab Description

{{< callout insight "Briefing" >}}

*Northbrew Coffee Co. — an indie roaster with eleven cafés across the Pacific Northwest — just launched their loyalty rewards site. The engineering team baked their account state into a single browser cookie so the experience would feel instant. They picked an encoding that's easy to read on the wire and easier still to read off it.*

{{< /callout>}}

### Visit the Site

We visit the webpage. The loyalty rewards UI loads up fine.

{{< figure align="center" src="/images/Cookie_Cutter_2.png" caption="*Visiting the Webpage*" >}}


### Inspect the Cookie

We open DevTools (or in burp) and check the cookies set on the {{< badge blue >}}/{{< /badge>}} request. We notice a cookie with a value that looks like Base64 - the telltale padding characters ({{< badge blue >}}={{< /badge>}}) and alphanumeric character set give it away.

{{< figure align="center" src="/images/Cookie_Cutter_3.png" caption="*Base64 Encoded Cookie*" >}}

### Decode the Cookie

We paste the cookie value into [CyberChef](https://cyberchef.io/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)) using the **From Base64** recipe.

The decoded output contains our flag directly inside the cookie value.

{{< figure align="center" src="/images/Cookie_Cutter_4.png" caption="*Decoded Cookie — Flag Revealed*" >}}



### Why This Works

Base64 is an **encoding**, not **encryption**. It is trivially reversible by anyone who intercepts or reads the cookie. Storing sensitive account state or flags inside a client-side cookie — regardless of encoding — means the data is entirely in the user's hands.

### Remediation

| Issue | Fix |
| --- | --- |
| Sensitive data stored client-side in a cookie | Keep sensitive state server-side; store only an opaque session ID in the cookie |
| Base64 mistaken for security | Use proper encryption (AES-GCM) or signed tokens (HMAC) if client-side storage is required |

{{< callout tip "The takeaway" >}}

encoding is not encryption. If a user can see the cookie, they can decode it.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/cookie-cutter

{{< /callout>}}

---

## #3: Session Swap

{{< callout info >}}
**Category:** Web Security / Access Control

**Vulnerability:** Role-based access control enforced via a client-editable cookie

{{< /callout>}}

{{< figure align="center" src="/images/Session_Swap_1.png" caption="*Session Swap Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

*Ridgeline Hotels rolled out a brand-new internal portal for front-desk agents, housekeeping leads and assistant managers. Engineering shipped it sprint-end. The role-based access control lives entirely in a cookie the browser is happy to edit.*

{{< /callout>}}

### Visit the Site & Log In

We visit the portal and sign in with our credentials. The portal loads and we can see an **Admin Console** option in the right pane.

{{< figure align="center" src="/images/Session_Swap_2.png" caption="*Logged In*" >}}


{{< figure align="center" src="/images/Session_Swap_3.png" caption="*Admin Console Visible*" >}}

### Attempt to Access the Admin Panel

Clicking on the Admin Console redirects us to {{< badge red >}}/admin{{< /badge>}}, but we receive an **Access Denied** response.

{{< figure align="center" src="/images/Session_Swap_4.png" caption="*Access Denied on /admin*" >}}


### Inspect the Cookie

Opening DevTools (or in burp) and checking the cookies, we spot a {{< badge gray >}}role=user{{< /badge>}} parameter. The server is trusting this client-supplied value to decide what we can access.

{{< figure align="center" src="/images/Session_Swap_5.png" caption="*role=user Cookie*" >}}

### Modify the Cookie

Using a cookie editor extension (or DevTools directly), we change the value from {{< badge gray >}}role=user{{< /badge>}} to {{< badge red >}}role=admin{{< /badge>}} and reload the page.

{{< figure align="center" src="/images/Session_Swap_6.png" caption="*Modifying role=user to role=admin*" >}}


The server accepts our self-assigned role and grants us full admin access.

{{< figure align="center" src="/images/Session_Swap_7.png" caption="*Admin Access Granted (Flag)*" >}}

### Why This Works

The application performs access control checks based on a cookie value the user can freely edit. There is no server-side validation that the role claim is legitimate or was issued by the server.

### Remediation

| Issue | Fix |
| --- | --- |
| Role stored in a client-editable cookie | Store the role server-side, keyed to the session ID |
| No server-side authorization check | Validate permissions against a database on every privileged request |

{{< callout tip "The takeaway" >}}

Never trust the client to tell you what permissions the client has.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/session-swap

{{< /callout>}}

---

## #4: Redirect Run

{{< callout info >}}

**Category:** Web Security / Information Disclosure

**Vulnerability:** Sensitive data leaked in the body of an intermediate redirect response

{{< /callout>}}

{{< figure align="center" src="/images/Redirect_Run_1.png" caption="*Redirect Run Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}
*Quikpay generates short, shareable receipt URLs for merchants. Visiting one bounces you to a friendly "thanks for your purchase" landing page. The redirect that gets you there isn't as silent as it looks — there's something the engineering team forgot to strip out of the intermediate response.*

{{< /callout>}} 

### Visit the Site

We visit the Quikpay site and click on the short follow link provided.

{{< figure align="center" src="/images/Redirect_Run_2.png" caption="*Visiting the Webpage — Follow Link*" >}}

### Follow the Redirect

Clicking the link triggers a redirect and lands us on the {{< badge purple>}}/receipt{{< /badge>}} page. In the browser it looks like a normal receipt confirmation page.

{{< figure align="center" src="/images/Redirect_Run_3.png" caption="*Redirected to /receipt*" >}}



### Intercept the Intermediate Response

In Burp Suite we intercept the redirect request (the {{< badge purple >}}3xx{{< /badge>}} response before the browser follows through). Inspecting the **response body** of this intermediate request reveals an {{< badge red >}}internal_ref{{< /badge>}} field that was never stripped out - it contains our flag.

{{< figure align="center" src="/images/Redirect_Run_4.png" caption="*internal_ref field in the intermediate redirect response — Flag inside*" >}}

### Why This Works

Browsers automatically follow redirects, so users never see the intermediate response. The developers assumed this meant its body was safe to use as a scratch pad. However, anyone intercepting traffic with a proxy like Burp can read the full response before the redirect is followed.

### Remediation

| Issue | Fix |
| --- | --- |
| Sensitive data in redirect response body | Send redirect responses with an empty body |
| Internal references exposed to clients | Never pass internal identifiers through client-facing responses |

{{< callout tip "The takeaway" >}}

The body of a redirect response is still visible to the client — treat it the same as any other response.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/redirect-run

{{< /callout>}}

---

## #5: Vellichor Press

{{< callout info >}}

**Category:** Web Security / Information Disclosure

**Vulnerability:** Sensitive admin note left in HTML source code

{{< /callout>}}

{{< figure align="center" src="/images/Vellichor_Press_1.png" caption="*Vellichor Press Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

*Vellichor Press is a six-person literary magazine that publishes four issues a year. Their new site went up last Tuesday on a tight deadline — Elias, the only engineer they have on retainer, told the editors he'd "do one more pass after launch." That pass never happened. Now the masthead looks immaculate to a reader, while a draft admin note sits where nobody scrolls.*

{{< /callout>}} 


### Visit the Site

We visit the Vellichor Press site. The UI looks clean - a standard literary magazine homepage with nothing obviously out of place.

{{< figure align="center" src="/images/Vellichor_Press_2.png" caption="*Visiting the Webpage*" >}}


### Check the Page Source

The lab description hints at something hidden that a reader wouldn't scroll to. We press {{< badge gray>}}Ctrl+U{{< /badge>}} to view the raw HTML source of the page.

Buried in the source code we find an admin note in an HTML comment along with our flag.

{{< figure align="center" src="/images/Vellichor_Press_3.png" caption="*Admin note with Flag found in HTML source*" >}}



### Why This Works

HTML comments and hidden elements are invisible in a rendered browser view but fully readable in the page source. Developers sometimes leave notes, credentials, or debug output in comments assuming users won't look - but the source is one keystroke away.

### Remediation

| Issue | Fix |
| --- | --- |
| Admin notes left in HTML comments | Never put sensitive information in client-side source code |
| No pre-launch source code review | Review rendered HTML in staging before deploying |

{{< callout tip "The takeaway" >}}

`Ctrl+U` is one of the first things a security tester presses. Your source code is public.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/marginalia

{{< /callout>}}

---

## #6: Sundial Observatory

{{< callout info >}}

**Category:** Web Security / Information Disclosure

**Vulnerability:** Private URLs exposed via `robots.txt`

{{< /callout>}}

{{< figure align="center" src="/images/Sundial_Observatory_1.png" caption="*Sundial Observatory Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *The Sundial Observatory has met on the second Saturday of every month since 1987, at a converted ranger station above the Cascade Plateau. Pavel, a retired aerospace tech, runs the website out of his garage. He's allergic to leaks but believes — as a matter of principle — that asking search engines politely to stay away is the same thing as keeping a page private.*
> 

{{< /callout>}}

### Visit the Site

We visit the Sundial Observatory homepage. Nothing interesting on the surface.

{{< figure align="center" src="/images/Sundial_Observatory_2.png" caption="*Visiting the Webpage*" >}}

### Find robots.txt

Scrolling to the footer we spot a link to {{< badge purple>}}robots.txt{{< /badge>}} - or we can navigate directly to {{< badge purple>}}/robots.txt{{< /badge>}}, which is a standard location every web server exposes.

{{< figure align="center" src="/images/Sundial_Observatory_3.png" caption="*robots.txt link found in footer*" >}}

The {{< badge purple>}}robots.txt{{< /badge>}} file contains several {{< badge red>}}Disallow{{< /badge>}} entries - paths Pavel wanted search engines to ignore.

{{< figure align="center" src="/images/Sundial_Observatory_4.png" caption="*robots.txt — multiple Disallow entries*" >}}

### Visit the Restricted Path

One of the disallowed paths is {{< badge green>}}/members-only-2026{{< /badge>}}. We navigate to it directly in the browser.

The page loads without any authentication prompt and our flag is right there.

{{< figure align="center" src="/images/Sundial_Observatory_5.png" caption="*Flag at /members-only-2026*" >}}

### Why This Works

{{< badge purple>}}robots.txt{{< /badge>}} is a public file. Its {{< badge red>}}Disallow{{< /badge>}} entries tell search engine crawlers what *not* to index — but they are readable by anyone, and they act as a directory of interesting paths rather than a protection mechanism. A {{< badge red>}}Disallow{{< /badge>}} entry provides zero access control.

### Remediation

| Issue | Fix |
| --- | --- |
| Private pages listed in robots.txt | Protect sensitive pages with authentication, not robots.txt |
| No access control on /members-only-2026 | Require login before serving any member-only content |

{{< callout tip "The takeaway" >}}

The takeaway: `robots.txt` is a signpost, not a lock. If a page needs to be private, put authentication in front of it.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/occultation

{{< /callout>}}

---

## #7: Pebble & Pine

{{< callout info >}}

**Category:** Web Security / Information Disclosure

**Vulnerability:** Secret hardcoded in client-side JavaScript

{{< /callout>}}

{{< figure align="center" src="/images/Pebble_Pine_1.png" caption="*Pebble & Pine Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Pebble & Pine is run by Marit and her partner Sasha out of a studio at the back of a kiln barn in the lower Catskills. They sell four to seven mugs at a time, in seasonal "runs," and ship them wrapped in linen. Marit wrote the website herself, with the help of her brother who works in advertising and "knows just enough JavaScript to be dangerous." The analytics file is his.*
> 

{{< /callout>}}

### Visit the Site

We visit the Pebble & Pine storefront. It's a minimal ceramics shop.

{{< figure align="center" src="/images/Pebble_Pine_2.png" caption="*Visiting the Webpage*" >}}

### Check the Page Source

The lab description calls out a JavaScript analytics file written by someone who "knows just enough JavaScript to be dangerous." We press {{< badge gray>}}Ctrl+U{{< /badge>}} to view the source and look through the linked JS files.

{{< figure align="center" src="/images/Pebble_Pine_3.png" caption="*Page Source — Linked JavaScript files*" >}}


Inside the analytics script we find a hardcoded internal reference constant, exposed on the {{< badge red>}}window{{< /badge>}} object for a "campaign exporter" that never got cleaned up:

```jsx
const INTERNAL_REF = "WEBVERSE{XXXXXXXXXXXXXXXXXXXX}";

// expose for the campaign exporter
window.__pp = window.__pp || {};
window.__pp.ref = INTERNAL_REF;
```

{{< figure align="center" src="/images/Pebble_Pine_4.png" caption="*Flag hardcoded in the analytics JavaScript file*" >}}

### Grab it from the Console

Since it's also attached to {{< badge red>}}window.__pp{{< /badge>}}, we can retrieve it directly from the browser console:

```jsx
window.__pp
```

{{< figure align="center" src="/images/Pebble_Pine_5.png" caption="*Flag retrieved via browser console*" >}}


### Why This Works

JavaScript shipped to the browser is fully readable by any visitor. Hardcoding secrets, tokens, or internal references in client-side JS -- even in minified files - exposes them to anyone who knows where to look.

### Remediation

| Issue | Fix |
| --- | --- |
| Secret hardcoded in client-side JS | Never put secrets in frontend code; serve them only from authenticated server-side endpoints |
| Internal reference exposed on window | Audit all JS files for sensitive constants before deploying |

{{< callout tip "The takeaway" >}}

> very byte of JavaScript you ship is public source code.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/glaze-recipe

{{< /callout>}}

---

## #8: Loop & Roam Records

{{< callout info >}}

**Category:** Web Security / Information Disclosure

**Vulnerability:** `.git` directory exposed on public web root; deleted secrets recoverable from git history

{{< /callout>}}

{{< figure align="center" src="/images/Loop_Roam_Records_1.png" caption="*Loop & Roam Records Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Loop & Roam was founded in a Detroit garage in 2011 by a bassist named Jovan who got tired of touring and wanted to put out his friends' records. The site is maintained by Aphra, a contract designer who was learning git when she built it. One night in February 2024 she committed a `.env` with prod credentials, noticed an hour later, ran `git rm .env`, and pushed. The deploy script `rsync`s the working tree — including the `.git/` directory — to the public web root.*
> 

{{< /callout>}}

### Visit the Site

We visit the Loop & Roam Records website.

{{< figure align="center" src="/images/Loop_Roam_Records_2.png" caption="*Visiting the Webpage*" >}}

### Spot the Exposed .git Directory

The description mentions a {{< badge red>}}.git{{< /badge>}} directory being {{< badge purple>}}rsync'd{{< /badge>}} to the public web root. We confirm this by navigating to {{< badge green>}}/.git/{{< /badge>}} -- it's accessible and directory listing is enabled.

{{< figure align="center" src="/images/Loop_Roam_Records_3.png" caption="*.git directory exposed on the public web root*" >}}

### Dump the Repository

We use {{< badge gray>}}git-dumper{{< /badge>}} to clone the exposed git repository to our local machine:

```bash
$ git-dumper https://d3b2f56a-4575-b-side-c08c5.challenges.webverselabs-pro.com ./web
```

{{< figure align="center" src="/images/Loop_Roam_Records_4.png" caption="*git-dumper downloading the repository*" >}}

{{< figure align="center" src="/images/Loop_Roam_Records_5.png" caption="*Repository successfully cloned locally*" >}}

### Find the Deleted .env in Git History

We check the commit log for anything suspicious:

```bash
$ git log 
```

We find a commit that removed the {{< badge red>}}.env{{< /badge>}} file.

{{< figure align="center" src="/images/Loop_Roam_Records_6.png" caption="*Git log showing the commit that deleted .env*" >}}

### Recover the Flag

We checkout that specific commit to to see the deleted {{< badge red>}}.env{{< /badge>}} file data:

```bash
$ git show cea0f3cf0164e18ad425b57bb7e1a70e9869e1ea
```

The {{< badge red>}}.env{{< /badge>}} file deleted data reappears with our flag inside.

{{< figure align="center" src="/images/Loop_Roam_Records_7.png" caption="*Flag recovered from the deleted .env in git history*" >}}

### Why This Works

{{< badge red>}}git rm{{< /badge>}} removes a file from the working tree and stages the deletion -- but the file's content is permanently stored in the commit where it was first added. Git history is immutable by default. Anyone with access to the {{< badge red>}}.git{{< /badge>}} directory can recover every file that was ever committed, including "deleted" ones.

### Remediation

| Issue | Fix |
| --- | --- |
| {{< badge red>}}.git{{< /badge>}} directory on public web root | Block access to {{< badge red>}}.git/{{< /badge>}} at the web server or reverse proxy level |
| Secret committed to git history | Rotate all credentials that were ever in the repository; use {{< badge green>}}git filter-repo{{< /badge>}} to scrub history |
| Secrets in {{< badge red>}}.env{{< /badge>}} committed at all | Use a secrets manager; add {{< badge red>}}.env{{< /badge>}} to {{< badge green>}}.gitignore{{< /badge>}} before the first commit |

{{< callout tip "The takeaway" >}}

> `git rm` does not delete history. If a secret ever touched a commit, assume it is compromised.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/b-side

{{< /callout>}}

---

## #9: Quikpay Receipts

{{< callout info >}}

**Category:** Web Security / Information Disclosure

**Vulnerability:** Debug endpoint exposed in production; verbose JSON response unlocked by `Content-Type` header change

{{< /callout>}}

{{< figure align="center" src="/images/Quikpay_Receipts_1.png" caption="*Quikpay Receipts Challenge*" >}}


### Lab Description

{{< callout insight "Briefing" >}}

> *Quikpay is a small payments backend used by a few dozen indie software shops. They take the integration seriously and the design seriously. They also have a debug branch in the resend handler that the engineering lead added during a late-night incident and never wrapped in a feature flag.*
> 

{{< /callout>}}

### Visit the Site

We visit the Quikpay dashboard and browse to the receipts section.

{{< figure align="center" src="/images/Quikpay_Receipts_2.png" caption="*Visiting the Webpage*" >}}

### View a Receipt

We open a receipt. Which can send a copy of receipt to the email on file.

{{< figure align="center" src="/images/Quikpay_Receipts_3.png" caption="*Viewing a Receipt*" >}}

### Find the Resend Endpoint in API Docs

The lab description hints at a debug branch in the **resend handler**. We visit the {{< badge gray>}}developers{{< /badge>}} page and find the receipt resend API documented there.

{{< figure align="center" src="/images/Quikpay_Receipts_4.png" caption="*Developers Page — Resend Receipt API*" >}}

### Intercept the Resend Request

We trigger a receipt resend and intercept it in Burp Suite. The request is sent as {{< badge purple>}}Content-Type: application/x-www-form-urlencoded{{< /badge>}}, but the response comes back as JSON a mismatch that suggests the server has a different code path for JSON requests.

{{< figure align="center" src="/images/Quikpay_Receipts_5.png" caption="*Request is form-encoded but response is JSON — a mismatch*" >}}

### Change the Content-Type and Resend

We send the request to Repeater ({{< badge gray>}}Ctrl+R{{< /badge>}}), then change the header:

```
> Content-Type: application/x-www-form-urlencoded  →  Content-Type: application/json
```

Sending with {{< badge gray>}}Ctrl+Shift+R{{< /badge>}} returns the full verbose JSON debug response -- and our flag is in it.

{{< figure align="center" src="/images/Quikpay_Receipts_6.png" caption="*Full JSON debug response with the Flag*" >}}

### Why This Works

The server has a debug code path that is triggered when the request {{< badge purple>}}Content-Type{{< /badge>}} is {{< badge green>}}application/json{{< /badge>}}. The debug branch was never wrapped in a feature flag and was deployed to production, where it returns verbose internal data to anyone who sends the right header.

### Remediation

| Issue | Fix |
| --- | --- |
| Debug code path active in production | Wrap debug handlers in feature flags; disable in production builds |
| Verbose response triggered by Content-Type | Validate and normalize request formats server-side; don't branch on client-supplied headers |

{{< callout tip "The takeaway" >}}

> Debug code paths left in production are vulnerabilities. Feature-flag everything you're not ready to ship.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/double-entry

{{< /callout>}}

---

## #10: Brackish Brewing Co.

{{< callout info >}}

**Category:** Web Security / Access Control

**Vulnerability:** IP-based access restriction bypassable via `X-Forwarded-For` header

{{< /callout>}}

{{< figure align="center" src="/images/Brackish_Brewing_Co_1.png" caption="*Brackish Brewing Co. Challenge*" >}}


### Lab Description

{{< callout insight "Briefing" >}}

> *Brackish Brewing has been in Coalridge since 2017 — a fifteen-barrel system, four year-round beers, a taproom that's walk-in only on weekends. The website's a Flask app the head brewer's partner wrote over a few rainy weekends; it has a small staff section where the floor manager posts the week's shifts and keg-pickup notes. The hosting setup got rearranged when they moved off their old reverse proxy in early 2025, and nobody on the brewing side thought to revisit the assumptions the staff section had been quietly making about where its traffic comes from.*
> 

{{< /callout>}}

### Visit the Site

We visit the Brackish Brewing Co. website. It's a standard brewery homepage.

{{< figure align="center" src="/images/Brackish_Brewing_Co_2.png" caption="*Visiting the Webpage*" >}}

### Find the Staff Section

The description mentions a small staff section. We navigate to {{< badge purple>}}/staff{{< /badge>}} and receive a {{< badge red>}}403 Forbidden{{< /badge>}} — the page is restricted to internal IP addresses only.

{{< figure align="center" src="/images/Brackish_Brewing_Co_3.png" caption="*403 Forbidden — /staff is restricted to internal IPs*" >}}

### Intercept the Request in Burp

We send the {{< badge purple>}}/staff{{< /badge>}} request to Burp and inspect it. The server is checking the client's IP to decide whether to allow access.

{{< figure align="center" src="/images/Brackish_Brewing_Co_4.png" caption="*Intercepted /staff request in Burp*" >}}

### Add X-Forwarded-For Header

We add the header {{< badge red>}}X-Forwarded-For: 127.0.0.1{{< /badge>}} to spoof our IP as localhost (an internal address) and forward the request.

The server now returns {{< badge green>}}200 OK{{< /badge>}} — we've bypassed the IP restriction.

```
X-Forwarded-For: 127.0.0.1
```

{{< figure align="center" src="/images/Brackish_Brewing_Co_5.png" caption="*200 OK — IP restriction bypassed*" >}}

### View in Browser

To render the page properly, in Burp's response panel we right-click → **Show response in browser → In original session**, copy the URL, and paste it into the browser.

{{< figure align="center" src="/images/Brackish_Brewing_Co_6.png" caption="*Staff page rendered in browser*" >}}

{{< figure align="center" src="/images/Brackish_Brewing_Co_7.png" caption="*Flag*" >}}

### Why This Works

The Flask app trusts the {{< badge purple>}}X-Forwarded-For{{< /badge>}} header to determine the client's real IP. This header is set by proxies to pass along the originating IP -- but it is entirely user-controlled when there is no trusted reverse proxy enforcing it. After the hosting migration removed the old reverse proxy, the application was left reading an unvalidated header directly from the client.

### Remediation

| Issue | Fix |
| --- | --- |
| IP access control based on X-Forwarded-For | Only trust X-Forwarded-For when set by a known, trusted reverse proxy |
| No authentication on /staff | Replace IP-based access control with proper session authentication |

{{< callout tip "The takeaway" >}}

> `X-Forwarded-For` is a client-supplied header. Never use it for access control decisions unless it is set exclusively by a trusted proxy you control.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/01/walk-in

{{< /callout>}}
