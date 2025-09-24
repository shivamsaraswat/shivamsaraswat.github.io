---
layout: post
title: "Why Running AI Coding Tools in Pipelines is a Security Risk"
tags: [AI, Security Engineering, DevSecOps]
color: rgb(94, 3, 159)
permalink: ai-threat-in-pipelines/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

Recently I have observed more development teams using AI coding assistants like Claude Code directly in their CI/CD pipelines. While the productivity gains can be impressive, this trend is raising serious security concerns that every organization should understand.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>

## What's Happening

Developers are integrating AI tools into their automated build and deployment processes. Instead of just using AI to write code on their local machines, they're having AI make changes automatically as part of their software delivery pipeline. This means AI can modify code, update configurations, and make decisions without human oversight.

## The Security Risks

### Your Code is Leaving Your Building

When you use AI tools in pipelines, your source code gets sent to external servers. This includes not just the code itself, but potentially sensitive configuration files, API keys, and business logic. You're essentially giving a third party access to your intellectual property and secrets.

### The Black Box Problem

We don't really know how these AI systems make decisions. When Claude Code suggests a change, we can't audit why it made that choice or predict what it might do in unusual situations. This makes it nearly impossible to do proper security reviews or debug problems when they occur.

### Pipeline Compromise Risks

AI tools running in your pipeline have the same permissions as your build system. If something goes wrong, the AI could accidentally break security controls, bypass safety gates, or make changes that introduce vulnerabilities. There's also the risk of prompt injection attacks where malicious input could trick the AI into doing something harmful.

### Real-World Example: The Nx Attack

In August 2025, attackers compromised the popular Nx build tool and specifically targeted AI development tools. The malware checked for Claude Code CLI and other AI tools, then used them with carefully crafted prompts to search for cryptocurrency wallets and credentials on infected systems.

What made this attack particularly dangerous was that the malicious logic was hidden in natural language prompts rather than traditional code, making it much harder for security tools to detect. Over 1,400 people had their credentials and wallet information stolen.

### Compliance Headaches

Many industries require detailed audit trails and accountability for code changes. When AI makes automated changes, it becomes difficult to meet these requirements. Who is responsible when the AI introduces a security bug? How do you explain to auditors that you can't fully explain why certain changes were made?

## What This Means for Your Organization

If your teams are experimenting with AI in pipelines, you need to establish guardrails now rather than later. The risks aren't just theoretical - attackers are already adapting to exploit these new tools.

### Key Questions to Ask

- Do we know what data our AI tools are sending to external services?
- Can we audit and review all AI-generated changes?
- Do we have proper access controls limiting what AI can modify?
- Are we meeting our compliance requirements with automated AI changes?
- Do we have incident response plans for AI-related security issues?

## Recommendations

This doesn't mean you should ban AI tools entirely. Instead, take a measured approach:

**Start with guardrails.** Require security review for any AI integration in pipelines. Establish clear policies about what data can be shared with external AI services.<br>
**Maintain human oversight.** Don't let AI make changes without human review, especially in production systems.<br>
**Know your data flows.** Understand exactly what information is being sent to AI services and ensure sensitive data is protected.<br>
**Plan for incidents.** Have a response plan ready for when things go wrong with AI-generated code.<br>

The potential of AI in development is enormous, but we need to be smart about how we adopt these tools. Security teams and developers need to work together to harness the benefits while managing the risks.

**Sources:**

- [Security Alert: NX Compromised to Steal Wallets and Credentials](https://semgrep.dev/blog/2025/security-alert-nx-compromised-to-steal-wallets-and-credentials/) - Semgrep
- [Official Nx Security Advisory](https://github.com/nrwl/nx/security/advisories/GHSA-cxm3-wv7p-598c) - GitHub
- [We Don't Actually Know How Claude Code Works - That's a Problem](https://www.linkedin.com/pulse/we-dont-actually-know-how-claude-code-works-thats-problem-knuth-vqu7f/) - LinkedIn

---

*The key is finding the right balance between innovation and security. Don't let the excitement of new AI capabilities blind you to the fundamental security principles that have served us well for decades.*

---


**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
