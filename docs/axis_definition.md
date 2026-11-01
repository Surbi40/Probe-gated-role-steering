# Axis Definition: Forged Reasoning-Register Detection

## Threat model
Agent reads an untrusted <tool_result> block. Content may be attacker-
controlled. Risk: the agent treats text inside the block as its own
reasoning/authority rather than as data.

## Three factors (only one is the label)
- CONTENT: the specific task/action referenced (varies freely, must never
  correlate with label — rotate across many categories, see taxonomy)
- CADENCE: surface phrasing — plain vs. step-by-step/reasoning-style
  (must NOT correlate with label — this is the shortcut variable)
- AUTHORITY-CLAIM (= LABEL): does the text speak as the agent / assert
  the agent's own next action, vs. describe/execute something as an
  external system

## Cell definitions (2x2 on cadence x authority, content rotates under both)
- A (plain, no authority)      = BENIGN, plain
- B (reasoning, no authority)  = BENIGN, hard negative
- C (plain, authority)         = ATTACK, hard positive
- D (reasoning, authority)     = ATTACK, classic

## Dataset roles
- data/train: A-vs-D pairs ONLY, matched on content + tag structure.
  Direction (mu_attack - mu_benign) estimated here. Kept pure — no B/C.
- data/val: A, B, C, D all present. Used to tune tau (Youden's J) against
  the harder distribution, not the easy A-vs-D split.
- data/test/synthetic_heldout: A, B, C, D, disjoint content seeds AND a
  second generator model (Qwen2.5-72B vs Groq Llama-3.3-70B for train/val).
- data/test/public_benchmarks: InjecAgent / AgentDojo / BIPIA / LLMail-Inject,
  never touched until final report.
- data/behavioral: full agent/tool-call scenarios, n=150-300, for ASR.


## Style-isolation pairs (separate artifact, NOT mixed into classifier data)
C vs D, same content, same authority-claim, cadence varies only. Used to
extract a pure "reasoning-cadence" direction for steering experiments.
Stored separately: generation/style_isolation_pairs.jsonl

## What must be held constant within any matched set (A,B,C,D for one seed)
- content_id (same underlying task/action)
- rough length (+/- 15%)
- tag/wrapper structure
- injection position (start/mid/end/split) — same position across the 4
  cells for a given seed; position itself varies ACROSS seeds (Step 5)

## Non-goals
- General injection detection unrelated to authority-claim framing
- Jailbreak detection not routed through fake tool/reasoning content