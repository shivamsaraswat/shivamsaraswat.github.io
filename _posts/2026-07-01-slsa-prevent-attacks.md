---
layout: post
title: "Could SLSA Have Stopped the Recent npm Supply Chain Attacks?"
tags: [Security Engineering, Supply Chain Security, SLSA, OpenSSF, Open Source]
color: rgb(0, 89, 206)
permalink: slsa-prevents-attacks/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

The last year saw a run of software supply chain attacks against the npm
ecosystem. Packages used by millions of projects were replaced with versions
that stole credentials, drained crypto wallets, or installed remote access
trojans. This post lists the main incidents, explains the SLSA framework, and
works through which attacks SLSA could have stopped and at which level.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>

## A short list of recent attacks

### chalk, debug, and 18 other packages (September 8, 2025)
An attacker phished the npm maintainer "qix" (Josh Junon) with a fake 2FA reset
email from the domain `npmjs.help`. The attacker took over the account and
published versions of 18-20 packages, including `chalk` and `debug`, that
together have over 2 billion weekly downloads. The payload was a browser script
that rewrote crypto wallet addresses during transactions. The versions were live
for about two hours.
Source: [Palo Alto Networks](https://www.paloaltonetworks.com/blog/cloud-security/npm-supply-chain-attack/),
[The Hacker News](https://thehackernews.com/2025/09/20-popular-npm-packages-with-2-billion.html)

### Shai-Hulud worm (September 15, 2025)
A self-replicating worm spread through npm. When it ran in an environment that
held npm tokens, it published new versions of any package those tokens could
reach. It also harvested credentials and sent them to public GitHub
repositories. It compromised over 500 packages. Researchers linked it to the
earlier s1ngularity/Nx token theft from late August 2025.
Source: [CISA Alert](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem),
[Unit 42](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/),
[Wiz](https://www.wiz.io/blog/shai-hulud-npm-supply-chain-attack)

### Shai-Hulud 2.0 (November 2025)
A second wave used the same propagation idea but moved the payload to the
pre-install step, which widened the impact. It affected tens of thousands of
GitHub repositories and added a fallback that could delete a user's home
directory.
Source: [Unit 42](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/),
[Datadog Security Labs](https://securitylabs.datadoghq.com/articles/shai-hulud-2.0-npm-worm/)

### axios (March 31, 2026)
An attacker took over the npm account of the axios maintainer (`jasonsaayman`),
changed the account email, and published two versions (`1.14.1` and `0.30.4`).
These versions added a dependency, `plain-crypto-js`, whose post-install script
ran a cross-platform remote access trojan on Windows, macOS, and Linux. axios
has over 100 million weekly downloads. The attacker published with a long-lived
npm token from the CLI and bypassed the project's GitHub Actions OIDC publishing
workflow. The legitimate `1.14.0` had been published through GitHub Actions; the
malicious `1.14.1` had no matching GitHub tag, release, or commit. Google and
Microsoft attributed the attack to a North Korea-linked group.
Source: [Microsoft](https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/),
[Google Threat Intelligence](https://cloud.google.com/blog/topics/threat-intelligence/north-korea-threat-actor-targets-axios-npm-package),
[Unit 42](https://unit42.paloaltonetworks.com/axios-supply-chain-attack/)

### Mini Shai-Hulud / TeamPCP (May 2026)
A group tracked as TeamPCP ran a worm that published versions through the
projects' own GitHub Actions release pipelines using hijacked OIDC tokens. It hit
TanStack, UiPath, AntV, and others. Because the malicious code went through the
real build pipeline, the published packages carried valid SLSA Build Level 3
provenance. This was the first reported npm worm to produce packages with valid
provenance.
Source: [StepSecurity](https://www.stepsecurity.io/blog/mini-shai-hulud-is-back-a-self-spreading-supply-chain-attack-hits-the-npm-ecosystem),
[Snyk](https://snyk.io/blog/mini-shai-hulud-antv-npm-supply-chain-attack/)

## What SLSA is

SLSA (Supply-chain Levels for Software Artifacts) is a framework from the
OpenSSF. It defines how to record and check the origin of a software artifact.
The record is called provenance: it states what built the artifact, what process
was used, and what the inputs were. A consumer sets an expectation for how a
package should be built, then checks each version's provenance against that
expectation.

SLSA is organized into tracks. Each track has levels. The current version is
v1.2, approved in November 2025.

- **Build Track** (stable since v1.0): levels L0 to L3.
  - L0: no provenance.
  - L1: provenance exists but can be forged.
  - L2: provenance is signed by a hosted build platform.
  - L3: the build runs in an isolated environment, and the signing key is not
    reachable from the build process.
- **Source Track** (added in v1.2): covers how source code is authored,
  reviewed, and managed before it reaches the builder.
- **Build Environment Track** and **Dependency Track**: still in draft.

Reference: [SLSA specification v1.2](https://slsa.dev/spec/v1.2/),
[SLSA security levels](https://slsa.dev/spec/v1.0/levels), [Understanding SLSA: Tracks, Levels, and the Checks at Each Step](https://blog.shivamsaraswat.com/understanding-slsa/)

## Could SLSA have prevented these attacks?

The attacks fall into two groups. SLSA handles them differently.

### Group 1: stolen credentials used to publish outside the build pipeline

This group includes chalk/debug, the original Shai-Hulud, Shai-Hulud 2.0, and
axios. In each case the attacker obtained publishing credentials (an account or
a token) and pushed versions that never went through the project's real build
process.

For axios, the legitimate releases came from GitHub Actions with OIDC. The
malicious release came from a manual token publish with no source commit behind
it. For Shai-Hulud, the worm spread by reusing stolen npm tokens to publish.

SLSA addresses this group through the Build Track:

- At **Build L2**, legitimate releases carry provenance signed by the build
  platform, linking the artifact to a source commit and a build process. A
  version published by hand with a stolen token cannot produce that signed
  provenance. A consumer who checks provenance against the expectation would
  reject it. So L2 lets you **detect** the bad version.
- At **Build L3**, the signing capability is out of reach of anyone holding only
  publish credentials. Stolen tokens are no longer enough to create a trusted
  release. The SLSA documentation states that higher build levels reduce the
  impact of compromised upload credentials. So L3 helps **prevent** the attack,
  not just detect it.

For this group the answer is: yes, **Build Track, L2 to detect, L3 to prevent**.

There is one condition. SLSA only stops these attacks if the registry or the
consumer actually checks provenance and rejects mismatches. Generating
provenance alone (L1) does nothing. The axios pipeline also showed a related
gap: it had OIDC set up but still passed a long-lived `NPM_TOKEN`, and npm used
the token when both were present. Removing standing tokens, which is part of the
L3 discipline, would have closed that path.

### Group 2: the build pipeline itself is compromised

This group is the Mini Shai-Hulud / TeamPCP wave. The attacker did not publish
outside the pipeline. The attacker hijacked OIDC tokens and published through the
real GitHub Actions pipeline. The result was malicious packages with valid SLSA
Build L3 provenance.

This shows a limit of the Build Track. SLSA provenance states where, how, and
from what source an artifact was built. It does not state that the source or the
build is safe. If the attacker controls the pipeline, the provenance is genuine
and verification passes.

For this group the Build Track is not enough. Useful controls here are:

- **Source Track**: protected branches, required review, and history integrity,
  to make unauthorized source changes and unauthorized releases harder.
- **Two-party review**: in [v1.2](https://slsa.dev/spec/v1.2/source-requirements#level-4-two-party-review) this is a policy requirement rather than a level.
- **Build Environment Track** [(draft)](https://slsa.dev/spec/draft/build-env-track-basics): aimed at the integrity of the build
  environment and the tokens it holds.

Even these do not fully prevent the attack. A privileged insider or a review
bypass can still get through. They raise the cost; they do not remove the risk.

## Summary

| Attack | Could SLSA have stopped it? | Track and level |
|---|---|---|
| chalk, debug, qix (Sep 2025) | Yes | Build L2 to detect, L3 to prevent, with verification |
| Shai-Hulud / 2.0 (Sep-Nov 2025) | Yes | Build L3 (removes the stolen-token publish path) |
| axios (Mar 2026) | Yes | Build L2 to detect, L3 to prevent, with verification |
| Mini Shai-Hulud / TeamPCP (2026) | No (Build Track alone) | Needs Source Track, two-party review, Build Environment Track; still partial |

---

*The common lesson across all of these: SLSA helps only when consumers or
registries verify provenance against an expected source and builder. [npm now
produces provenance](https://github.blog/changelog/2023-09-26-npm-provenance-general-availability/) and supports [OIDC trusted publishing](https://docs.npmjs.com/trusted-publishers), but most installs do
not check it. Provenance without verification is a record that no one reads.*

---

## Official and primary sources

- SLSA specification: [https://slsa.dev/spec/v1.2/](https://slsa.dev/spec/v1.2/)
- SLSA security levels: [https://slsa.dev/spec/v1.0/levels](https://slsa.dev/spec/v1.0/levels)
- Understanding SLSA: Tracks, Levels, and the Checks at Each Step: [https://blog.shivamsaraswat.com/understanding-slsa/](https://blog.shivamsaraswat.com/understanding-slsa/)
- CISA alert on the npm compromise: [https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem)
- Microsoft on the axios attack: [https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/](https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/)
- Google Threat Intelligence on the axios attack: [https://cloud.google.com/blog/topics/threat-intelligence/north-korea-threat-actor-targets-axios-npm-package](https://cloud.google.com/blog/topics/threat-intelligence/north-korea-threat-actor-targets-axios-npm-package)
- Unit 42 on Shai-Hulud and axios: [https://unit42.paloaltonetworks.com/npm-supply-chain-attack/](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/) and [https://unit42.paloaltonetworks.com/axios-supply-chain-attack/](https://unit42.paloaltonetworks.com/axios-supply-chain-attack/)

---

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top<svg width="12" height="12" viewBox="0 0 24 24" aria-hidden="true"><path fill="currentColor" d="M12 4l-8 8h5v8h6v-8h5z"/></svg></strong> </a>
