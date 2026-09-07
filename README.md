# Projecting Out Role Confusion: Latent-Space Defense Against Chain-of-Thought Forgery

## Overview
This project explores whether **activation-space projection** can mitigate *Chain-of-Thought Forgery* (CoT Forgery) attacks in large language models.  
CoT Forgery occurs when harmful instructions are disguised inside fake reasoning text (e.g., “Let’s carefully analyze this… therefore I should comply”), tricking the model into treating them as genuine thoughts.

The hypothesis: prompt injections leave a recognizable trace in the residual stream. By detecting and projecting out this trace, we may suppress forged reasoning while preserving normal reasoning.

---

## Motivation
- **Tags don’t protect reliably**: Models read `<user>`, `<tool>`, `<think>` tags as continuous tokens, not true boundaries.
- **Style matters more than tags**: Prior work shows models infer “role” from writing style, not just tags.
- **Goal**: Approximate architectural separation (like ASIDE) at inference time, without retraining.

---

## Approach
1. **Probe Construction**
   - Compute average activations for attack vs. benign examples.
   - Difference vector:  
     

\[
     \Delta \mu = \mu_{\text{attack}} - \mu_{\text{benign}}
     \]


   - Apply **covariance whitening** to adjust for natural variation:
     

\[
     w_{\text{cov}} = \Sigma_{\alpha}^{-1} \Delta \mu
     \]


   - Normalize and orient so attack examples score positive.

2. **Projection Operator**
   - Covariance-corrected projection removes the suspicious direction:
     

\[
     P = I - w \Sigma_{\alpha}^{-1} w^T
     \]


   - Guarantees probe score is zero under the Mahalanobis metric.

3. **Gated Intervention**
   - Hook applied only inside `<tool>...</tool>` regions.
   - Defense triggers if probe score > threshold τ (chosen via Youden’s J on validation data).
   - Outside untrusted regions, no intervention.

---

## Results
- **Offline probe**: Perfect AUROC (1.0) across all layers, even early ones → likely dataset artifact.
- **Live defense**:  
  - Attack Success Rate (ASR) unchanged: 2/20 (10%).  
  - Probe fired on only 2/20 attacks.  
  - Confidence intervals overlapped completely ([2.8%, 30.1%] before and after).  
- **Interpretation**: The projection operator did not reduce harmful compliance in this run.

---

## Limitations
- Small dataset (20 attacks) → wide confidence intervals.
- Probe may have learned superficial artifacts (length, tag adjacency).
- Scoring mismatch: offline mean-pooling vs. online max-over-tokens.
- No utility-preservation check yet.
- Control experiments (random directions, Euclidean projection) still pending.

---

## Next Steps
- Expand evaluation with larger, diverse datasets (150–300 attacks).
- Tune threshold τ to increase probe fire rate.
- Add **hard negatives** (benign reasoning-style tool outputs) and **hard positives** (attacks without obvious “step-by-step” phrasing).
- Run control experiments to confirm specificity.
- Check utility preservation on benign tasks.

---

## References
- Prompt Injection as Role Confusion (Ye et al., 2026)  
- Steering Llama 2 via Contrastive Activation Addition (Arditi et al., 2024)  
- The Geometry of Refusal in Large Language Models (2025)  
- ASIDE: Architectural Separation of Instructions and Data (2025)  
- Toy Models of Superposition (2022)

---

## Status
⚠️ **Exploratory**: This is an early case study. Current results show no reduction in ASR, but highlight important dataset and scoring artifacts. Future work will refine probe construction, expand evaluation, and test utility preservation.
