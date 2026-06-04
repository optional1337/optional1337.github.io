---
title: "How to Pass the CPTS: A Complete Roadmap"
date: 2026-06-04
weight: 1
tags: ["CPTS"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "A practical, step-by-step roadmap for passing the HackTheBox CPTS covering prerequisites, the HTB path, lab practice, exam strategy, and report writing."
summary: "A practical, step-by-step roadmap for passing the HackTheBox CPTS covering prerequisites, the HTB path, lab practice, exam strategy, and report writing."
canonicalURL: "https://canonical.url/to/page"
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/cptscover.jpeg"
    alt: "CPTS CERT"
    caption: "CPTS Journey"
    relative: false
    hidden: false
---

## Introduction

On {{< badge blue >}}January 2, 2026 at 1:09 PM IST{{< /badge >}}, I got an email from {{< badge red >}}HackTheBox{{< /badge >}} confirming I had passed the CPTS exam. Two months of course content, a month of grinding labs, another month in the pro labs all of it came down to that one moment.

{{< figure align="center" src="/images/cptsemail.png" caption="Certified!" >}}

I came into this field with no professional IT background just a BCA degree and a genuine obsession with offensive security. Before CPTS, I had cleared the eJPT and CRTA, which gave me a foundation in pentesting and Active Directory. Here's the [verified cert](https://www.credly.com/badges/647b921f-961b-4662-8f6e-57c66e16d9e4/linked_in_profile).

This is the guide I wish existed when I started a clear, practical roadmap from zero to CPTS.

---

## Prerequisites & Who This Is For

The CPTS exam requires you to complete the **Penetration Tester path** on HackTheBox Academy first. It's 28 modules of reading-based content (no videos), and it's one of the most well-structured courses available in this space. That's your starting point.

Before diving in, you only need one thing:

{{< callout insight "Actionable Insight" >}}
Be comfortable with basic Linux and Windows navigation that's genuinely all you need before starting the path.
{{< /callout >}}

You don't need to know what {{< badge yellow >}}nmap{{< /badge >}} is. You don't need to have done a single CTF. The path builds everything from the ground up. Whether you're a complete beginner or someone already doing pentesting, there's a prep path here that fits where you are.

---

## The HTB Penetration Tester Path (28 Modules)

### What the Path Covers

Each module follows a consistent structure: concept first, then hands-on lab where you retrieve a flag. Theory and practice, back to back, every single module.

Topics covered include: {{< badge yellow >}}networking{{< /badge >}}, {{< badge yellow >}}reconnaissance{{< /badge >}}, {{< badge yellow >}}scanning{{< /badge >}}, {{< badge yellow >}}exploitation{{< /badge >}}, {{< badge yellow >}}privilege escalation{{< /badge >}}, {{< badge yellow >}}Active Directory attacks{{< /badge >}}, {{< badge yellow >}}credential exfiltration{{< /badge >}}, {{< badge yellow >}}post-exploitation{{< /badge >}}, {{< badge yellow >}}web attacks{{< /badge >}}, {{< badge yellow >}}API attacks{{< /badge >}}, and more. By the end, you'll have touched almost every major area of offensive security.

The final module {{< badge green >}}Attacking Enterprise Networks{{< /badge >}} (AEN) is where everything comes together. You're given an IP and asked to compromise an entire network from scratch. Do it completely blind. If you get genuinely stuck, take a hint, get unstuck, and keep going.

{{< callout insight "💡 How to get the most out of the path" >}}
Don't rush through the reading. Understand the attack, understand the concept, take proper notes. The notes you build here will be what you lean on during the exam.
{{< /callout >}}

### Realistic Time Estimates

| Availability | Expected Duration |
|---|---|
| Fully free, solid hours daily | {{< badge purple >}}2–3 months{{< /badge >}} |
| Balancing work / college | {{< badge purple >}}6–8 months{{< /badge >}} |

*Don't rush it. The course rewards patience.*

---

## Note-Taking: Build Your Field Manual

The single most impactful habit you can build during prep is a {{< badge yellow >}}Field Manual{{< /badge >}}.

A Field Manual is a hierarchically structured set of notes where everything has a place. When you're mid-exploit and need the exact syntax for a service enumeration technique, you don't Google it you open your Field Manual and find it in seconds. Build it in whatever tool works for you: {{< badge green >}}Notion{{< /badge >}}, {{< badge green >}}Obsidian{{< /badge >}}, {{< badge green >}}OneNote{{< /badge >}}.

{{< figure align="center" src="/images/cheatsheat.png" caption="Field Manual" >}}

Here's an example structure:

```
- Information Gathering
    - Discovery
        - Nmap
    - Service Enumeration
        - FTP (21)
        - SSH (22)
        - DNS (53)
    - Web Enumeration
        - Passive
            - Infrastructure Identification
            - Subdomain Enumeration
            - Google Dorking
        - Active
            - Web Server Enumeration
            - Subdomain & Vhosts Fuzzing
            - Directory & Page Fuzzing
            - Parameter & Value Fuzzing
    - Application Enumeration
        - WordPress / Joomla / Drupal / Tomcat / Jenkins
    - Active Directory Enumeration
        - Host, User, Password Policy, SMB, ACL Enumeration
        - Trust Relationships
```

{{< callout insight "💡 Want to build your own Field Manual?" >}}
This video walks through exactly how to set one up from scratch.
{{< /callout >}}

{{< youtube 7LU6m_CF3cQ >}}

---

## Hands-On Lab Practice

### HTB Tracks (Recommended)

Once you've finished the course content, focus on building a solid methodology through the dedicated HTB tracks. Spend roughly a month here before considering the exam.

Complete both:

- [CPTS Track](https://app.hackthebox.com/tracks/76)
- [AD Track](https://app.hackthebox.com/tracks/60)

Between the two, you'll cover most of what the exam tests.

{{< image-row "/images/cptstrack.png" "CPTS Track" "/images/cptsad.png" "AD Track" >}}

### Pro Labs (Optional)

{{< callout insight "💡 Want to go further?" >}}
HTB Pro Labs are available but go beyond the CPTS scope. Treat them as a bonus, not a requirement.
{{< /callout >}}

---

## When Are You Ready to Book the Exam?

You'll never feel 100% ready and that's fine. If you've finished the course content, completed both tracks, and built your Field Manual, you're prepared. At some point you just have to go for it.

HackTheBox gives you **two exam attempts**. If you don't pass the first time, HTB provides detailed feedback pointing you to exactly what needs revisiting.

{{< callout warning "⚠️ Second attempt requires a report" >}}
To unlock your second attempt, you must submit a report even if you didn't pass, and even if the report is blank. Don't skip this or you'll lose your retry.
{{< /callout >}}

---

## Exam Preparation Strategy

There's no secret formula just habits that, if consistent, will put you in a strong position by exam day:

- **Go slow and understand every attack.** Ask yourself: how does this work? Why does it work? What breaks it? That depth is exactly what the exam tests.

- **Build your Field Manual as you go.** Every module, every lab if you learn a technique you know you'll forget, write it down immediately. Your Field Manual is your exam companion.

- **Set a fixed exam date from the start.** Treat it like a hard deadline. Without one, it's easy to drift across months of prep.

- **Revisit weak areas before the exam.** Once you've finished the course and labs, go back only to the topics where you felt shaky. Targeted review beats going through everything again.

- **Pace based on what you already know.** Move fast through familiar topics, slow down on new territory. There's no prize for finishing the course quickly.

---

## Exam Execution: What You Need to Know

The CPTS exam is a **10-day practical** where you compromise an entire network, capture `12/14 flags`, and submit a commercial-grade penetration testing report.

### General Tips

- **Stay calm.** You've done the work. You don't need all 10 days stay hydrated, sleep properly, trust your prep.
- **Enumerate constantly.** Multiple machines are chained together. Getting to the next machine always starts with thorough enumeration. When in doubt, enumerate more.
- **If you're stuck, step back.** You're probably in a rabbit hole. Take a break, come back with fresh eyes. Nine times out of ten the answer is: enumerate again.
- **The 1st and 8th flags are the tricky ones.** Don't let them shake you stay with the flow and enumerate harder.
- **Sleep seriously.** It's 10 days. Rest your brain a rested mind finds what an exhausted one keeps missing.

### Pivoting with Ligolo-ng

The exam requires double pivoting chaining through multiple hosts to reach deeper targets. Chisel + proxychains over SOCKS works, but it's complex and painfully slow for nmap scans.

**Ligolo-ng solves both problems:** it creates a proper routed network interface on your attack machine so every tool works natively at full speed no proxy overhead, no compatibility issues. Learn it before the exam, not during.

{{< callout info "🛠️ Full setup guide" >}}
A dedicated post covers setting up Ligolo-ng end-to-end from installing proxy and agent to adding routes and handling multiple subnets: https://optional1337.github.io/posts/ligolo-ng/
{{< /callout >}}

### Active Directory: BloodHound + PowerView

When you hit Active Directory and BloodHound isn't surfacing anything useful don't trust BloodHound alone. Cross-reference with PowerView. It often surfaces relationships and permissions that BloodHound misses entirely.

Manually enumerate the node you have. Sometimes that's the key.

```powershell
# Import PowerView and check ACLs for a specific user
Import-Module .\PowerView.ps1
$sid = Convert-NameToSid wley
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} -Verbose
```

{{< callout insight "⚠️ BloodHound vs PowerView" >}}
BloodHound is great for the big picture. PowerView is better for digging into specific nodes. Use both never rely on just one.
{{< /callout >}}

### Writing the Report (50% of Your Score)

The report is **50% of the exam**. You can capture all 14 flags and still fail if the report isn't up to standard.

{{< callout warning "⚠️ Don't leave the report until the end" >}}
Write as you go. Document each finding the moment you find it don't wait until you have 12 flags and then try to reconstruct everything from memory. The report takes serious time.
{{< /callout >}}

Use AI to help write and polish your findings. It's fast, it's allowed, and it makes a real difference in quality:

```text
Hey, I am writing my CPTS exam report help me make this attack chain clear and professional.

The tester found that the user Jimmy has AddMember rights over the HelpDesk group.
The tester added themselves to the group and leveraged the access to escalate privileges.
```

{{< callout info "🛠️ Use SysReptor for your report" >}}
It's fast, structured, and purpose-built for pentest reports. Highly recommended over Word or Google Docs.
{{< /callout >}}

{{< youtube ItVsQmHLicc >}}

---

## Conclusion

CPTS is not an easy certification. It will test your patience, your consistency, and your ability to think methodically under pressure. But it is absolutely achievable and the version of you that comes out the other side will have a level of practical skill that most certifications simply don't build.

Don't wait until you feel ready. Build your Field Manual, finish the course, grind the labs, set your exam date, and go. The preparation is the journey the exam is just the confirmation.

{{< callout insight "💬 One last thing" >}}
If you're a beginner or someone without a professional security background this cert is for you too. You don't need a perfect background. You just need to show up every day and keep going.
{{< /callout >}}

Good luck. You've got this.