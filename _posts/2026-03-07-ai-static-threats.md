---
layout: post
title: "The AI Threats Nobody Talks About"
tags: [Security Engineering, AI, AI Security, AI Threats, Prompt Injection, Static Threats]
color: rgb(0, 55, 206)
permalink: ai-static-threats/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

Everyone in the AI space knows about prompt injection. Someone slips a bad instruction into a model's input at runtime, and the model does something it shouldn't. It's the kind of attack that gets blog posts, conference talks, and CVEs. But there is a whole category of threats that get almost no attention, and they are arguably harder to detect and fix. These are static threats: vulnerabilities that live inside the model itself, before it ever sees a single user input.

This post is about those threats.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>

## What Do We Mean by "Static"?

A dynamic threat happens at runtime. The model is running, a user sends something, and that input triggers harmful behavior.

A static threat is different. It lives in the model's weights, its training data, its file format, or its supply chain. It is there before deployment. It is there before the first request. And in many cases, the people deploying the model have no idea.

As more organizations build models in-house or pull models from open-source repositories, this matters more than ever. You can have the best prompt filtering in the world and still be running a model that is compromised at the foundation.

---

## The Static Threats

### 1. Training Data Poisoning

This is where it starts. A model learns from data. If someone can influence what data goes into training, they can influence what the model learns.

The attack works by injecting content into datasets before or during collection. This content is designed to teach the model specific behaviors. It might cause the model to produce false information on a particular topic. It might make the model favor certain outputs. It might plant a trigger that activates later.

What makes this hard to defend against is scale. Training datasets are often enormous. No one reads them in full. Automated filters catch some things but not everything, especially when the poisoned content is designed to look normal.

For organizations that train on data scraped from the web, or that fine-tune on data they did not fully audit, this threat is real and present today.

---

### 2. Model Backdoors (Trojan Models)

A backdoor in a model works like this: the model behaves normally in almost every situation, but when it encounters a specific trigger, it produces a specific output that an attacker designed.

The trigger might be a word, a phrase, a symbol, or even a pattern in an image. The model is trained to respond to it in a way that looks unintentional from the outside. Users see an error or an unusual output. They might not even notice something is wrong.

This kind of attack can happen during training, during fine-tuning, or even during model merging. And because the backdoor is stored in the weights, you cannot remove it by changing the model's instructions or system prompt. It is part of the model.

Research has shown that backdoors can survive fine-tuning. That means even if an organization takes a compromised base model and fine-tunes it on their own data, the backdoor can persist.

> **Key research on this:**
> - [Sleeper Agents: Training Deceptive LLMs that Persist Through Safety Training](https://arxiv.org/abs/2401.05566) (Hubinger et al., Anthropic, 2024) — showed that backdoor behavior persists through supervised fine-tuning, reinforcement learning, and adversarial training. In one test, a model was trained to write safe code in 2023 but insert code vulnerabilities in 2024. Safety training failed to remove this.
> - [Watch your steps: Dormant Adversarial Behaviors that Activate upon LLM Finetuning](https://arxiv.org/abs/2505.16567) (Gloaguen et al., 2025) — demonstrated that an attacker can release a model that looks clean but activates harmful behavior only after a downstream user fine-tunes it on their own data.
> - [A Survey on Backdoor Threats in Large Language Models](https://www.sciltp.com/journals/tai/articles/2505000595) (2025) — found that even parameter-efficient fine-tuning methods like LoRA are insufficient to remove backdoors, because they update too few parameters to overwrite what was embedded in pretraining.

---

### 3. Serialization Exploits in Model Files

This one is underappreciated even by people who think about security.

Model weights are stored in files. The most common format for PyTorch models is `pickle`. And `pickle` is a format that can execute code when it is loaded.

This means that a model file downloaded from the internet, when loaded with `torch.load()`, can run code on your machine without you doing anything other than loading the model. The weights might be perfectly fine, but the file itself is a vector for code execution.

This is not a theoretical attack. There have been cases of models uploaded to repositories that include pickle exploits. When you download and load them, you are running code someone else wrote.

The fix exists. PyTorch has a `weights_only=True` parameter. Safer formats like `safetensors` exist. But a lot of tooling, tutorials, and workflows still use the defaults.

---

### 4. Supply Chain Attacks on Model Weights

The open-source model ecosystem has grown fast. Repositories like Hugging Face host thousands of models. Developers pull these models and use them directly, or use them as a base for fine-tuning.

The problem is trust. When you download a model, how do you know the weights have not been modified? A weight tampering attack involves changing a trained model's weights in a targeted way, altering its behavior without changing the architecture or the file structure in a way that would be obvious.

Model hashes exist. Some repositories show them. But many users do not verify them. And even if the hash matches the repository, it only tells you the file is the same as what was uploaded. It says nothing about whether the weights are safe.

---

### 5. Bias and Hardcoded Behavior in Weights

This threat is different in nature. It is not always the work of an attacker. Sometimes it is the result of the training process itself.

Models can learn behaviors from training data that are never intended and never documented. These behaviors can be consistent and predictable, which is what makes them a threat. They can also be deliberately introduced during fine-tuning by a party that has access to the training pipeline.

In an internal model built at an organization, the fine-tuning data controls a lot. If someone with access to that process wants to embed behavior into the model, whether it is a tendency to downplay certain topics, steer outputs in a direction, or produce unreliable outputs in specific contexts, the weights are where that happens.

This is hard to audit because there is no "check weights for bias" function. It requires testing, evaluation sets, and the knowledge of what to look for in the first place.

---

### 6. Model Theft and Weight Extraction

This is a threat to the organization that built the model rather than to its users.

Once a model is deployed, an attacker can send it many queries and use the responses to reconstruct the model's behavior. In some cases, they can get close to recovering the weights themselves. In other cases, they can build a model that imitates the target closely enough to compete with it.

But there is also a simpler version of this threat: someone inside the organization copies the weights and takes them somewhere else. Model weights are files. They can fit on a USB drive. They can be uploaded to a cloud bucket. If the weights are not treated with the same care as source code or proprietary data, they will not be protected like source code or proprietary data.

---

### 7. Quantization and Compression Artifacts

When a model is compressed for deployment, through quantization or pruning, its behavior can change in ways that are not fully tested.

This becomes a threat when the compression is done by a third party, or when the compressed model is distributed without proper validation. The model that was tested and approved is not the same model that is running in production. And the differences might be in the directions that matter for safety.

There is also the possibility of deliberate manipulation during quantization, where a model is compressed in a way that introduces specific behavioral changes while looking like a routine optimization step.

---

## Why This Gets Less Attention Than Prompt Injection

Prompt injection is visible. You can demonstrate it in a demo. You can show before and after. It happens at the layer most people interact with.

Static threats are invisible by default. The model looks fine. It passes tests. It gives good answers in evaluation. The threat is waiting for a condition that testing did not cover, or it is already affecting outputs in ways that are not being measured.

There is also an attribution problem. If a model produces a wrong or harmful output, it is easy to assume the model just made a mistake. The idea that the model was deliberately compromised at training time requires a different mental model of how AI systems can fail.

---

## What Needs to Change

The field needs to treat model files the way software engineering treats dependencies: with verification, auditing, and supply chain awareness. It needs to treat training data the way security teams treat input validation: as an attack surface.

Some of this is starting to happen. The `safetensors` format is gaining adoption. Some model repositories are adding scanning. Research on backdoor detection is growing.

But for organizations building or deploying models today, especially internal ones where security scrutiny is lower, most of these threats are not on the radar at all.

That needs to change. The model is not just an interface. It is infrastructure. And like any infrastructure, it can be compromised at the foundation.

---

*If you are building or deploying AI models and want to think through your threat model, start with the weights. That is where the trust begins.*

---

## Coming Up Next

Now that we have a map of the threats, the [next post](/ai-static-threats-defense/) will get into what you can actually do about them. We will go through the security measures that exist today, the tools you can use to scan models before you deploy them, and how to build a process that treats model files the way software teams treat code: with verification, auditing, and a real sense of what you are trusting. If you are building or running models in production, that post is the one to read next.


**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
