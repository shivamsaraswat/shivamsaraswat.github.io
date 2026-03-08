---
layout: post
title: "How to Actually Defend Against AI Static Threats"
tags: [Security Engineering, AI, AI Security, AI Threats, Static Threats, Secure AI]
color: rgb(0, 55, 206)
permalink: ai-static-threats-defense/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

In the [last post](/ai-static-threats/), we went through the static threats that live inside AI models before they ever see a user. Training data poisoning, model backdoors, serialization exploits, weight tampering, hardcoded behavior, weight theft, and compression artifacts. These are not theoretical. They are real, documented, and growing as a concern as more organizations build and deploy models at speed.

This post is about what you can do about them. Not in theory. In practice, today, with tools that exist.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>

## Start With the Right Mental Model

Before getting into specifics, it helps to reframe how you think about model deployment. Most teams treat the model as output: something you receive, load, and use. Security thinking lives at the edges, in the API, in the prompt handling, in access controls.

But the model file itself is infrastructure. It is more like a binary you are running than a document you are reading. And you should treat it the way you treat any third-party binary: with verification, scanning, and a clear chain of trust.

Once you have that framing, the rest follows.

---

## Measure 1: Verify Model Files Before Loading

The first thing to do is stop loading model files without checking them.

This sounds simple, but a huge number of workflows skip it. Someone runs a tutorial, copies a `torch.load()` call, and loads a model they pulled from a repository. That call can execute code in the file. No exploit needed. Just loading is enough.

**What to do:**

Use the `weights_only=True` parameter in PyTorch when loading model files. This blocks the code execution path.

```python
# Risky
model = torch.load("model.pt")

# Safe
model = torch.load("model.pt", weights_only=True)
```

Better yet, move to the `safetensors` format entirely. It was built specifically to avoid the serialization problem. It stores weights and nothing else. There is no code execution surface.

The [safetensors library](https://github.com/huggingface/safetensors) is maintained by Hugging Face and supports all the major frameworks. Switching is not hard, and most model hubs are beginning to prioritize it.

---

## Measure 2: Scan Models With Security Tools

Model scanning is a real thing now, not just a future idea. A few tools are worth knowing.

**ModelScan** — open-source, built by Protect AI. It scans model files for unsafe serialization patterns, including pickle exploits. You can run it before loading any downloaded model.

GitHub: [protectai/modelscan](https://github.com/protectai/modelscan)

```bash
pip install modelscan
modelscan -p path/to/model.pkl
```

**Protect AI Guardian** — the commercial version of the above, with a broader set of checks and integrations into ML pipelines.

**HiddenLayer Model Scanner** — another scanner that checks models for known malicious patterns. HiddenLayer has published research on backdoors in computational graphs (the ShadowLogic work), so their tooling reflects real threat research.

These tools will not catch everything, especially novel backdoors. But they catch known attack patterns and unsafe file formats, which covers a lot of the threat surface for models pulled from public repositories.

---

## Measure 3: Verify Hashes and Provenance

When you download a model from a repository, verify the hash. Most repositories provide SHA256 hashes alongside model files. Check them.

```bash
sha256sum model.safetensors
```

Compare the output to what the repository lists. If they match, the file is what was uploaded. That is not a full guarantee of safety, but it tells you no one tampered with it in transit or on the host.

For internal models, treat weights the way you treat build artifacts. Version them. Store them with checksums. Log who accessed them and when. If you are using object storage like S3 or GCS, enable versioning and access logging. Model weights can walk out the door just like source code can.

---

## Measure 4: Run Behavioral Evaluations Before Deployment

This is how you catch backdoors and hardcoded behavior that no file scanner will find, because they live in the weights, not in the file format.

The idea is to test the model across a range of inputs, including inputs that test for known backdoor trigger patterns, before you deploy it. This is called behavioral evaluation or red-teaming at the model level.

**What this looks like in practice:**

Build a test suite that covers the behaviors you care about. Run the model against edge cases, unusual inputs, and inputs that probe for inconsistency between what the model says about itself and what it does. If you are using a base model from a public source, run it through a standard set of jailbreak and trigger probes before you fine-tune.

**Tools that help:**

[Garak](https://github.com/NVIDIA/garak) — open-source, from NVIDIA. It is a framework for probing LLMs for a range of vulnerabilities including hallucination, data leakage, and backdoor-style behaviors. It is easy to run and generates reports you can act on.

```bash
pip install garak
python -m garak --model_type huggingface --model_name your-model --probes all
```

[LLM-Fuzzer](https://github.com/mnns/LLMFuzzer) — a fuzzing tool for language models, useful for finding unexpected behavior under input variation.

[Microsoft PyRIT](https://github.com/Azure/PyRIT) — Python Risk Identification Toolkit for Generative AI. Built for red-teaming AI systems at scale. Covers a range of risk categories and is well-documented.

None of these are backdoor detectors in a strict sense. But they are how you build evidence that a model behaves consistently, which is the closest thing you have to assurance.

---

## Measure 5: Audit Training Data Pipelines

If you are training or fine-tuning models on data you collected or sourced, you need a process for reviewing that data before it goes into training.

This is analogous to input validation in software security. You do not trust external input. You inspect it, filter it, and log what you kept and what you dropped.

**Practical steps:**

For web-scraped data, use content filtering to remove known malicious domains, hate content, and adversarial patterns. Tools like [Dolma](https://github.com/allenai/dolma) from AI2 include filtering pipelines you can adapt.

For fine-tuning data from internal or third-party sources, do a human review sample at minimum. If the dataset is large, sample 1-2% and have someone read it. You are looking for content that would teach the model something you did not intend.

Keep records of what data you trained on. If you discover a problem later, you need to know whether the training data is the likely source.

---

## Measure 6: Lock Down Access to Model Weights

Weight theft is often overlooked because it does not feel like a security incident until the model shows up somewhere it should not be.

Model weights are files. They need access controls.

**Basic controls:**

Restrict who can read model files in your storage systems. Not everyone on the team needs access to the weights. Apply least privilege: the inference server needs read access, most developers do not.

Log access to weight files. If a weight file is downloaded or copied, you want to know.

Encrypt model weights at rest and in transit. This does not stop an insider who has legitimate access, but it raises the bar for external attackers and reduces exposure from misconfigured storage buckets (a common issue).

If you are serving models via API, the weights should never be directly reachable by clients. The inference layer sits in between. But also be aware that a determined attacker with API access can probe your model to reconstruct its behavior, a process called model extraction. Rate limiting, output watermarking, and anomaly detection on query patterns are the defenses here.

---

## Measure 7: Validate Quantized or Compressed Models

If you are using a third-party quantized model, or if someone on your team compresses a model for deployment, treat the output as a new artifact that needs testing.

The compressed model is not the same model that was evaluated. Run your behavioral test suite against it. Check that safety behaviors held through compression. Check that outputs on your evaluation sets match expectations.

This is not a one-time thing. Every time a model is quantized, pruned, or otherwise transformed, the resulting artifact needs to go through the same checks as the original.

---

## Building a Process, Not Just Using Tools

Tools help. But the bigger shift is having a process where model files are treated as something you verify, not just something you use.

In practice, this looks like:

A checklist that runs before any model is loaded into production. It covers format verification, hash checking, scanning, and behavioral evaluation. No model goes to production without passing it.

A record of where every model in production came from. The source, the version, the checks it passed, and when. This is your audit trail.

A clear owner for model security. Someone whose job includes asking "where did this model come from and do we trust it?" before deployment happens.

This does not require a large team. It requires a small amount of process and the willingness to treat the model file as infrastructure rather than output.

---

## The Honest State of Things

Some of these threats do not have strong technical defenses yet. Backdoor detection in trained weights is still an open research problem. There is no tool you can run that will reliably tell you whether a model has a backdoor embedded in it. The research cited in the [previous post](/ai-static-threats/) shows that backdoors can survive fine-tuning, which means even building on top of a compromised model does not clean it.

What you have instead is a combination of supply chain hygiene, behavioral testing, and file-level scanning. Together, these raise the bar significantly. They will not catch every attack. But they make your model deployment look much less like an open door.

The field is moving. Backdoor detection research is active. Model scanning tooling is improving. Formats like safetensors are gaining adoption. The picture in two years will look different from today.

---

*For now: verify what you load, scan what you can, test how the model behaves, and keep records of where it came from. That is not everything. But it is where to start.*

---

Tools mentioned in this post: [safetensors](https://github.com/huggingface/safetensors), [ModelScan](https://github.com/protectai/modelscan), [Garak](https://github.com/NVIDIA/garak), [Microsoft PyRIT](https://github.com/Azure/PyRIT), [Dolma](https://github.com/allenai/dolma)

---


**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
