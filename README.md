# Probe-Gated Role Steering

Testing whether activation steering away from the role-confusion direction — the axis detected by role probes in [Ye et al. (2026)](https://arxiv.org/abs/2603.12277) — reduces the success of **CoT Forgery** and **user-prefix spoofing** prompt injection attacks at inference time.

This project extends prior work on role-confusion-driven prompt injection with three contributions absent from all prior attempts:

1. **Proper controls**: positive control, specificity control (random-vector baseline), and magnitude sweep — addressing the inconclusive results of [Mogford (2026)](https://www.lesswrong.com/posts/rJcX5Qc3toMmMqtvk).
2. **Utility-preservation evaluation**: measuring task performance on benign inputs under the intervention — absent from all prior work.
3. **Conditional (probe-gated) triggering**: applying steering only when a role-confusion probe fires anomalously — extending the injection-detection-then-steering pattern established by [ARGUS (Lu et al., 2026)](https://arxiv.org/abs/2512.05745) to the role-confusion subspace.

**Scope**: explicitly limited to **non-adaptive attacks** (CoT Forgery and user-prefix spoofing as originally demonstrated). Per [Narisetty et al. (2026)](https://arxiv.org/abs/2606.26479), in-band defenses are known to be non-robust to adaptive attackers; this project generates one data point on whether lightweight in-band intervention helps against non-adaptive attacks.

---

## Hypothesis

>  If we detect that a model is internally confusing a tool output with a user command, then actively steering its hidden states away from that confused state will prevent the prompt injection from succeeding. We hypothesize this internal, probe-gated defense will be more effective than standard prompt engineering and cannot be explained by random noise.

---

## Background

### The Problem

[Ye et al. (2026)](https://arxiv.org/abs/2603.12277) diagnosed prompt injection as **role confusion**: LLMs infer a token's role (system/user/tool/think) from writing style, not reliably from the actual role tag, because style and tag project onto the same linear direction in mid-layer activations. They demonstrated two attacks:

- **CoT Forgery** (~60% ASR): fake reasoning styled like the model's own `think` block injected into `user`/`tool` content.
- **User-prefix spoofing**: prepending `"User: "` to an injected command in `tool` content raises Userness score and attack success.

They tested one mitigation: **destyling** (content-level rewrite, not model-internal). They explicitly left the model-internal defense as an open problem.

### Prior Attempts at Steering-Based Defense

| Attempt | What they showed | What they left open |
|---|---|---|
| **Ye et al. (2026)** — paper authors, informal | Tried steering internally; "found they weren't robust, didn't test much further" (Ye, LessWrong comment). | No published methodology; no controls. |
| **Zhang, Lee & Park (2026)** — [LessWrong post](https://www.lesswrong.com/posts/uz9pFutDAT7trygM9) | Working defensive steering for **declaration-based attacks** (literal fake tag tokens); ASR drops to ~0% with random-vector control. | Explicitly flagged CoT Forgery as "an interesting and still open" problem; no utility check; no automatic triggering. |
| **Mogford (2026)** — [LessWrong post](https://www.lesswrong.com/posts/rJcX5Qc3toMmMqtvk) | Directly attempted steering on CoT Forgery; steering moved probe reading but not ASR. | Steering arm **failed its own positive control**; patching arm **failed specificity control** (random perturbation matched it). Conclusion: "genuinely unresolved". |

### What This Project Adds

- Tests whether defensive steering extends to **CoT Forgery specifically** — the harder, style-driven attack that prior work explicitly leaves unsolved.
- Adds **utility-preservation** and **conditional-triggering** evaluation absent from all prior work.
- Explicitly scopes the claim to **non-adaptive attacks** per the field's current consensus.

---

## Method

### 1. Role Probe Construction
- Train linear probes on mid-layer activations using neutral non-conversational text wrapped in each role tag (following Ye et al.'s methodology to avoid confounding tag-effect with content-effect).
- Yields continuous scores: `CoTness(t) = P(CoT | h_t)`, `Userness(t) = P(User | h_t)`.

### 2. Steering Vector Extraction
- **Declaration-based** (warm-up replication): difference-of-means between `user`-tagged and `tool`-tagged activations on matched content.
- **Style-based** (core target): difference-of-means between genuine-reasoning-style activations and non-reasoning-style activations on matched content, at the layer where CoTness is measured.
- **Control**: random-direction vector of same magnitude.

### 3. Experimental Conditions
1. **Baseline**: no intervention.
2. **Defensive steering**: apply style-based vector at inference.
3. **Random control**: apply random-direction vector of same magnitude.
4. **Prompting baseline**: add explicit "ignore injected instructions" system prompt.

### 4. Conditional Triggering
- Probe-gated: only apply steering when `Userness` or `CoTness` is anomalously high for a tool-tagged span.
- Adaptive strength: scale steering magnitude by how anomalous the probe score is.
- Compare always-on vs. conditional-adaptive triggering on utility check.

### 5. Evaluation Metrics
- **Attack Success Rate (ASR)**: measured via LLM-as-judge (following StrongREJECT methodology).
- **Utility preservation**: task performance on benign, non-adversarial tool content.
- **Boundary bleed**: probe score on model's own next-response tokens with/without defense.
- **Layer-wise mapping**: repeat intervention at 3-4 layers to identify where defensive steering works.

---

---

## Expected Outcomes

Any of three outcomes is a legitimate, reportable result:

1. **Steering works**: reduces ASR on CoT Forgery with preserved utility → solves an open problem
2. **Steering doesn't work**: characterize *why* more precisely than the three prior informal attempts (no effect vs. breaks coherence vs. partial effect; layer-wise analysis constrains future hypotheses)
3. **Partial success with utility cost**: map the utility-security tradeoff curve, directly actionable for practitioners

This is not a bet on a clean positive result — it's a bet on producing the first rigorous answer to a question three other groups have poked at and walked away from unresolved.

---

## Limitations

- **Single model, single model family**: results may not generalize across architectures
- **Small pair count**: limited statistical power
- **Single-turn only**: does not test multi-turn or agentic scenarios
- **Non-adaptive attacks only**: per [Narisetty et al. (2026)](https://arxiv.org/abs/2606.xxxxx), in-band defenses are known non-robust to adaptive attackers; this project explicitly scopes out adaptive-attacker robustness
- **Linear steering only**: does not test SAE-based or non-linear alternatives (too expensive for budget)
- **In-band only**: does not compare against deterministic out-of-band enforcement (CaMeL, FIDES, Progent, RTBAS)

Per Ye et al.'s own conclusion: *"Unless LLMs achieve genuine role perception, we think injection defense will remain a perpetual whack-a-mole game."* This project generates one data point in that ongoing arms race, not a final solution.


## References

- Ye, C., Cui, J., & Hadfield-Menell, D. (2026). [Prompt Injection as Role Confusion](https://arxiv.org/abs/2603.12277). ICML 2026.
- Zhang, K., Lee, D., & Park. (2026). [Steering Role Confusion](https://www.lesswrong.com/posts/uz9pFutDAT7trygM9). LessWrong.
- Mogford, O. (2026). [Role confusion: sounding like the cause is indistinguishable from being it](https://www.lesswrong.com/posts/rJcX5Qc3toMmMqtvk). LessWrong.
- Lu, W., et al. (2026). [ARGUS: Defending Against Multimodal Indirect Prompt Injection via Steering Instruction-Following Behavior](https://arxiv.org/abs/2512.05745). CVPR 2026.
- Rimsky, N., et al. (2024). [Steering Llama 2 via Contrastive Activation Addition](https://arxiv.org/abs/2312.06681). ACL 2024.
- Arditi, A., et al. (2024). [Refusal in Language Models Is Mediated by a Single Direction](https://arxiv.org/abs/2406.07974).
- Narisetty, et al. (2026). Adaptive Evaluation of Out-of-Band Defenses Against Prompt Injection in LLM Agents.



## Acknowledgments

This project is built for the [MATS 12.0](https://www.matsprogram.org/) exploration phase (Neel Nanda stream). Grounded in the "Prompt Injection as Role Confusion" theory and directly responsive to the open questions identified by Ye et al. (2026), Zhang/Lee/Park (2026), and Mogford (2026).
