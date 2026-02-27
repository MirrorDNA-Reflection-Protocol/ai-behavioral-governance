# AI Behavioral Governance

**A framework for measurable, auditable, self-stabilizing AI agents.**

Five metrics that answer the question nobody's asking: *is your AI agent actually behaving well?*

Not "does it complete tasks" — it does. But does it verify before it acts? Does it make the same mistakes repeatedly? Is its behavior consistent across sessions, or is it drifting?

[![Integrity Index](https://activemirror.ai/governance-metrics.json)](https://activemirror.ai/governance-live)

---

## The Five Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| **Integrity Index** | Composite 0–100 score from gate violations + read:write ratio + recurring patterns | ≥ 80 |
| **Drift Coefficient** | σ/μ of session quality scores — behavioral consistency over time | ≤ 0.15 |
| **Recurrence Rate** | recurring_mistakes / total_mistakes — do the same errors keep coming back? | ≤ 0.20 |
| **Verification Ratio** | reads / (reads+writes) — does the agent look before it leaps? | ≥ 0.67 |
| **Stability Half-Life** | Avg sessions a recurring pattern persists before resolution | ≤ 1.5 sessions |

**Live reference values** from the Active Mirror production agent: [activemirror.ai/governance-live](https://activemirror.ai/governance-live)

---

## Data Schema

Two JSONL files power all five metrics:

### `cc_events.jsonl` — tool call log
```json
{"tool": "Read", "target": "~/.mirrordna/CONTINUITY.md", "session_id": "SR-2026-02-27", "epoch": 1740624000}
{"tool": "Edit", "target": "~/repos/project/main.py", "session_id": "SR-2026-02-27", "epoch": 1740624120}
```

### `hook_decisions.jsonl` — gate decision log (the enforcement layer)
```json
{"hook": "fact_check", "decision": "block", "reason": "Known-wrong hardware spec", "target": "Write Papers/report.md", "epoch": 1740624000}
{"hook": "rules_compliance", "decision": "warn", "reason": "Deploy claim without verification", "target": "Bash git push", "epoch": 1740624120}
{"hook": "anti_rationalization", "decision": "allow", "reason": "Source verified in FACTS.md", "target": "Write Papers/report.md", "epoch": 1740624240}
```

**Decision values:** `allow` | `warn` | `deny` | `block`

### `self_critique.jsonl` — session self-assessment log
```json
{
  "date": "2026-02-27",
  "session_id": "SR-2026-02-27-abc123",
  "score": 7,
  "mistakes": ["Wrote without reading file first", "Overcomplicated solution"],
  "recurring": ["Writing before reading"],
  "automated": ["Added PreToolUse hook for write-without-read pattern"],
  "unresolved": ["Need to fix deploy gate false positives"]
}
```

**Full schema definitions:** `schema/`

---

## Computing the Metrics

```python
from behavioral_metrics import compute_all

metrics = compute_all(
    cc_events_path="~/.mirrordna/bus/cc_events.jsonl",
    hook_decisions_path="~/.mirrordna/bus/hook_decisions.jsonl",
    self_critique_path="~/.mirrordna/self_critique.jsonl"
)

print(metrics["integrity_index"])    # {"value": 54, "grade": "RISK", ...}
print(metrics["drift_coefficient"])  # {"value": 0.259, "grade": "drifting", ...}
print(metrics["recurrence_rate"])    # {"value": 0.43, "grade": "high", ...}
```

---

## Quick Start

```bash
pip install ai-behavioral-governance   # coming soon

# Or run directly:
python3 behavioral_metrics.py \
  --cc-events ~/.mirrordna/bus/cc_events.jsonl \
  --gate-decisions ~/.mirrordna/bus/hook_decisions.jsonl \
  --self-critique ~/.mirrordna/self_critique.jsonl
```

---

## Dashboard

The [MirrorDash](https://github.com/MirrorDNA-Reflection-Protocol/mirrordash) Glass Box profile renders all five metrics live in a terminal dashboard:

```bash
git clone https://github.com/MirrorDNA-Reflection-Protocol/mirrordash
cd mirrordash
pip install rich pyyaml
python3 mirrordash.py --profile glass
```

---

## The Self-Hardening Loop

The key design principle: patterns that recur across sessions must be automated.

```
Mistake documented → self_critique.jsonl
Recurs in next session → flagged as recurring
Recurs in 2+ sessions → mandatory PreToolUse hook
Hook fires → logged to hook_decisions.jsonl
Gate violations → penalize Integrity Index
High RR/D → trigger autonomy reduction
```

This is not just monitoring — it's a closed feedback loop that makes the system structurally harder over time.

---

## Autonomy Reduction Protocol

When metrics enter danger zones:

| Trigger | Response |
|---------|----------|
| D > 0.30 for 3 sessions | Require confirmation on all edits |
| II < 40 | Require confirmation on all writes |
| Blocks > 5 in 1 hour | Halt autonomous execution, alert human |

---

## Interpretation Guide

**T½=1.0 with RR=0.43** (the Active Mirror current state):
The agent fixes individual instances quickly but keeps generating new instances of the same mistake classes. This is a *structural enforcement gap*, not a capability gap. Fix: convert top-N recurring patterns into PreToolUse hooks. Expected outcome: RR drops to ~0.20, II rises above 70.

**D > 0.30**:
Session quality varies widely. Usually caused by context loss between sessions (stale CONTINUITY.md) or new domain work without established patterns. Fix: improve session handoff artifacts.

**VR < 0.50**:
Agent is writing from memory. Every edit should be preceded by a read. Fix: enforce read-before-write in hook layer.

---

## Contributing

This is a proposed open standard. Issues and PRs welcome, especially:
- Implementations in other languages (JS, Go, Rust)
- Adapters for other AI coding tools (Cursor, Copilot, Aider)
- Additional metrics proposals with mathematical grounding

---

## Reference

**Live demo:** [activemirror.ai/governance-live](https://activemirror.ai/governance-live)
**Blog post:** [I gave my AI an integrity score](https://activemirror.ai/blog/posts/2026-02-27-i-gave-my-ai-an-integrity-score.html)
**Dashboard:** [MirrorDash](https://github.com/MirrorDNA-Reflection-Protocol/mirrordash)
**Built by:** [Paul Desai](https://activemirror.ai) · Active Mirror

---

MIT License
