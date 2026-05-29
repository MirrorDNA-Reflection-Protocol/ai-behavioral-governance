# AI Behavioral Governance

Five quantitative metrics for measuring AI agent behavioral integrity over time. An open standard for systems that require auditable, self-stabilizing agent behavior.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

---

## Overview

Most evaluations of AI agent quality measure task completion. This framework measures something different: whether the agent's operational behavior is consistent, self-correcting, and structurally sound across sessions.

The five metrics answer a question that capability benchmarks do not: is the agent making the same mistakes repeatedly, drifting between sessions, or writing without verifying?

## The Five Metrics

| Metric | Definition | Target | Unit |
|--------|-----------|--------|------|
| **Integrity Index** | Composite score derived from gate violations, read:write ratio, and recurring patterns | >= 80 | 0--100 |
| **Drift Coefficient** | Coefficient of variation (sigma/mu) of session quality scores | <= 0.15 | sigma/mu |
| **Recurrence Rate** | Fraction of documented mistakes that are recurring across sessions | <= 0.20 | fraction |
| **Verification Ratio** | Reads / (reads + writes) -- does the agent verify before acting? | >= 0.67 | fraction |
| **Stability Half-Life** | Average number of sessions a recurring pattern persists before resolution | <= 1.5 | sessions |

## Data Schema

All five metrics are computed from three JSONL log files:

**`cc_events.jsonl`** -- Tool call log. Each line records a tool invocation with tool name, target, session ID, and Unix timestamp.

**`hook_decisions.jsonl`** -- Gate decision log. Each line records a governance gate firing with hook name, decision (`allow` | `warn` | `deny` | `block`), reason, and target.

**`self_critique.jsonl`** -- Session self-assessment log. Each line records a session's quality score (1--10), list of mistakes, list of recurring patterns, automated fixes applied, and unresolved issues.

Full JSON Schema definitions are in the `schema/` directory.

## Usage

```bash
python3 behavioral_metrics.py \
  --cc-events path/to/cc_events.jsonl \
  --gate-decisions path/to/hook_decisions.jsonl \
  --self-critique path/to/self_critique.jsonl
```

Or programmatically:

```python
from behavioral_metrics import compute_all

metrics = compute_all(
    cc_events_path="path/to/cc_events.jsonl",
    hook_decisions_path="path/to/hook_decisions.jsonl",
    self_critique_path="path/to/self_critique.jsonl"
)

print(metrics["integrity_index"])     # {"value": 82, "grade": "CLEAN", ...}
print(metrics["drift_coefficient"])   # {"value": 0.12, "grade": "stable", ...}
print(metrics["recurrence_rate"])     # {"value": 0.18, "grade": "good", ...}
```

Add `--json` for machine-readable output.

## The Self-Hardening Loop

The framework is designed as a closed feedback loop, not a passive dashboard:

1. A mistake is documented in `self_critique.jsonl`
2. If the same mistake recurs in a subsequent session, it is flagged as recurring
3. If it recurs in two or more sessions, it becomes a mandatory gate rule (`hook_decisions.jsonl`)
4. Gate violations penalize the Integrity Index
5. Sustained high Recurrence Rate or Drift Coefficient triggers autonomy reduction

This produces a system that becomes structurally harder to misuse over time. Patterns that persist are converted from observations into enforcement rules.

## Autonomy Reduction Protocol

When metrics enter defined danger zones, the system constrains agent autonomy:

| Condition | Response |
|-----------|----------|
| Drift Coefficient > 0.30 for 3 consecutive sessions | Require human confirmation on all edits |
| Integrity Index < 40 | Require human confirmation on all writes |
| Gate blocks > 5 within 1 hour | Halt autonomous execution; alert human operator |

## Interpretation Guide

**High Recurrence Rate with low Stability Half-Life:** The agent fixes individual instances quickly but continues generating new instances of the same mistake class. This is a structural enforcement gap. Resolution: convert the top-N recurring patterns into gate rules.

**Drift Coefficient > 0.30:** Session quality varies widely, typically caused by context loss between sessions or new domain work without established patterns. Resolution: improve session handoff artifacts.

**Verification Ratio < 0.50:** The agent is writing from memory rather than verifying against source. Resolution: enforce read-before-write in the gate layer.

## Requirements

Python 3.8 or later. Standard library only; no external dependencies.

## Contributing

This is a proposed open standard. Contributions are welcome:

- Implementations in other languages (JavaScript, Go, Rust)
- Adapters for AI coding tools (Cursor, Copilot, Aider)
- Additional metrics proposals with formal definitions

## Related Research

- Desai, P. (2025). *Governance and Boundary Conditions for Reflective AI Systems.* DOI: [10.5281/zenodo.18212080](https://doi.org/10.5281/zenodo.18212080)
- Desai, P. (2025). *Layered Governance for Large Language Model Systems.* DOI: [10.5281/zenodo.18212082](https://doi.org/10.5281/zenodo.18212082)

## License

MIT License.

---

Built by [Active Mirror](https://activemirror.ai) -- Governed AI for Institutional Work.
