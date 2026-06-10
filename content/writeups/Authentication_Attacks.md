---
title: "WebVerse: Junior Web Hacker (Authentication Attacks)"
date: 2026-06-10
weight: 4
tags: ["webverse","Authentication Attacks","Junior Web Hacker"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Authentication Attacks: Six web security challenge labs writeup covering default credentials, brute force, MFA bypass, and more."
summary: "Authentication Attacks: Six web security challenge labs writeup covering default credentials, brute force, MFA bypass, and more."
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/authentication_attacks_cover.svg"
    alt: "Authentication Attacks"
    caption: "WebVerse: Junior Web Hacker (Authentication Attacks)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Authentication Attacks:** Six web security challenge labs writeup covering default credentials, brute force, MFA bypass, and more.

**Platform:** webverse

{{< /callout>}}

---

## #1: Lake Forks Permits

{{< callout info >}}

**Category:** Web Security / Authentication

**Vulnerability:** Default credentials never rotated before production launch

{{< /callout>}}

{{< figure align="center" src="/images/Lake_Forks_Permits_1.png" caption="*Lake Forks Permits Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Lake Forks County's permits portal was redesigned in 2018 by a consultant who set the staff login credentials as a temporary default and noted "RESET BEFORE GOING LIVE" in the project binder on the clerk's shelf. The binder is in a different binder now.*

{{< /callout>}}

### Visit the Site

We start by visiting the webpage. It's a normal-looking county permits portal — nothing unusual on the surface.

{{< figure align="center" src="/images/Lake_Forks_Permits_2.png" caption="*Visiting the Webpage*" >}}

### Find the Staff Login

In the footer we find a **Staff Access** link. Clicking it redirects us to the {{< badge purple >}}/login{{< /badge>}} page.

{{< figure align="center" src="/images/Lake_Forks_Permits_3.png" caption="*Staff Access Link in Footer*" >}}

### Log In with Default Credentials

The lab description tells us the credentials were set as a temporary default and never changed. We try the most common default: {{< badge gray >}}admin:admin{{< /badge>}}.

{{< figure align="center" src="/images/Lake_Forks_Permits_4.png" caption="*Logging in with admin:admin*" >}}

It works — we're in and the flag is on the dashboard.

{{< figure align="center" src="/images/Lake_Forks_Permits_5.png" caption="*Flag*" >}}

### Why This Works

Default credentials are set during development as a convenience and must be rotated before going live. When that step is skipped, the application ships with a publicly known password. Anyone who recognizes the default has immediate admin access with no further exploitation required.

### Remediation

| Issue | Fix |
| --- | --- |
| Default credentials never rotated | Enforce a mandatory credential change as part of the deployment checklist |
| No post-launch credential audit | Periodically audit all accounts for default or weak passwords |

{{< callout tip "The takeaway" >}}

"RESET BEFORE GOING LIVE" is not a control. It is a wish. Automate credential rotation or block deployment until it's done.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/02/front-counter

{{< /callout>}}

---

## #2: Pinegrass Library Co-op

{{< callout info >}}

**Category:** Web Security / Authentication

**Vulnerability:** Username enumeration via verbose error messages + weak default password brute force

{{< /callout>}}

{{< figure align="center" src="/images/Pinegrass_Library_Co-op_1.png" caption="*Pinegrass Library Co-op Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Pinegrass has been a member-funded library co-op since 1962. The online portal was set up by Cyrus, the volunteer IT person, who picked a "temporary" password for every staff member's account so they could each log in once and change it. None of them did. The login form, separately, was written by a copy editor who made the error messages clearer than is good for them.*

{{< /callout>}}

### Visit the Site

We visit the site and navigate around normally. It's a library co-op portal — books, members, staff.

{{< figure align="center" src="/images/Pinegrass_Library_Co-op_2.png" caption="*Visiting the Webpage*" >}}

### Find Staff Names

We navigate to the {{< badge purple >}}/about{{< /badge>}} page and find a list of staff and volunteer names. We note **Marie Castan** as a target.

{{< figure align="center" src="/images/Pinegrass_Library_Co-op_3.png" caption="*/about Page — Staff Names*" >}}

### Test the Login Form

We go to {{< badge purple >}}/login{{< /badge>}} and try {{< badge gray >}}admin:admin{{< /badge>}}. The error message says: {{< badge red >}}admin is not a member{{< /badge>}} — the server is telling us the username doesn't exist. This is **username enumeration** via a verbose error message.

{{< figure align="center" src="/images/Pinegrass_Library_Co-op_4.png" caption="*Verbose error — username enumeration confirmed*" >}}

We also notice the member ID field shows a default placeholder hint ({{< badge gray >}}jsmith{{< /badge>}}). Based on the format and Marie Castan's name, we try {{< badge gray >}}mcastan{{< /badge>}} as the member ID. Using a wrong password now gives us a different error: {{< badge red >}}Password Incorrect for member mcastan{{< /badge>}} — confirming the username is valid.

{{< image-row "/images/Pinegrass_Library_Co-op_5.png" "Member ID format hint" "/images/Pinegrass_Library_Co-op_6.png" "Different error confirms mcastan is a valid member ID" >}}


### Brute Force the Password

The login page has no rate limiting, no CAPTCHA, and no lockout. We use {{< badge gray >}}ffuf{{< /badge>}} with a common password wordlist to brute force {{< badge gray >}}mcastan{{< /badge>}}'s password:

```bash
$ ffuf -u https://c22f4c3f-4575-roll-call-9b76c.challenges.webverselabs-pro.com/login -X POST -d "member_id=mcastan&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -w /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-1000.txt -mc all -fr "Password Incorrect for member mcstan."
```

We get a hit — the default password is {{< badge green >}}password1{{< /badge>}}.

{{< figure align="center" src="/images/Pinegrass_Library_Co-op_7.png" caption="*ffuf — password1 is the correct password*" >}}

### Log In and Get the Flag

We log in as {{< badge gray >}}mcastan:password1{{< /badge>}} and land on the member account page with the flag.

{{< figure align="center" src="/images/Pinegrass_Library_Co-op_8.png" caption="*Flag*" >}}

### Why This Works

Two separate issues combined to make this trivial. First, the login form returns different error messages for non-existent usernames versus wrong passwords — letting an attacker enumerate valid usernames. Second, the default password {{< badge red >}}password1{{< /badge>}} was never changed by the member, and no brute-force protection exists on the login endpoint.

### Remediation

| Issue | Fix |
| --- | --- |
| Verbose error messages reveal username existence | Return a single generic error: `Invalid member ID or password` |
| Default password never changed | Force a password change on first login; reject common passwords |
| No brute-force protection | Implement rate limiting, account lockout, or CAPTCHA on the login form |

{{< callout tip "The takeaway" >}}

Different error messages for wrong username vs wrong password are a free gift to attackers. Always return the same error regardless of which field is wrong.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/02/roll-call

{{< /callout>}}

---

## #3: Pivot HR

{{< callout info >}}

**Category:** Web Security / Authentication

**Vulnerability:** MFA gate implemented but not enforced on protected endpoints

{{< /callout>}}

{{< figure align="center" src="/images/Pivot_HR_1.png" caption="*Pivot HR Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Pivot HR is a six-person SaaS for indie small businesses — payroll, time-off, the occasional 1099. They wired up MFA in a hurry after a customer asked for SOC 2. The MFA gate works perfectly. It just doesn't gate anything past the front door.*

{{< /callout>}}

### Visit the Site & Log In

We visit the Pivot HR site and go to {{< badge purple >}}/login{{< /badge>}}. We sign in with our credentials.

{{< figure align="center" src="/images/Pivot_HR_2.png" caption="*Visiting /login*" >}}

{{< figure align="center" src="/images/Pivot_HR_3.png" caption="*Signed in — redirected to MFA*" >}}

### Hit the MFA Wall

After logging in we're redirected to {{< badge purple >}}/mfa{{< /badge>}}, which asks for a 6-digit code. Entering a random code like {{< badge gray >}}123456{{< /badge>}} gives us: {{< badge red >}}Code didn't match{{< /badge>}}.

{{< figure align="center" src="/images/Pivot_HR_4.png" caption="*MFA page — code didn't match*" >}}

### Inspect the Session Cookie

We intercept the login request in Burp and inspect the cookie that gets set. It contains:

```json
{"mfa_verified": false, "user_id": 4912}
```

Encoded as:

```
Cookie: session=eyJtZmFfdmVyaWZpZWQiOmZhbHNlLCJ1c2VyX2lkIjo0OTEyfQ.aikU-Q.9XDp-iFd6FPFTu_oUd6MLRMgXRI
```

The {{< badge red >}}mfa_verified{{< /badge>}} flag is {{< badge red >}}false{{< /badge>}} — the application knows we haven't completed MFA.

{{< figure align="center" src="/images/Pivot_HR_5.png" caption="*Session cookie showing mfa_verified: false*" >}}

### Skip Straight to the Dashboard

Instead of brute-forcing the 6-digit code, we check what other endpoints exist on the application.

{{< figure align="center" src="/images/Pivot_HR_6.png" caption="*Finding dashboard endpoint*" >}}

We navigate directly to {{< badge purple >}}/dashboard{{< /badge>}} with our current cookie — {{< badge red >}}mfa_verified: false{{< /badge>}} still set.

The dashboard loads. The server never checked the flag.

{{< figure align="center" src="/images/Pivot_HR_7.png" caption="*Dashboard accessible with mfa_verified: false — MFA bypassed*" >}}

### Why This Works

The MFA check only blocks access to the {{< badge purple >}}/mfa{{< /badge>}} page itself. Every other protected endpoint — including {{< badge purple >}}/dashboard{{< /badge>}} — does not verify the {{< badge red >}}mfa_verified{{< /badge>}} flag server-side before serving content. The MFA "gate" is only on the front door, not on any of the rooms behind it.

### Remediation

| Issue | Fix |
| --- | --- |
| MFA flag not checked on protected endpoints | Enforce `mfa_verified: true` on every authenticated route server-side |
| MFA state stored in a client-readable cookie | Store MFA state server-side in the session; never trust client-supplied verification flags |

{{< callout tip "The takeaway" >}}

Adding MFA to the login page is not the same as enforcing MFA. Every protected endpoint must independently verify the MFA state before serving a response.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/02/side-door

{{< /callout>}}

---

## #4: Heartwood Outfitters

{{< callout info >}}

**Category:** Web Security / Authentication

**Vulnerability:** Short numeric password reset code with no rate limiting — fully enumerable

{{< /callout>}}

{{< figure align="center" src="/images/Heartwood_Outfitters_1.png" caption="*Heartwood Outfitters Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Heartwood's site was built in a long weekend by a co-founder who reads more about fly-fishing than web security. The reset flow uses a short numeric code (because the previous one with hex tokens "confused a few customers"). The verify endpoint has no rate-limit, no captcha, and no lockout — so the 10000-code space is fully enumerable.*

{{< /callout>}}

### Visit the Site

We visit the Heartwood Outfitters site and navigate to the account section. The account button redirects us to {{< badge purple >}}/login{{< /badge>}}.

{{< figure align="center" src="/images/Heartwood_Outfitters_2.png" caption="*Visiting the Webpage*" >}}

### Explore the Forgot Password Flow

We click **Forgot Password** and land on {{< badge purple >}}/account/forgot{{< /badge>}}. We submit a test email — the response tells us a 4-digit reset code has been sent. There's a button to enter it which takes us to {{< badge purple >}}/account/reset{{< /badge>}}.

{{< image-row "/images/Heartwood_Outfitters_3.png" "Forgot Password page" "/images/Heartwood_Outfitters_4.png" "4-digit reset code sent" >}}

{{< image-row "/images/Heartwood_Outfitters_5.png" "Reset code entry page at /account/reset" "/images/Heartwood_Outfitters_6.png" "Invalid email or code — need a real email" >}}

### Find a Valid Target Email

Entering a random code with a made-up email gives Invalid email or code. We need a real email in the database. We navigate to the {{< badge purple >}}/about{{< /badge>}} page and find an admin email: {{< badge red >}}admin@heartwood-outfitters.example{{< /badge>}}. We trigger the forgot password flow again using this email.

{{< figure align="center" src="/images/Heartwood_Outfitters_7.png" caption="*/about page — admin email found*" >}}

### Generate the Code Wordlist

A 4-digit numeric code has only 10,000 possible values — trivially enumerable. We generate the full wordlist with {{< badge gray >}}crunch{{< /badge>}}:

```bash
$ crunch 4 4 0123456789 -t %%%% >> num.txt
```

{{< figure align="center" src="/images/Heartwood_Outfitters_8.png" caption="*crunch generating the 4-digit numeric wordlist*" >}}

### Brute Force the Reset Code

We use {{< badge gray >}}ffuf{{< /badge>}} to iterate through all 10,000 codes while simultaneously setting a new password for the admin account:

```bash
$ ffuf -u https://24c36299-4575-spare-key-eff59.challenges.webverselabs-pro.com/account/reset -X POST -d "email=admin%40heartwood-outfitters.example&code=FUZZ&password=optional123" -H "Content-Type: application/x-www-form-urlencoded" -w num.txt -mc all -fr "Invalid email or code."
```

We get a hit — the correct code is {{< badge green >}}1075{{< /badge>}}. Since reset codes are single-use, the request has already set the admin password to {{< badge green >}}optional123{{< /badge>}}.

{{< figure align="center" src="/images/Heartwood_Outfitters_9.png" caption="*ffuf — correct reset code 1075 found*" >}}

### Log In as Admin

We sign in with {{< badge gray >}}admin@heartwood-outfitters.example: optional123{{< /badge>}} and get the flag.

{{< figure align="center" src="/images/Heartwood_Outfitters_10.png" caption="*Flag*" >}}

### Why This Works

A 4-digit numeric code has only 10,000 possible values. Without rate limiting, lockout, or CAPTCHA on the {{< badge purple >}}/account/reset{{< /badge>}} endpoint, an attacker can iterate the entire space in seconds and take over any account whose email they know — which is often public.

### Remediation

| Issue | Fix |
| --- | --- |
| 4-digit reset code — only 10,000 combinations | Use a cryptographically random token of at least 128 bits |
| No rate limiting on reset endpoint | Limit reset attempts per IP and per email; add exponential backoff |
| Admin email publicly visible on /about | Don't expose staff emails on public pages |

{{< callout tip "The takeaway" >}}

A password reset code is only as strong as its entropy times its rate limiting. Short codes with no lockout are not tokens — they are guesses waiting to be made.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/02/spare-key

{{< /callout>}}

---

## #5: Halftrack Model Railroad Club

{{< callout info >}}

**Category:** Web Security / Authentication

**Vulnerability:** Predictable username format + weak password with no rate limiting

{{< /callout>}}

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_1.png" caption="*Halftrack Model Railroad Club Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Halftrack has met in the basement of the VFW on the second Tuesday of every month since 1971. Hollis Kerrigan has been president since 1994. He picked the member-portal admin password himself when the website went up in 1998 and has refused to change it, on principle. There is no rate limit on the login form.*

{{< /callout>}}

### Visit the Site

We visit the Halftrack website and look around.

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_2.png" caption="*Visiting the Webpage*" >}}

### Find the Target Username

On the {{< badge purple >}}/about{{< /badge>}} page we find the name **Hollis Kerrigan** — president since 1994 and the highest-value account on the portal.

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_3.png" caption="*/about Page — Hollis Kerrigan*" >}}

### Confirm the Username Format

The member portal at {{< badge purple >}}/login{{< /badge>}} shows the username format: **first initial + last name, lowercase, no spaces or dots**. From the FAQ on {{< badge purple >}}/about{{< /badge>}} we also confirm this is the correct format.

Hollis Kerrigan → {{< badge gray >}}hkerrigan{{< /badge>}}

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_4.png" caption="*Login page — username format shown*" >}}

We try {{< badge gray >}}hkerrigan{{< /badge>}} with a random password and get: {{< badge red >}}Incorrect username or password{{< /badge>}}. The format is confirmed by the FAQ on the {{< badge purple >}}/about{{< /badge>}} page.

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_5.png" caption="*Username accepted, password wrong*" >}}

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_6.png" caption="*Username format confirmed in FAQ*" >}}

### Brute Force the Password

No rate limiting on the login form. We run {{< badge gray >}}ffuf{{< /badge>}} against it with a common password list:

```bash
$ ffuf -u https://f505d2a4-4575-combination-4a36d.challenges.webverselabs-pro.com/login -X POST -d "username=hkerrigan&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -w /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-1000.txt -mc all -fr "Incorrect username or password."
```

We get a hit — the password is {{< badge green >}}password1{{< /badge>}}.

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_7.png" caption="*ffuf — password1 is the correct password*" >}}

### Log In and Get the Flag

We log in as {{< badge gray >}}hkerrigan: password1{{< /badge>}} and get access to the member portal and the flag.

{{< figure align="center" src="/images/Halftrack_Model_Railroad_Club_8.png" caption="*Flag*" >}}

### Why This Works

The username format is publicly documented and deterministic. The password hasn't been changed since 1998 and is in every common wordlist. With no rate limiting on the login form, an attacker can test thousands of passwords per minute with zero friction.

### Remediation

| Issue | Fix |
| --- | --- |
| Weak password unchanged since 1998 | Enforce a minimum password complexity policy; prompt long-standing accounts to update |
| No rate limiting on login | Implement rate limiting, lockout after N failed attempts, or CAPTCHA |
| Predictable username format | Usernames don't need to be secret, but weak passwords combined with known usernames are a free credential pair |

{{< callout tip "The takeaway" >}}

A password that was weak in 1998 is still weak in 2026. Mandatory password rotation policies and rate limiting on login forms are not optional.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/02/combination
{{< /callout>}}

---

## #6: Skein

{{< callout info >}}

**Category:** Web Security / Authentication

**Vulnerability:** Persistent "remember me" cookie with a forgeable structure (username:md5(password) base64-encoded) — authentication enforced via cookie alone

{{< /callout>}}

{{< figure align="center" src="/images/Skein_1.png" caption="*Skein Challenge*" >}}

### Lab Description

{{< callout insight "Briefing" >}}

> *Skein is a volunteer-moderated forum for spinners, knitters and quilt-makers. Members hate logging in every time they swap a colourway, so the "Keep me signed in for 30 days" checkbox is on by default. The head moderator, who manages a small army of fibre-fest organisers, picked a memorable password back when the forum was just three friends and a Google sheet.*

{{< /callout>}}

### Visit the Site

We visit the Skein forum and navigate around. We check different pages to understand the layout and functionality.

{{< figure align="center" src="/images/Skein_2.png" caption="*Visiting the Webpage*" >}}

### Find a Target Username

On the {{< badge purple >}}/members{{< /badge>}} page we find a list of usernames. We note **loomweaver** as a high-value target — confirmed to be the head moderator Marin Loomweaver.

{{< figure align="center" src="/images/Skein_3.png" caption="*/members Page — loomweaver found*" >}}

### Create an Account and Inspect Cookies

We register an account with the **Keep me signed in for 30 days** checkbox enabled (it's on by default). After logging in we intercept the request in Burp and examine the cookies.

{{< image-row "/images/Skein_4.png" "Creating an account" "/images/Skein_5.png" "Logged in as optional" >}}

{{< figure align="center" src="/images/Skein_6.png" caption="*Inspecting the cookie*" >}}

We have two cookies:

```
rmbr=b3B0aW9uYWw6ZDU3YzI0ZjNmZTUyZDE2ZTcxNjliOTEyZGQ2NDdmMGQ=;
sk_session=88eb6a0973c7f640f4fb28207bb31612
```

{{< badge red >}}sk_session{{< /badge>}} is the session token and {{< badge red >}}rmbr{{< /badge>}} is the remember-me cookie. Decoding the {{< badge red >}}rmbr{{< /badge>}} value from Base64 reveals the structure:

```
username:md5(password)
```

We confirm this by checking our own password's MD5 hash — it matches exactly. The {{< badge red >}}rmbr{{< /badge>}} cookie is entirely forgeable if we know a target's username and password hash.

{{< figure align="center" src="/images/Skein_7.png" caption="*rmbr cookie decoded — username:md5(password) in Base64*" >}}

### Determine Which Cookie Authenticates

We send the {{< badge purple >}}/account{{< /badge>}} request to Burp Repeater ({{< badge gray >}}Ctrl+R{{< /badge>}}) and remove the {{< badge red >}}sk_session{{< /badge>}} cookie entirely, keeping only {{< badge red >}}rmbr{{< /badge>}}. We're still logged in — the {{< badge red >}}rmbr{{< /badge>}} cookie alone is sufficient for authentication.

{{< figure align="center" src="/images/Skein_8.png" caption="*sk_session removed — still authenticated via rmbr alone*" >}}

---

### Method 1 — Forge the Cookie via Brute Force

We don't know {{< badge gray >}}loomweaver{{< /badge>}}'s password, but we can build a wordlist of pre-computed cookie values and fuzz for the correct one.

**Build the cookie wordlist:**

```bash
$ while read p; do
    hash=$(echo -n "$p" | md5sum | cut -d' ' -f1)
    echo -n "loomweaver:$hash" | base64
done < /usr/share/seclists/Passwords/Common-Credentials/darkweb2017_top-1000.txt > cookies.txt
```

{{< figure align="center" src="/images/Skein_9.png" caption="*Cookie wordlist generating*" >}}

{{< figure align="center" src="/images/Skein_10.png" caption="*Cookie wordlist — loomweaver:md5(password) base64-encoded for each candidate*" >}}

**Fuzz for the correct cookie:**

```bash
$ ffuf -u https://691677df-4575-bump-key-3b1a8.challenges.webverselabs-pro.com/account -H "Cookie: rmbr=FUZZ" -w cookies.txt -fc 302
```

We get a hit — a {{< badge green >}}200{{< /badge>}} instead of a {{< badge red >}}302{{< /badge>}} redirect means we're authenticated.

{{< figure align="center" src="/images/Skein_11.png" caption="*ffuf — correct rmbr cookie value hit*" >}}

We open DevTools or a cookie editor, replace the {{< badge red >}}rmbr{{< /badge>}} cookie with the found value, and visit {{< badge purple >}}/account{{< /badge>}} — we're now logged in as Marin Loomweaver.

{{< figure align="center" src="/images/Skein_12.png" caption="*Logged in as Marin Loomweaver via forged rmbr cookie*" >}}

---

### Method 2 — Direct Password Brute Force

The footer shows the community email {{< badge gray >}}post@skein.community{{< /badge>}}. From this we can guess the moderator's email format: {{< badge gray >}}loomweaver@skein.community{{< /badge>}}.

{{< figure align="center" src="/images/Skein_13.png" caption="*Footer email — guessing loomweaver@skein.community*" >}}

We brute force the login directly:

```bash
$ ffuf -u https://691677df-4575-bump-key-3b1a8.challenges.webverselabs-pro.com/login -X POST -d "email=loomweaver%40skein.community&password=FUZZ&keep_signed_in=on" -H "Content-Type: application/x-www-form-urlencoded" -w /usr/share/seclists/Passwords/Common-Credentials/darkweb2017_top-1000.txt -mc all -fr "We couldn't find an account with that email and password." -fs 6892
```

We get a hit — the password is {{< badge green >}}password123{{< /badge>}}.

{{< figure align="center" src="/images/Skein_14.png" caption="*ffuf — password123 is the correct password*" >}}

We log in with {{< badge gray >}}loomweaver@skein.community: password123{{< /badge>}} and get the flag.

{{< figure align="center" src="/images/Skein_15.png" caption="*Flag*" >}}

### Why This Works

The {{< badge red >}}rmbr{{< /badge>}} cookie encodes {{< badge red >}}username:md5(password){{< /badge>}} in Base64 — a structure that is trivially reversible and forgeable. MD5 is not a password hashing function; it is a checksum. It has no salt and can be cracked or iterated in milliseconds. Combined with a weak password and no brute-force protection on either the login form or the cookie endpoint, an attacker only needs to know the username.

### Remediation

| Issue | Fix |
| --- | --- |
| Remember-me cookie encodes password hash | Use an opaque, server-side session token with no derivable structure |
| MD5 used for password hashing | Use a proper password hashing algorithm: bcrypt, Argon2, or scrypt |
| No rate limiting on login or account endpoint | Implement rate limiting and lockout on all authentication endpoints |
| Weak password on a high-privilege account | Enforce password complexity; audit privileged accounts for common passwords |

{{< callout tip "The takeaway" >}}

A remember-me cookie should be an opaque random token that maps to a server-side session. If you can reverse-engineer the cookie structure from the value, it is not a token — it is a window into your authentication logic.

Do this lab on webverse Now: https://dashboard.webverselabs-pro.com/learning-paths/junior-web-hacker/02/bump-key

{{< /callout>}}