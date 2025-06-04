---
layout: post
title: "Why High CVSS Scores Don't Always Mean Real Danger"
tags: [Cyber Security, SSDLC, Security Engineering]
color: rgb(160, 7, 27)
permalink: cvss-is-not-everything/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

You check your vulnerability scanner and see red alerts everywhere. Critical vulnerabilities with CVSS scores of 9.0 and higher are lighting up your dashboard like a Christmas tree. Your heart rate spikes. Time to panic, right?

Not so fast.

Here's something that might surprise you: a vulnerability with a CVSS score of 9.8 might be less dangerous to your systems than one scored at 6.5. The reason comes down to one simple truth - CVSS scores measure technical severity, not real-world risk.

<!--more-->

<hr>

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}

<hr>

## Understanding What CVSS Actually Measures

CVSS (Common Vulnerability Scoring System) works like a calculator. It takes technical factors about a vulnerability and spits out a number. The system looks at things like:

- Can attackers reach this from the internet?
- How hard is it to exploit?
- What damage can it cause?

But CVSS doesn't know anything about your environment. It doesn't know if you have firewalls, network segmentation, or other security controls. It's like rating how dangerous a lion is without knowing if it's in the wild or behind bars at a zoo.

## Real Examples Where High Scores Don't Equal High Risk

Let's look at some actual vulnerabilities to see how this plays out.

### Example 1: The Python Cookie Vulnerability [(CVE-2024-7592)](https://ubuntu.com/security/CVE-2024-7592)

This vulnerability got a CVSS score of 7.5, which sounds scary. The problem affects Python's way of handling web cookies. An attacker could send specially crafted cookies that make your server work really hard, potentially causing slowdowns.

But here's why most organizations treated this as low priority:

Most web servers limit how big HTTP requests can be. This makes the attack much harder to pull off. If your web server only accepts small requests, an attacker can't send the huge malicious cookies needed to cause problems. The vulnerability exists, but your existing protections make it nearly impossible to exploit.

### Example 2: Log4Shell in Isolated Systems

Log4Shell (CVE-2021-44228) received the maximum CVSS score of 10.0. And rightly so - it's a serious vulnerability that lets attackers run code on your servers just by sending a log message.

But imagine you have a system that uses Log4j but runs in an isolated network with no internet access. Maybe it's a factory control system or an internal application that only talks to a database. Even though the vulnerability has a perfect 10.0 score, it might be practically unexploitable in your environment.

The vulnerability is still there, but your network setup makes it much less dangerous than the CVSS score suggests.

### Example 3: Database Remote Code Execution

Consider a database vulnerability with a CVSS score of 9.1 that allows remote code execution. Sounds terrible, right? But what if:

- Your database only accepts connections from your application servers
- Those application servers are in a separate network zone
- The database runs with limited permissions
- You have monitoring that alerts on unusual database activity

Suddenly, that 9.1 vulnerability becomes much more manageable. An attacker would need to compromise your application first, then break through network controls, and even then they'd have limited access.

<svg viewBox="0 0 850 580" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="highRisk" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:#ff4444;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#cc0000;stop-opacity:1" />
    </linearGradient>
    <linearGradient id="mediumRisk" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:#ff8800;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#cc6600;stop-opacity:1" />
    </linearGradient>
    <linearGradient id="lowRisk" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:#44dd44;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#228822;stop-opacity:1" />
    </linearGradient>
    <filter id="shadow" x="-20%" y="-20%" width="140%" height="140%">
      <feDropShadow dx="2" dy="2" stdDeviation="3" flood-color="#00000030"/>
    </filter>
  </defs>

  <!-- Background -->
  <rect width="850" height="580" fill="#f8f9fa"/>
  
  <!-- Title -->
  <text x="400" y="40" text-anchor="middle" font-family="Arial, sans-serif" font-size="24" font-weight="bold" fill="#2c3e50">
    CVSS Score vs Real-World Risk
  </text>
  
  <!-- Subtitle -->
  <text x="400" y="65" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" fill="#666">
    Why high CVSS doesn't always mean high danger
  </text>

  <!-- Scenario 1: High CVSS, Low Real Risk -->
  <g transform="translate(50, 100)">
    <!-- Server -->
    <rect x="20" y="40" width="60" height="80" rx="8" fill="#4a90e2" filter="url(#shadow)"/>
    <rect x="30" y="50" width="40" height="4" fill="#fff" opacity="0.7"/>
    <rect x="30" y="58" width="40" height="4" fill="#fff" opacity="0.7"/>
    <rect x="30" y="66" width="40" height="4" fill="#fff" opacity="0.7"/>
    <text x="50" y="140" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#333">
      Database Server
    </text>
    <text x="50" y="155" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">
      CVE with CVSS 9.1
    </text>

    <!-- Firewall barriers -->
    <rect x="120" y="30" width="8" height="100" fill="#e74c3c" rx="2"/>
    <text x="124" y="20" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="#e74c3c" font-weight="bold">
      FIREWALL
    </text>
    
    <rect x="160" y="30" width="8" height="100" fill="#e74c3c" rx="2"/>
    <text x="164" y="20" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="#e74c3c" font-weight="bold">
      NETWORK
    </text>
    <text x="164" y="145" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="#e74c3c" font-weight="bold">
      ISOLATION
    </text>

    <!-- Attacker -->
    <circle cx="250" cy="80" r="20" fill="#ff6b6b" filter="url(#shadow)"/>
    <text x="250" y="85" text-anchor="middle" font-family="Arial, sans-serif" font-size="20">👤</text>
    <text x="250" y="110" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#333">
      Attacker
    </text>

    <!-- Blocked arrows -->
    <path d="M 230 80 L 180 80" stroke="#ff6b6b" stroke-width="3" fill="none" marker-end="url(#arrowhead)" stroke-dasharray="5,5"/>
    <path d="M 170 80 L 140 80" stroke="#ff6b6b" stroke-width="3" fill="none" marker-end="url(#arrowhead)" stroke-dasharray="5,5"/>
    
    <!-- X marks for blocked -->
    <g transform="translate(185, 75)">
      <circle r="8" fill="#ff4444" opacity="0.8"/>
      <path d="M -4 -4 L 4 4 M 4 -4 L -4 4" stroke="white" stroke-width="2"/>
    </g>
    
    <g transform="translate(145, 75)">
      <circle r="8" fill="#ff4444" opacity="0.8"/>
      <path d="M -4 -4 L 4 4 M 4 -4 L -4 4" stroke="white" stroke-width="2"/>
    </g>

    <!-- Risk assessment -->
    <rect x="300" y="50" width="120" height="60" fill="white" stroke="#ddd" rx="8" filter="url(#shadow)"/>
    <text x="360" y="70" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#333">
      Actual Risk
    </text>
    <rect x="320" y="80" width="80" height="15" fill="url(#lowRisk)" rx="7"/>
    <text x="360" y="105" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#228822" font-weight="bold">
      LOW
    </text>
  </g>

  <!-- Scenario 2: Medium CVSS, High Real Risk -->
  <g transform="translate(50, 350)">
    <!-- Web Server -->
    <rect x="20" y="40" width="60" height="80" rx="8" fill="#27ae60" filter="url(#shadow)"/>
    <circle cx="35" cy="60" r="4" fill="#fff" opacity="0.7"/>
    <circle cx="50" cy="60" r="4" fill="#fff" opacity="0.7"/>
    <circle cx="65" cy="60" r="4" fill="#fff" opacity="0.7"/>
    <rect x="30" y="75" width="40" height="30" fill="#fff" opacity="0.2" rx="4"/>
    <text x="50" y="140" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#333">
      Web Server
    </text>
    <text x="50" y="155" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">
      CVE with CVSS 6.5
    </text>

    <!-- Internet cloud -->
    <ellipse cx="200" cy="80" rx="40" ry="25" fill="#87ceeb" opacity="0.7" filter="url(#shadow)"/>
    <text x="200" y="85" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#333">
      Internet
    </text>

    <!-- Direct access arrow -->
    <path d="M 160 80 L 90 80" stroke="#ff6b6b" stroke-width="4" fill="none" marker-end="url(#arrowhead)"/>
    <text x="125" y="100" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="#ff6b6b" font-weight="bold">
      DIRECT ACCESS
    </text>

    <!-- Multiple attackers -->
    <circle cx="280" cy="60" r="15" fill="#ff6b6b" filter="url(#shadow)"/>
    <text x="280" y="65" text-anchor="middle" font-family="Arial, sans-serif" font-size="14">👤</text>
    
    <circle cx="310" cy="80" r="15" fill="#ff6b6b" filter="url(#shadow)"/>
    <text x="310" y="85" text-anchor="middle" font-family="Arial, sans-serif" font-size="14">👤</text>
    
    <circle cx="300" cy="110" r="15" fill="#ff6b6b" filter="url(#shadow)"/>
    <text x="300" y="115" text-anchor="middle" font-family="Arial, sans-serif" font-size="14">👤</text>

    <text x="300" y="135" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#333">
      Many Attackers
    </text>

    <!-- Risk assessment -->
    <rect x="350" y="50" width="120" height="60" fill="white" stroke="#ddd" rx="8" filter="url(#shadow)"/>
    <text x="410" y="70" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#333">
      Actual Risk
    </text>
    <rect x="370" y="80" width="80" height="15" fill="url(#highRisk)" rx="7"/>
    <text x="410" y="105" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#cc0000" font-weight="bold">
      HIGH
    </text>
  </g>

  <!-- Key factors box -->
  <rect x="540" y="150" width="280" height="300" fill="white" stroke="#ddd" rx="12" filter="url(#shadow)"/>
  <rect x="540" y="150" width="280" height="40" fill="#3498db" rx="12"/>
  <rect x="540" y="175" width="280" height="15" fill="#3498db"/>
  
  <text x="680" y="175" text-anchor="middle" font-family="Arial, sans-serif" font-size="16" font-weight="bold" fill="white">
    Key Factors That Matter
  </text>

  <g transform="translate(560, 210)">
    <!-- Network Access -->
    <circle cx="10" cy="10" r="6" fill="#e74c3c"/>
    <text x="25" y="15" font-family="Arial, sans-serif" font-size="12" fill="#333" font-weight="bold">
      Network Access
    </text>
    <text x="25" y="30" font-family="Arial, sans-serif" font-size="10" fill="#666">
      Can attackers reach the system?
    </text>

    <!-- Existing Protections -->
    <circle cx="10" cy="50" r="6" fill="#f39c12"/>
    <text x="25" y="55" font-family="Arial, sans-serif" font-size="12" fill="#333" font-weight="bold">
      Existing Protections
    </text>
    <text x="25" y="70" font-family="Arial, sans-serif" font-size="10" fill="#666">
      Firewalls, monitoring, segmentation
    </text>

    <!-- Asset Value -->
    <circle cx="10" cy="90" r="6" fill="#9b59b6"/>
    <text x="25" y="95" font-family="Arial, sans-serif" font-size="12" fill="#333" font-weight="bold">
      Asset Value
    </text>
    <text x="25" y="110" font-family="Arial, sans-serif" font-size="10" fill="#666">
      What data/systems are at risk?
    </text>

    <!-- Exploit Difficulty -->
    <circle cx="10" cy="130" r="6" fill="#27ae60"/>
    <text x="25" y="135" font-family="Arial, sans-serif" font-size="12" fill="#333" font-weight="bold">
      Exploit Difficulty
    </text>
    <text x="25" y="150" font-family="Arial, sans-serif" font-size="10" fill="#666">
      How hard is it really to exploit?
    </text>

    <!-- Business Context -->
    <circle cx="10" cy="170" r="6" fill="#34495e"/>
    <text x="25" y="175" font-family="Arial, sans-serif" font-size="12" fill="#333" font-weight="bold">
      Business Context
    </text>
    <text x="25" y="190" font-family="Arial, sans-serif" font-size="10" fill="#666">
      Impact on operations and users
    </text>

    <!-- Bottom message -->
    <rect x="0" y="195" width="240" height="40" fill="#ecf0f1" rx="6"/>
    <text x="120" y="210" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#2c3e50" font-weight="bold">
      CVSS Score ≠ Real Risk
    </text>
    <text x="120" y="225" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#7f8c8d">
      Context is everything
    </text>
  </g>

  <!-- Arrow marker definition -->
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" 
     refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#ff6b6b" />
    </marker>
  </defs>

  <!-- Legend at bottom -->
  <g transform="translate(50, 550)">
    <text x="0" y="0" font-family="Arial, sans-serif" font-size="12" fill="#333" font-weight="bold">
      Takeaway:
    </text>
    <text x="70" y="0" font-family="Arial, sans-serif" font-size="12" fill="#666">
      A protected system with high CVSS can be safer than an exposed system with lower CVSS
    </text>
  </g>
</svg>

## What Actually Matters in the Real World

Instead of just looking at CVSS scores, smart security teams ask these questions:

**Can attackers actually reach the vulnerable system?** A critical vulnerability on a system with no network access is often less urgent than a medium-severity bug on your public website.

**What would an attacker gain?** A vulnerability that gives access to test data is different from one that exposes customer credit cards.

**Do you have other protections in place?** Firewalls, monitoring, access controls, and network segmentation all change the risk calculation.

**How hard is it really to exploit?** Some vulnerabilities need perfect conditions that rarely exist in real environments.

## The Bottom Line

CVSS scores are useful starting points, but they're not the whole story. Think of them like weather forecasts - they give you general guidance, but you still need to look outside your window to see what's actually happening.

The most effective security teams use CVSS scores as one input among many. They consider their specific environment, existing protections, and business context to make smart decisions about which vulnerabilities to fix first.

Remember: a vulnerability that's technically severe but practically unexploitable in your environment is often less important than one that's moderately severe but easy for attackers to reach and exploit.

The goal isn't to fix every high CVSS score vulnerability immediately. The goal is to reduce real risk to your organization. Sometimes that means focusing on the 6.5 vulnerability instead of the 9.8 one.

---

*Your security scanner might disagree, but your actual security will be better for it.*

---


**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
