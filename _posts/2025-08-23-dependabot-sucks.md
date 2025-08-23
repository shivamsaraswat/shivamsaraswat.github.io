---
layout: post
title: "Why Every Developer Thinks Dependabot Sucks (And They're Right)"
tags: [Security at Scale, Security Engineering, DevSecOps, Dependabot]
color: rgb(102, 0, 175)
permalink: dependabot-sucks/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

Dependabot gets a lot of criticism as an SCA tool, and much of it is justified. While GitHub markets it as a security solution, it lacks core capabilities that development teams need for managing supply chain risks.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>

## The Transitive Dependency Problem

The most significant weakness is Dependabot's inability to handle transitive dependencies across most ecosystems. Looking at [GitHub's own documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/dependency-graph-supported-package-ecosystems#supported-package-ecosystems), only 4 out of 15 supported package managers have built-in transitive dependency analysis:

- Maven (Java, Scala)
- npm, pnpm (JavaScript)  
- Yarn (JavaScript)

This means 73% of ecosystems lack native transitive dependency scanning. Python, Go, Rust, Ruby, .NET, and PHP developers get zero visibility into indirect dependencies unless they manually configure dependency submission actions.

Consider this: if you're using pip for Python, Dependabot only scans your direct dependencies listed in requirements.txt. But your actual vulnerability exposure comes from hundreds of transitive dependencies you never directly specified. A vulnerability in a sub-dependency three levels deep remains invisible.

GitHub only [added transitive support for Maven in March 2025](https://github.blog/changelog/2025-03-26-transitive-dependencies-are-now-available-for-maven/). [NPM support](https://github.blog/security/supply-chain-security/unlocking-security-updates-for-transitive-dependencies-with-npm/) came earlier but still recently. Meanwhile, dedicated SCA tools have offered this functionality for years across all ecosystems.

## What's Bad About Dependabot

### Alert Fatigue & Noise

- Generates tons of pull requests and security alerts without proper prioritization
- Treats all vulnerabilities equally regardless of actual exploitability or business impact
- Creates PR spam that teams often ignore, defeating the purpose

### Lack of Context

- No understanding of whether vulnerable code paths are actually reachable in your application
- Can't distinguish between a dev dependency vulnerability and a critical runtime exposure
- Missing business context about which applications are customer-facing vs internal tools

### Poor Quality Intelligence

- Relies heavily on automated vulnerability detection that produces false positives
- Often flags vulnerabilities in transitive dependencies that aren't exploitable in your context
- Limited remediation guidance beyond "update to version X"

### Workflow Disruption

- Automated PRs can break builds without warning
- Limited customization for update schedules and policies
- Doesn't integrate well with existing security review processes

### Basic Reporting

- Minimal analytics and trending data
- Hard to get organization-wide visibility into security posture
- No risk scoring or business impact assessment

## Technical Limitations Beyond Transitive Dependencies

The [documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/dependency-graph-supported-package-ecosystems) reveals other problems:

- Python setup.py files may not parse completely
- GitHub Actions only get alerts for semantic versioning, not SHA-pinned versions
- Workflows must be in `.github/workflows/` to be recognized
- No license compliance scanning
- Basic SBOM generation - While GitHub [introduced SBOM export in 2023](https://github.blog/enterprise-software/governance-and-compliance/introducing-self-service-sboms/), it only generates SBOMs based on the dependency graph, meaning the same transitive dependency limitations apply. The SBOM will be incomplete for 73% of ecosystems.
- No container or binary analysis

As noted in the [GitHub Actions documentation](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts): "Dependabot will only create Dependabot alerts for vulnerable GitHub Actions that use semantic versioning. You will not receive alerts for a vulnerable action that uses SHA versioning."

The [SBOM export documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/exporting-a-software-bill-of-materials-for-your-repository) confirms that SBOMs are generated from the dependency graph, inheriting all its limitations. For ecosystems without transitive dependency support, your SBOM will miss critical components, making it unsuitable for compliance requirements like Executive Order 14028.

## What's Actually Good About It

### Accessibility

- Free for public repos, cheap for private ones
- Zero setup complexity if you're already on GitHub
- Covers most major package ecosystems (npm, pip, bundler, etc.)

### Automation

- Does catch real vulnerabilities automatically
- Handles the tedious work of dependency version tracking
- Better than having no SCA tool at all

### Integration

- Native GitHub integration means it fits into existing workflows
- Plays reasonably well with GitHub's security tab and insights

## The Reality Check

Dependabot works okay for small teams or projects where someone can manually triage alerts, but it scales poorly and lacks the intelligence that modern development teams need. Many organizations end up supplementing it with tools like Snyk Open Source, Semgrep Supply Chain or building custom filtering on top of it.

It's basically the "good enough" option that's better than nothing but frustrating if you need actual strategic dependency management.

## When Dependabot Makes Sense

Despite its limitations, Dependabot can work for:

- Open source projects with limited budgets
- Small teams without dedicated security resources  
- Projects with simple dependency trees
- Organizations just starting their security journey

If you're stuck with Dependabot, you can improve the experience by implementing proper configuration and workflows. This guide shows how to set up automatic PR management using Dependabot: [[How to Make Dependabot Actually Work for You](https://blog.shivamsaraswat.com/dependabot-automatic-pr/)]

## The Bottom Line

Dependabot is not a comprehensive SCA solution. It's a basic dependency updater with vulnerability alerts bolted on. The lack of transitive dependency support for 73% of ecosystems makes it unsuitable for serious security programs.

For organizations that need real supply chain security, look at dedicated SCA tools that offer:

- Complete transitive dependency analysis across all languages
- Reachability analysis to reduce false positives
- License compliance scanning
- Risk-based prioritization
- Detailed remediation guidance
- Enterprise reporting and governance

## Sources

1. [GitHub Dependency Graph Supported Package Ecosystems](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/dependency-graph-supported-package-ecosystems)
2. [Transitive Dependencies Now Available for Maven - GitHub Blog](https://github.blog/changelog/2025-03-26-transitive-dependencies-are-now-available-for-maven/)
3. [Unlocking Security Updates for Transitive Dependencies with npm - GitHub Blog](https://github.blog/security/supply-chain-security/unlocking-security-updates-for-transitive-dependencies-with-npm/)
4. [About Dependabot Alerts - GitHub Docs](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts)
5. [About Dependabot Version Updates - GitHub Docs](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/about-dependabot-version-updates)
6. [Workflow Syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

*Dependabot might be free, but the cost of missing vulnerabilities in your dependency tree could be much higher than investing in proper SCA tooling.*

---

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
