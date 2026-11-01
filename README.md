# Projecting Out Role Confusion

**An exploratory study of activation-space defenses against Chain-of-Thought (CoT) forgery in large language models.**

## Overview

This project investigates whether forged reasoning-style instructions in untrusted tool content can be detected in a language model's internal activations and selectively suppressed using a covariance-aware projection.

The work explores contrastive activation probing and gated inference-time intervention using **Qwen3-1.7B**. The goal is to study a possible latent-space defense against reasoning-style prompt injection without retraining the model.

## Repository Contents

- `notebooks/` — Research implementation and experiments.
- `dataset/` — Datasets constructed for the experiments.
- `docs/` — Detailed research write-up.
- `requirements.txt` — Python dependencies.

## Research Write-up

The full write-up covers the threat model, mathematical formulation, dataset construction, experimental methodology, results, limitations, and planned work.

**[Read the full research write-up (PDF)](docs/Projecting_out_role_confusion.pdf)**

## Current Status

**Exploratory — not yet a demonstrated defense.**

The expanded evaluation shows that the probe carries a measurable detection signal on held-out attack and benign examples. However, the observed false-positive rate is above the intended target, and the effect of the expanded projection intervention on model behavior has not yet been reported. The initial pilot did not reduce attack success.

See the [full write-up](docs/Projecting_out_role_confusion.pdf) for the complete results and discussion.

## Reproducibility

Install the project dependencies with:

```bash
pip install -r requirements.txt
```

Open the notebook in `notebooks/` to inspect the implementation and experiments. Dataset availability and any additional setup requirements are described in the notebook and research write-up.
