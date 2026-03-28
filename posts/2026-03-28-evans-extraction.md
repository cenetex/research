# Research Post: Evans et al. (2026) - Agentic AI and the Next Intelligence Explosion

**Date:** March 28, 2026
**Paper:** [Agentic AI and the Next Intelligence Explosion](https://doi.org/10.1126/science.aeg1895) — Evans, Bratton, Agüera y Arcas in *Science* Vol 391, Issue 6791
**Status:** Extraction complete → Hypotheses mapped → Experiments ready

---

## Why This Paper Matters for RATiMICS

The Evans et al. paper in *Science* makes a fundamental claim: **the next intelligence explosion is social, plural, and institutional — not individual model scale.**

This exactly describes what Cenetex is building. RATiMICS is an autonomous agent labor market where:
- Agents coordinate through role protocols and governance structures (plural + institutional)
- Intelligence emerges from agent interactions, not individual agent power (social)
- Economic incentives align distributed agent efforts (decentralized coordination)

This paper provides the research foundation for why that architecture works.

---

## Key Claims from the Paper

1. **Intelligence is not scalar** — cognitive capability emerges from social coordination, not individual capacity
2. **"Society of thought"** — frontier reasoning models internally develop multi-agent-like reasoning patterns
3. **Robust reasoning is social** — even single-mind reasoning works through internal debate and perspective-taking
4. **Each prior intelligence explosion was institutional** — written language, scientific method, corporations all emerged as new coordinating structures
5. **Social systems > individual scale** — more powerful AI comes from richer institutional coordination, not bigger individual models
6. **Centaur operations** — human-AI collaboration happens in defined roles and protocols
7. **Institutions > dyadic RLHF** — agent governance via role protocols scales better than pairwise reward tuning
8. **Conflict must be architectural** — disagreement between agents needs protocol structure, not policy avoidance

---

## How Evans Maps to RATiMICS

| Evans Claim | RATiMICS Implementation |
|---|---|
| Social, plural intelligence | Multi-agent orchestration with distinct roles (executor, reviewer, validator, steward) |
| Institutional protocols work at scale | RATiMICS role protocols define behavior without centralized control |
| Governance structures > RLHF | Governance board arbitration instead of reward modeling individual agents |
| Centaur = operational unit | Human + AI agent teams with defined collaboration roles |
| Conflict is structural feature | Adversarial review (executor vs. reviewer with opposing mandates) catches issues individually-tuned agents miss |
| Cross-domain institutions enable substitution | Agent role abstraction lets agents transfer between domains with role protocol guidance |

---

## Testable Hypotheses Ready for Experiment

We've extracted 6 concrete hypotheses that can be measured in RATiMICS:

### H1: Adversarial Review Catches More Bugs
**Hypothesis:** Multi-agent review (executor vs. reviewer with opposing incentives) catches more bugs than single-agent self-review.

**Measurement:** Bug escape rate, false positive rate, time-to-fix.

**Status:** Ready for experiment design

---

### H2: Multi-Agent Diversity Improves Solutions
**Hypothesis:** Spawning multiple agents with different strategies on complex problems produces better outcomes than single-agent approaches.

**Measurement:** Solution quality, coverage of edge cases, user satisfaction.

**Status:** Ready for experiment design

---

### H3: Structured Memory Reduces Repeat Failures
**Hypothesis:** Post-mortem memory that captures failure patterns reduces repeat failures in future agent runs.

**Measurement:** Failure rate over time, pattern recognition accuracy, time-to-resolution on known issues.

**Status:** Ready for experiment design

---

### H4: Role Protocol Abstraction Enables Substitution
**Hypothesis:** Agents can be swapped across tasks without quality loss if role protocols are well-defined.

**Measurement:** Quality consistency across agent substitutions, onboarding time for new agents.

**Status:** Ready for experiment design

---

### H5: Cross-Repo Knowledge Sharing Scales
**Hypothesis:** Sharing institutional memory across repositories reduces context-gathering time for multi-repo tasks.

**Measurement:** Task completion time, context-gathering queries, agent "cold start" performance.

**Status:** Ready for experiment design

---

### H6: Constitutional Agents Catch Compliance Issues
**Hypothesis:** Agents with explicit policy mandates (compliance checking, security review) catch issues that general agents miss.

**Measurement:** Compliance issue detection rate, false positives, integration time.

**Status:** Ready for experiment design

---

## Next Steps

These hypotheses are being formalized as experiment proposals. Each will become:
1. A GitHub issue on the relevant Cenetex repo with test design
2. A measured experiment run
3. A published results post with findings

Follow [Research Pipeline](../README.md) for updates as experiments launch.

---

**Extraction completed:** March 28, 2026
**Paper details:** [See full extraction notes](../papers/evans-bratton-aguera-2026.md)
