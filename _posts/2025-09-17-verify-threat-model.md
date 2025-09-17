---
layout: post
title: "Did Your Threat Model Actually Work?"
tags: [Threat Modelling, Security Engineering, DevSecOps]
color: rgb(159, 3, 135)
permalink: verify-threat-model/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

You've done the hard work. Your team created a comprehensive threat model, identified risks, and handed recommendations to the development team. But here's the million-dollar question: **How do you know they actually implemented the security controls?**

If you're like most security teams, you're probably hoping developers added the right code comments, wrote proper tests, and followed all your recommendations. But let's be honest – that's not always realistic.

Here's how to independently verify that your threat model recommendations actually made it into production.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>

## The Problem with Developer-Dependent Verification

Most security teams make this mistake:
- **"Did you implement input validation for threat XYZ?"**
- **"Can you show me the code that handles authentication?"**
- **"Please add threat IDs to your code comments."**

The problem? You're completely dependent on developers to self-report their security implementations. That's like asking students to grade their own exams.

## The Independent Verification Approach

Instead of asking developers what they built, **test what actually works**. Here are three simple methods any security team can use:

### 1. Automated Black-Box Testing

Create simple scripts that test your threats directly:

```bash
# Test authentication bypass (Threat: AUTH-001)
curl -X GET https://yourapp.com/admin
# Should return 401, not 200

# Test for SQL injection (Threat: DB-001)
curl "https://yourapp.com/search?q=' OR '1'='1"
# Should not return database errors
```

No code access needed. No developer cooperation required.

### 2. Use Security Scanning Tools

Configure your existing security tools to focus on your threat model:

**Static Analysis (SAST) - Check the Code Itself:**
- **Semgrep/Snyk/Checkmarx**: Create custom rules for your specific threats
  - Threat AUTH-001 (Weak Authentication): Scan for hardcoded passwords, weak crypto
  - Threat INJECTION-001 (SQL Injection): Look for dynamic SQL queries without parameterization
  - Threat XSS-001 (Cross-Site Scripting): Find unescaped user input in templates

```bash
# Example: Custom Semgrep rule for SQL injection threat
rules:
  - id: sql-injection-python
    message: "Potential SQL injection - using string formatting in execute()"
    languages: [python]
    pattern: |
      $CURSOR.execute("..." % ...)
    severity: WARNING

# This rule will catch this:
query = "SELECT * FROM users WHERE name = '%s'" % user_input  # Flagged!
cursor.execute(query)

# But won't flag this (safe approach):
cursor.execute("SELECT * FROM users WHERE name = %s", (user_input,))  # Safe!
```

**Dynamic Analysis (DAST) - Test the Running App:**
- **OWASP ZAP**: Set up custom scan policies for each threat category
- **Burp Suite**: Create specific test cases targeting your identified risks

**Infrastructure Scanning:**
- **Nmap**: Verify network security controls are in place
- **AWS Config/Azure Policy**: Check cloud configurations match threat requirements

The key is mapping your threats to specific tool configurations and creating custom rules that look for exactly what your threat model identified.

### 3. Simple Configuration Checks

Many threats can be verified by checking configurations:

```bash
# Check if sensitive data is encrypted (Threat: DATA-001)
grep -r "password\|api_key" config/ | grep -v "encrypted"

# Verify HTTPS enforcement (Threat: NET-001)
curl -I http://yourapp.com | grep -i "redirect\|301\|302"
```

## Connecting Verification to Penetration Testing

Here's where it gets powerful. Create a simple mapping between your threat model and pentest scenarios:

| Threat ID | Threat Description | Pentest Test Case |
|-----------|-------------------|-------------------|
| AUTH-001 | Authentication bypass | Try accessing admin pages without login |
| INJECTION-001 | SQL injection in search | Test search with malicious SQL payloads |
| XSS-001 | Cross-site scripting | Submit script tags in form fields |

Share this with your pentesters. Now they're not just finding random vulnerabilities – they're specifically validating your threat model.

## Start Simple: The 3-Step Approach

**Week 1: Pick Your Top 5 Threats**
- Choose the most critical threats from your threat model
- Write one simple test for each (like the curl examples above)

**Week 2: Automate the Tests**
- Put your tests in a script
- Run them weekly against your applications
- Track results in a simple spreadsheet

**Week 3: Expand Coverage**
- Add more threats to your testing
- Configure security scanners to focus on your specific risks
- Share results with development teams

## Making It Sustainable

The best approach is the one you'll actually use. Start with:

1. **Simple scripts** that test basic functionality
2. **Existing tools** configured for your threats
3. **Regular testing** (weekly or monthly)
4. **Clear reporting** that maps back to your threat model

Don't try to verify everything at once. Pick the threats that keep you up at night and start there.

## The Bottom Line

Your threat model is only as good as its implementation. Stop relying on developers to self-report their security work.

Build simple verification methods that give you confidence your security controls actually exist in production.

Because at the end of the day, attackers don't care about your documentation – they care about what's actually running in production.

---

*Want to get started? Pick one critical threat from your threat model and write a simple test for it this week. You'll be surprised what you discover.*

---


**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
