---
title: "From zero to CPTS: My journey!"
date: 2026-06-04
weight: 1
# aliases: ["/first"]
tags: ["CPTS"]
author: "Gourav."
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
# draft: false
hidemeta: false
# comments: false
description: "All the preparation in the world means nothing if exam day falls apart. I share practical tips on time management, handling tricky questions, and keeping your head calm - based on my own experience sitting in that chair."
summary: "All the preparation in the world means nothing if exam day falls apart. I share practical tips on time management, handling tricky questions, and keeping your head calm - based on my own experience sitting in that chair."
canonicalURL: "https://canonical.url/to/page"
# disableHLJS: true # to disable highlightjs
# disableShare: false
# disableHLJS: false
hideSummary: false
# searchHidden: true
ShowReadingTime: true
# ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
# ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/cptscover.jpeg" # image path/url
    alt: "CPTS CERT" # alt text
    caption: "CPTS Journey" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link

---

## Introduction

I have a habit of checking my email obsessively - first thing in the morning, between study sessions, right before bed. Most of the time it's nothing. 

But on {{< badge blue >}}January 2, 2026 at 1:09 PM IST{{< /badge >}}, it was everything.

A notification from {{< badge red >}}HackTheBox{{< /badge >}} sat at the top of my inbox. I already knew what it was about - I had submitted my CPTS exam just three days earlier, and the results had come back faster than I expected. My finger hovered over it for a second longer than I'd like to admit.

{{< figure align="center" src="/images/cptsemail.png" caption="Certified!" >}}

I passed. Two months of course content, a month of grinding labs, another month in the pro labs - all of it had led to this one quiet moment staring at my phone screen in the middle of the afternoon.

I'm a BCA student. Not a seasoned IT professional. Not someone who came in with years of networking experience. Just someone who decided to go all in on learning offensive security - and figured out a path that worked.

This blog series is everything I wish someone had written before I started. If you're thinking about CPTS and don't know where to begin, keep reading.

### Who I am & credibility

My name is Gourav, and I'm a BCA graduate who's been actively learning and working in cybersecurity for the past two years. I came into this field purely out of curiosity - no professional IT background, no prior work experience in security - just a genuine obsession with understanding how systems get broken and how to defend them.

Before sitting the CPTS, I had already cleared the eJPT and CRTA certifications, which gave me a solid foundation in penetration testing concepts and active directory attacks. So by the time I started the CPTS path, I wasn't starting from scratch - but I still had a lot to learn.

Here's the cert, in case you want to [verify](https://www.credly.com/badges/647b921f-961b-4662-8f6e-57c66e16d9e4/linked_in_profile):

{{< figure align="center" src="/images/cpts.jpg" caption="Certified!" >}}

I'm writing this series as someone who went through the entire process as a student - not as a professional with years of industry experience. If anything, I think that makes this more useful for a lot of you.


### What this blog series covers
This series is a complete walkthrough of how I cleared the CPTS - the preparation, the techniques, the problems I ran into, and what actually worked.

Whether you're a complete beginner who has never touched Linux, or someone already doing pentesting professionally, I've written this so both of you can follow along and build a prep path that fits where you are right now.

### How I got into cybersecurity
It started in 10th grade. I stumbled across the word **cybersecurity** and did what any curious teenager would do - Googled it. The first thing that came up was OSCP. I had no idea what it was, but I went down the rabbit hole anyway and found a YouTube video by [Shubham Gupta](https://www.linkedin.com/in/hackerspider1/) (aka {{< badge blue >}}hackerspider1{{< /badge >}})  someone sharing his entire OSCP journey. That video planted the seed. He was the first person who made me think this was something I could actually do.

Life moved on. 12th grade happened, exams happened, and cybersecurity got pushed to the back. But when I got into BCA, something clicked back into place. That's where I touched Linux for the first time - before that, I genuinely thought Windows was the only operating system that existed.

From there, I found TryHackMe, worked through the basics, and completed somewhere around 50-60 boxes. That eventually led me to HackTheBox - and the rest of the story is what this blog is about.  

## Background knowledge you need
Before anything else - the CPTS exam requires you to complete the Penetration Tester path on HackTheBox Academy. It's 28 modules of reading-based content (no videos), and it's genuinely one of the most comprehensive and well-structured courses I've come across. If you haven't started it yet, that's your starting point.

{{< callout tip >}}
💡 Already familiar with the path? Feel free to [skip](#understanding-the-course-content) ahead to the next section.
{{< /callout >}}


Now, what do you actually need to know before jumping in? Honestly not much. You don't need any prior pentesting experience. The path is designed to build that from the ground up.

{{< callout insight "Actionable Insight" >}}
Before starting the CPTS path, make sure you're comfortable with basic Linux and Windows navigation.
{{< /callout >}}

That's genuinely it. You don't need to know what {{< badge yellow >}}nmap{{< /badge >}} is. You don't need to have done a single CTF. By the time you finish the path, you'll have a level of practical, hands-on knowledge that most courses don't come close to giving you.


## Understanding the course content
The Penetration Tester path has 28 modules. Each module follows the same structure first it teaches you the concept, then it tests you through either assessment questions or a hands-on lab where you need to exploit something and retrieve a flag. Theory and practice, back to back, every single module.

The topics covered across the path are comprehensive. You'll go through {{< badge purple >}}networking{{< /badge >}}, {{< badge purple >}}reconnaissance{{< /badge >}}, {{< badge purple >}}scanning{{< /badge >}}, {{< badge purple >}}exploitation{{< /badge >}}, {{< badge purple >}}privilege escalation{{< /badge >}}, {{< badge purple >}}Active Directory attacks{{< /badge >}}, {{< badge purple >}}credential exfiltration{{< /badge >}}, {{< badge purple >}}post-exploitation{{< /badge >}}, {{< badge purple >}}web attacks{{< /badge >}}, {{< badge purple >}}API attacks{{< /badge >}}, and more. By the end, you'll have touched almost every major area of offensive security.

The final module {{< badge purple >}} Attacking Enterprise Networks {{< /badge >}} (AEN)  is where everything comes together. You're given an IP and asked to compromise an entire network from scratch using everything you've learned. Most people in the community will tell you to do it completely blind. I agree - but with one caveat: if you get genuinely stuck, take a hint, get unstuck, and keep going. Don't let your ego turn a learning exercise into a week of staring at a screen.

{{< callout insight "💡 How to get the most out of the path" >}}

Don't rush through the reading. Understand the attack, understand the concept, and take proper notes. Every lab will teach you something - write it down. The notes you build here will be what you lean on during the exam.

{{< /callout >}}

As for how long it takes if you're fully free and can dedicate solid hours daily, expect around {{< badge purple >}}2–3 months {{< /badge >}}. If you're balancing work, college, or other commitments, a more realistic timeline is {{< badge purple >}} 6–8 months {{< /badge>}}. Either way, don't rush it. The course rewards patience.

## Note-taking strategy during preparation
My biggest recommendation for CPTS prep build a {{< badge yellow>}}Field Manual{{< /badge>}}. It will save you more time than any other habit you pick up during this journey.

A Field Manual is a hierarchically structured set of notes where everything has a place. When you're mid-exploit and need to remember the exact flag syntax for a service enumeration technique, you don't Google it you open your Field Manual and find it in seconds. You can build it in any tool you're comfortable with {{< badge green>}}Notion{{< /badge>}}, {{< badge green>}}Obsidian{{< /badge>}}, {{< badge green>}}OneNote{{< /badge>}}, whatever works for you.

Here's an example of how mine was structured:

{{< figure align="center" src="/images/cheatsheat.png" caption="Field Manual" >}}

`Ex:` 
- Information Gathering
    - Discovery
        - Nmap
    - Serivce Enmeration
        - FTP (21)
        - SSH (22)
        - DNS (53) 
    - Web Enumeration
        - Reconnaissance
        - Passive
            - Infrastructure Identification
            - Subdomain Enumeration
            - Google Dorking
        - Active
            - Web Server Enumeration
            - Subdomain & Vhosts Fuzzing
            - Directory & Page Fuzzing
            - Parameter & Value Fuzzing
        - Tools
    - Application Enumeration
        - WordPress
        - Joomla
        - Drupal
        - Tomcat
        - Jenkins
    - Active Directory Enumeration
        - Host Enumeration
        - User Enumeration
        - Password Policy Enumeration
        - SMB Enumeration
        - ACL Enumeration
        - Enumeration Trust Relationships
        - Tools


{{< callout insight "💡 Want to build your own Field Manual?" >}}

Check out this video it walks through exactly how to set one up from scratch.

{{< /callout>}}

{{< youtube 7LU6m_CF3cQ >}}

## Hands-on labs & practical preparation

Once you've finished the course content, the next focus is building a solid methodology and for that, I'd highly recommend the dedicated HTB tracks. They're well-aligned with the CPTS path and specifically designed to reinforce what you've learned. I'd suggest spending around a month here before even thinking about the exam.

I completed both the [CPTS Track](https://app.hackthebox.com/tracks/76) and the [AD Track](https://app.hackthebox.com/tracks/60) between the two, you'll cover most of what the exam tests.


{{< image-row "/images/cptstrack.png" "CPTS Track" "/images/cptsad.png" "AD Track" >}}

{{< callout insight "💡 Want to go further?" >}}

If you want more practice, HTB Pro Labs are an option but be aware they go beyond the CPTS scope. Treat them as a bonus, not a requirement.

{{< /callout>}}

### When You Know You're Ready for the Exam
Honestly? You'll never feel 100% ready and that's fine. If you've finished the course content, completed the tracks, and built your Field Manual, you are more than prepared. At some point you just have to sit down and go for it.

One thing I really appreciate about HackTheBox is that they give you **two exam attempts**. So don't go in paralysed by pressure. If you don't pass the first time, HTB will give you detailed feedback based on your report, pointing you to exactly what you need to revisit.

{{< callout warning "⚠️ Important - second attempt requires a report" >}}

To unlock your second attempt, you must submit a report even if you didn't pass, and even if the report is blank. Don't skip this step or you'll lose your retry.
{{< /callout>}}

## Exam preparation strategy

There's no secret formula here just a handful of habits that, if you stick to them throughout the entire journey, will put you in a genuinely strong position by exam day.

- **Go slow and understand every attack:** Don't just follow along with the module and move on. Ask yourself how does this work? Why does it work? What breaks it? That depth of understanding is exactly what the exam tests. You can't Google your way through it.

- **Build your Field Manual as you go:** Don't save note-taking for later. Every module, every exercise, every lab in the CPTS track if you learn a technique or a syntax you know you'll forget, write it down immediately. Your Field Manual is your exam companion.

- **Set a fixed exam date from the start:** Consistency and motivation are hard to maintain over months. What helped me was treating the exam date like a deadline not something flexible. Pick a date, put it on your calendar, and let that drive your pace. Without it, it's easy to drift.

- **Revisit your weak areas before the exam:** Once you've finished the full course and labs, go back through the topics where you felt shaky. Don't revise everything just the gaps. That targeted review in the final stretch is far more effective than going through everything again.

- **Pace yourself based on what you already know:** If a topic is familiar, move through it faster. If it's new territory, slow down and take your time. There's no prize for finishing the course quickly there's only the exam, and you want to be ready for it.


## Exam day tips & experience
The CPTS exam is a 10-day practical where you need to compromise an entire network, capture `12/14 flags`, and submit a commercial-grade penetration testing report. It sounds intimidating it isn't, if you've done the prep.

- **Stay calm:** You've put in the work. You don't need all 10 days just stay hydrated, sleep properly, and trust your preparation.

- **Enumerate constantly:** It's a full network multiple machines chained together. Getting to the next machine always starts with thorough enumeration. When in doubt, enumerate more.

- **If you're completely stuck, take a step back:** If you've tried everything and nothing is moving, you're probably in a rabbit hole. Stop poking around aimlessly, take a short break, and come back with fresh eyes. Nine times out of ten the answer is enumerate again.

- **1st and 8th flags are the tricky ones:** Don't let them shake you. Stay with the flow, enumerate harder, and you'll get there.

- **Sleep Seriously:** It's a 10-day exam. Don't spend 24 hours straight at your desk take naps, step outside, eat properly. Your brain needs rest to think clearly, and a rested mind will find what an exhausted one keeps missing.

### Pivoting with Ligolo-ng

The CPTS exam requires double pivoting at certain points you're not just moving from your machine into one internal subnet, you're chaining through multiple hosts to reach deeper targets. If you try to do this with Chisel and proxychains over SOCKS, you'll hit two problems fast: it's complex to set up correctly, and once it's running, even a simple nmap scan will be painfully slow.

**Ligolo-ng solves both:** It creates a proper routed network interface on your attack machine, so every tool works natively at full speed no proxy overhead, no compatibility issues. When you're doing double pivoting in the exam, this difference is massive. Set it up before you need it and understand how it works don't learn it under exam pressure.

{{< callout info "🛠️ Full setup guide" >}}

I've written a dedicated blog post covering exactly how to set up and use Ligolo-ng for pivoting from installing the proxy and agent to adding routes and handling multiple subnets. Check it out here: [your Ligolo blog post link]
{{< /callout >}}

### BloodHound And PowerView

When you hit Active Directory and BloodHound isn't showing anything useful beyond the node you already own don't trust BloodHound alone. If you're confident you're on the right path, cross-reference with PowerView. It often surfaces relationships and permissions that BloodHound misses entirely.

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

### Writing the Report

The report is **50% of the exam**. You can capture all 14 flags and still fail if your report isn't up to standard. Mine ended up being 170 pages.

{{< callout warning "⚠️ Don't leave the report until the end" >}}

Write as you go. Document each finding the moment you find it don't wait until you have 12 flags and then try to reconstruct everything from memory. The report takes serious time.

{{< /callout >}}

Use AI to help write and polish your findings. It's fast, it's allowed, and it makes a real difference in quality. Here's the kind of prompt that works well:

```text
Hey, I am writing my CPTS exam report help me make this attack chain clear and professional.

The tester found that the user Jimmy has AddMember rights over the HelpDesk group.
The tester added themselves to the group and leveraged the access to escalate privileges.
```

{{< callout info "🛠️ Use SysReptor for your report" >}}

It's fast, structured, and purpose-built for pentest reports. Highly recommended over writing in Word or Google Docs.

{{< /callout>}}
 

{{< youtube ItVsQmHLicc >}}


## Conclusion

That's the full picture from the moment I first heard the word **cybersecurity** in 10th grade to the email from HackTheBox on January 2nd that changed everything.

CPTS is not an easy certification. It will test your patience, your consistency, and your ability to think methodically under pressure. But it is absolutely achievable and the version of you that comes out the other side will have a level of practical skill that most certifications simply don't build.

If I could leave you with one thing it's this: don't wait until you feel ready. Build your Field Manual, finish the course, grind the labs, set your exam date, and go. The preparation is the journey the exam is just the confirmation.

{{< callout insight "💬 One last thing" >}}

If you're a BCA student, a beginner, or someone who doesn't come from a professional security background this cert is for you too. I'm proof of that. You don't need a perfect background. You just need to show up every day and keep going.

{{< /callout>}}

Good luck. You've got this.