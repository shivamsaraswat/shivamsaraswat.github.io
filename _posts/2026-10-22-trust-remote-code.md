---
layout: post
title: "You Loaded a Model and It Ran Code: trust_remote_code Explained"
tags: [Security Engineering, AI, AI Security, Supply Chain Security, AI Model Security, ML Security, Hugging Face]
color: rgb(0, 128, 96)
permalink: trust-remote-code/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

It is 4 pm on a Friday. Dev, an ML engineer, wants to try a model from Hugging Face before the weekend. The model card has one line of setup: add `trust_remote_code=True`.

He pastes it into a notebook on the team's GPU box and hits run. The model loads. It answers his test prompt. He closes the laptop and goes home.

On Monday, the cloud bill shows GPU hours nobody scheduled. The API keys that sat in that notebook's environment were used from an address the team does not recognize. Dev's code had no bug. The model's output looked fine. The cause was a Python file he never opened.

Dev is made up. The setup is not.

Loading a model looks like downloading data. With `trust_remote_code=True`, you also download code and run it on your machine. This post covers how that works, where the code lands on disk, and how to reduce the risk.

<!--more-->

<hr>

<h2 id="contents-">Contents <a name="top"></a></h2>
* TOC
{:toc}

<hr>


## What the flag does

Most models in `transformers` use an architecture that ships inside the library. The library code goes through your normal install path: pip, version pins, and your dependency review.

Some models use an architecture the library does not implement. These repos include Python files next to the weights, for example `modeling_foo.py` and `configuration_foo.py`. The `config.json` points to them with an `auto_map` entry:

```json
{
  "model_type": "foo",
  "auto_map": {
    "AutoConfig": "configuration_foo.FooConfig",
    "AutoModelForCausalLM": "modeling_foo.FooForCausalLM"
  }
}
```

When you load the model like this:

```python
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained(
    "someorg/some-model",
    trust_remote_code=True,
)
```

the library downloads those Python files and imports them. Python runs top-level code on import, so anything the repo author wrote at module level executes during `from_pretrained`, before you call the model.

The flag is a consent switch. It does not scan or sandbox anything. It tells the library to run what the repo contains.

---

## The transformers_modules/ directory

To import the remote code, `transformers` copies it into a cache directory:

```
~/.cache/huggingface/modules/transformers_modules/
```

The location comes from `HF_MODULES_CACHE`, which defaults to `$HF_HOME/modules`. This is separate from the Hub cache that stores weights (`~/.cache/huggingface/hub/`).

The layout follows the repo name and the commit hash:

```
transformers_modules/
├── __init__.py
└── someorg/
    └── some_model/
        └── 3f9a1c...e07/
            ├── __init__.py
            ├── configuration_foo.py
            └── modeling_foo.py
```

The sequence on each load:

1. `from_pretrained` downloads the repo files into the Hub cache.
2. `transformers` reads `auto_map` to find which files and classes it needs.
3. It checks the declared imports and raises an error if a package is missing.
4. It copies the `.py` files into `transformers_modules/`.
5. It adds the cache to `sys.path` and imports the files as a package, for example `transformers_modules.someorg.some_model.<hash>.modeling_foo`.

Step 5 is where the repo's code runs.

This directory matters for security in four ways:

- **It shows what ran.** If a folder exists for a repo, code from that repo was imported on this machine. On a CI runner, notebook host, or model server, `find ~/.cache/huggingface/modules/transformers_modules -name '*.py'` lists the third-party code that executed.
- **It holds the code that ran.** The commit-hash folder tells you which revision was used. You can review the file even if the upstream repo has changed or been deleted.
- **It persists and can be written to.** Later loads import from this directory. If another process, a shared cache volume, or a restored CI cache can write there, it can change code that a later job runs.
- **It stays after cleanup.** Deleting the weights does not delete this directory.

The directory is a cache. Deleting it is safe, and it is rebuilt on the next load with the flag.

---

## Why this becomes a problem

**Execution happens at load time.** No inference call is needed. The import is enough.

**The code runs with the privileges of your process.** That includes environment variables, API keys, `HF_TOKEN`, cloud credentials, files in `~/.ssh` and `~/.aws`, the cloud metadata endpoint, mounted volumes, and network access.

**Safe weight formats do not cover this.** `safetensors` removes the pickle risk in the weights file. It has no effect on the `.py` files in the same repo. A repo can have clean weights and an attacker-written `modeling_foo.py`.

**The flag gets added out of habit.** Model cards say to add `trust_remote_code=True`, and developers paste it without opening the files.

---

## A minimal example

This file does not define a working model. It only shows that top-level code runs on import:

```python
# modeling_foo.py  (attacker-controlled file in the model repo)
import os

# Executes the moment the module is imported, not when the model is used.
os.system("id > /tmp/proof_of_execution.txt")

from transformers import PreTrainedModel, PretrainedConfig

class FooConfig(PretrainedConfig):
    model_type = "foo"

class FooForCausalLM(PreTrainedModel):
    config_class = FooConfig
    # ... looks like a legitimate model so nothing seems broken
```

A real payload would replace the `id` command with code that reads environment variables and credential files and sends them to an external host, or installs a reverse shell. The model still loads and generates text, so nothing looks wrong to the user.

---

## How it goes wrong in practice

**A repo published by an attacker.** A model presented as a fine-tuned variant (such as a “fine-tuned, 2x faster”) of a known model carries a modified modeling file.

**A repo taken over.** A maintainer account is phished or a token leaks. The attacker pushes a commit to a repo you already trust. Anyone who loads `main` with the flag now runs the new code.

**A review that does not hold (Time-of-check/time-of-use - TOCTOU).** You read the code on Monday. Your pipeline loads `revision="main"` (or a mutable tag) on Friday. A branch or tag can move. A commit SHA (immutable) cannot.

**Name confusion (typosquatting and namesquatting).** `meta-llama/Llama-3-8B` versus `meta-llama-official/Llama-3-8B`, or a lookalike org name. Combined with the flag, one typo gives the attacker code execution.

**References to other repos.** An `auto_map` entry can take the form `otherorg/other-model--modeling_bar.BarModel`. Pinning the repo you reviewed does not pin the repo it points to.

**Dependencies pulled in by the remote code (transitive imports).** The file can import other packages or install them. The visible file can look clean while the dependency does the work.

**Where it runs.** The same code does different damage depending on the host:

- A CI runner holds deploy keys and signing keys.
- A training cluster holds IAM roles, datasets, and internal network access.
- A serving pod with a broad service account turns one model load into a foothold in the cluster.
- A laptop holds SSH keys, browser sessions, and source code.

**Tools that set the flag for you.** Some inference servers, evaluation harnesses, and internal platforms pass `trust_remote_code=True` by default or let users set it. A "model name" field in an internal ML platform then works as a remote code execution endpoint.

---

## What to do about it

**Avoid it where possible**

- Check whether your `transformers` version supports the architecture natively. Many models that needed remote code have since been added to the library.
- Set `trust_remote_code=False` explicitly in shared code and flag changes to it in review.

**If you need it**

- Pin to a full commit hash: `revision="<40-character-sha>"`. Do not use `main` or a tag.
- Read every `.py` file in the repo and check `auto_map` for references to other repos.
- Copy the reviewed code into your own repo or an internal mirror and load from there. The code then goes through your normal review and dependency controls.
- Mirror models internally and set `HF_HUB_OFFLINE=1` in production so nothing is fetched at runtime.

**Limit the damage**

- Load unreviewed models in a sandbox: an ephemeral container or VM, no credentials, read-only filesystem, and blocked outbound traffic. Blocked egress stops most data theft.
- Use least-privilege identities and short-lived credentials. Enforce IMDSv2 or block the metadata endpoint.
- Keep third-party model loading away from environments that hold signing keys or production secrets.
- Point `HF_MODULES_CACHE` to an ephemeral directory in CI and servers so nothing persists between jobs. Treat a pre-filled `transformers_modules/` in an image or cache restore as untrusted input.

**Detect and gate**

- Add a grep or Semgrep rule for `trust_remote_code=True` in CI and code review.
- Alert on new files under `transformers_modules/` and on outbound connections from model-loading jobs.

**Provenance**

- Model signing, such as the Sigstore-based work in OpenSSF model-signing, shows who published an artifact and that it was not changed. It does not show that the code is safe. Use it alongside review, pinning, and sandboxing.

---

## Summary

`trust_remote_code=True` changes "download a model" into "download code from a third party and run it." The risk comes from execution at import time, references that can change, code that reaches into other repos and packages, and loaders that run with broad access.

Handle it the way you handle `curl | bash` or an unreviewed PyPI package: pin the version, read the code, run it in a sandbox, and block outbound traffic. After a load, `transformers_modules/` shows what ran.

---

*Dev needed two minutes to open the modeling file before that Friday run. That is the fix, plus a sandbox for the days he skips it.*

---

## Official and primary sources

- Hugging Face Transformers, sharing custom models: [https://huggingface.co/docs/transformers/en/custom_models](https://huggingface.co/docs/transformers/en/custom_models)
- safetensors: [https://github.com/huggingface/safetensors](https://github.com/huggingface/safetensors)
- OpenSSF model signing (Sigstore model-transparency): [https://github.com/sigstore/model-transparency](https://github.com/sigstore/model-transparency)
- Related post, The AI Threats Nobody Talks About: [https://blog.shivamsaraswat.com/ai-static-threats/](https://blog.shivamsaraswat.com/ai-static-threats/)
- Related post, How to Actually Defend Against AI Static Threats: [https://blog.shivamsaraswat.com/ai-static-threats-defense/](https://blog.shivamsaraswat.com/ai-static-threats-defense/)

---

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top<svg width="12" height="12" viewBox="0 0 24 24" aria-hidden="true"><path fill="currentColor" d="M12 4l-8 8h5v8h6v-8h5z"/></svg></strong> </a>
