---
title: "WebVerse: MooKoo"
date: 2026-06-14
weight: 4
tags: ["webverse","SSTI","Ruby","RCE"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Ruby ERB Server-Side Template Injection (SSTI) leading to RCE - Challenge Lab Writeup."
summary: "Ruby ERB Server-Side Template Injection (SSTI) leading to RCE - Challenge Lab Writeup."
# canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/mookoo_cover.svg"
    alt: "MooKoo challenge"
    caption: "MooKoo challenge (SSTI)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **MooKoo**

**Category:** Web Security / Server-Side Template Injection

**Vulnerability:** Ruby **ERB Template Injection** leading to **Remote Code Execution**

**Platform:** webverse

{{< /callout>}}

{{< figure align="center" src="/images/MooKoo_1.png" caption="*MooKoo Challenge*" >}}

## Lab Description

{{< callout insight "Briefing" >}}
*The Hartwells built MooKoo a website to rehome their animals before the gates close for good. It was thrown together quickly, the way a small farm shop tends to be — and it shows once you start poking at the corners.*
{{< /callout>}}

---

## Visiting the Site

The MooKoo Family Farm site greets us with a closing-down banner and a "Meet the herd" call to action. Standard small-business storefront — nothing screams "vulnerable" yet.

{{< figure align="center" src="/images/MooKoo_2.png" caption="MooKoo Family Farm - Landing Page" >}}

Clicking into one of the animals takes us to a product page for **Clover** the goat. The interesting part is in the address bar - the page is rendered based on a {{< badge blue >}}product{{< /badge >}} query parameter.

{{< figure align="center" src="/images/MooKoo_3.png" caption="Product page for Clover - note the product query parameter" >}}

## Probing the `product` Parameter

Since the page content clearly depends on {{< badge blue >}}product{{< /badge >}}, the obvious first move is to throw in a value that doesn't exist and see how the app responds.

```
/product?product=test
```

The server happily echoes our input back inside its "not found" message:

{{< figure align="center" src="/images/MooKoo_4.png" caption="Unrecognised product value reflected back in the response" >}}

That reflection is the first real signal - whatever we put in {{< badge blue >}}product{{< /badge >}} ends up somewhere in the rendered page.


## HTML Injection

To check whether that reflection is encoded, we try a simple HTML tag:

```
/product?product=<h1>test</h1>
```

The {{< badge red >}}&lt;h1&gt;{{< /badge >}} tag is **not** sanitised - "test" comes back rendered as an actual heading instead of literal text.

{{< figure align="center" src="/images/MooKoo_5.png" caption="Injected <h1> tag is rendered, confirming unescaped output" >}}

So the value isn't HTML-encoded before being placed into the page. The natural next step is to check how far that goes.


## A Promising Lead... That Goes Nowhere

If raw HTML is rendered, raw JavaScript might run too. A classic {{< badge gray >}}&lt;script&gt;alert()&lt;/script&gt;{{< /badge >}} confirms it:

```
/product?product=<script>alert()</script>
```

{{< figure align="center" src="/images/MooKoo_6.png" caption="Script tag executes - but only reflects back to us" >}}

The alert fires - but this is purely **reflected, self-XSS**. The payload only ever executes in the browser of whoever follows the crafted link, and there's no victim to send it to here. On its own this doesn't get us anywhere. Time to dig into what's actually rendering this page.


## Fingerprinting the Backend

Instead of HTML/JS payloads, we try something messier - a string that mixes syntax from several templating engines at once, hoping to break the parser and leak an error:

```
/product?product=$&#123;&#123;%3C%[%%27%22&#125;&#125;%\.
```

This trips the server up entirely, and it responds with a raw **400 Bad Request** straight from the underlying web server:

{{< figure align="center" src="/images/MooKoo_7.png" caption="Malformed request leaks the backend stack - WEBrick / Ruby" >}}

The error banner gives away the stack: {{< badge purple >}}WEBrick/1.8.1 (Ruby/3.2.11){{< /badge >}}. So we're dealing with a Ruby application served via WEBrick. That immediately narrows down which template engines are realistically in play - most likely **ERB**, Ruby's built-in templating system.

## A Hint in the Source

Before jumping straight to payloads, it's worth checking the site's static assets - sometimes developers leave breadcrumbs behind. A quick look at {{< badge blue >}}style.css{{< /badge >}} turns up exactly that:

{{< figure align="center" src="/images/MooKoo_8.png" caption="A developer comment in style.css hints at where SSTI output lands" >}}

The comment reads:

```css
/* notice / not-found (the SSTI output lands in .notice) */
```

That's about as direct a hint as it gets - the {{< badge blue >}}product{{< /badge >}} value isn't just reflected, it's being **evaluated** somewhere before landing in the {{< badge gray >}}.notice{{< /badge >}} element.

## Confirming Ruby SSTI

With a Ruby backend confirmed and a hint pointing straight at template evaluation, the natural test is a basic **ERB** injection:

```erb
<%= `id` %>
```

ERB's {{< badge purple >}}&lt;%= %&gt;{{< /badge >}} tags evaluate and print Ruby expressions, and backticks in Ruby run a shell command and return its output - a perfect one-shot test for both SSTI *and* command execution at once.

Since the payload contains characters the URL won't tolerate as-is, we URL-encode it first using **CyberChef**:

{{< figure align="center" src="/images/MooKoo_9.png" caption="URL-encoding the ERB payload with CyberChef" >}}

This gives us:

```
%3C%25=%20%60id%60%20%25%3E
```

Dropping that into the {{< badge blue >}}product{{< /badge >}} parameter:

```
/product?product=<%25= `id` %25>
```

The server evaluates it and prints the output of {{< badge green >}}id{{< /badge >}} straight into the page:

{{< figure align="center" src="/images/MooKoo_10.png" caption="ERB payload executes - command output reflected in the response" >}}

```
uid=1100(farmhand) gid=999(farm) groups=999(farm)
```

This is **Server-Side Template Injection**: the {{< badge blue >}}product{{< /badge >}} parameter is being concatenated directly into an ERB template and rendered, giving us arbitrary Ruby (and shell) execution as the {{< badge purple >}}farmhand{{< /badge >}} user.


## Reading the Flag

With confirmed command execution, grabbing the flag is just a matter of swapping the command:

```erb
<%= `cat /flag.txt` %>
```

URL-encoded and sent through the same {{< badge blue >}}product{{< /badge >}} parameter:

{{< figure align="center" src="/images/MooKoo_11.png" caption="Flag retrieved via SSTI" >}}

---

## Why This Works

The root cause is that the {{< badge blue >}}product{{< /badge >}} query parameter is concatenated **directly into an ERB template string**, which is then rendered server-side. ERB has no concept of "untrusted input" - anything inside {{< badge purple >}}&lt;% %&gt;{{< /badge >}} or {{< badge purple >}}&lt;%= %&gt;{{< /badge >}} is treated as Ruby code and executed with the privileges of the application process.

A properly implemented page should:

- Treat user-supplied values as **data**, never as template source.
- Look up the requested product from a fixed dataset (e.g. by ID/slug) and pass only the *result* into the template - never the raw input.
- Run the application with a minimally privileged, sandboxed user.


## Remediation

| Issue | Fix |
| --- | --- |
| User input concatenated into an ERB template string | Never build templates from request data - render static templates and pass user input only as variables/data |
| Unescaped HTML reflected in the response | HTML-encode all user-controlled output before rendering |
| Verbose server errors leak stack details (WEBrick/Ruby version) | Disable detailed error pages in production; return generic error responses |
| Application runs with shell access via backticks | Avoid shell invocation entirely for user-influenced data; if unavoidable, run under a tightly sandboxed, least-privilege account |


{{< callout tip "The takeaway">}}

Any time user input is unescaped *and* the response changes shape based on it, it's worth asking what's doing the rendering. A reflected value that lands inside a templating context - not just raw HTML - can turn a "harmless" reflection into full remote code execution. Fingerprinting the stack (even via an error message) is often the fastest route to the right payload family.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/challenges/mookoo

{{< /callout>}}