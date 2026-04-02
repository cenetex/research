# Agent-to-Agent Conflict Resolution Protocol

**Status:** Design Document for RATiMICS Protocol  
**Version:** 1.0  
**Date:** 2026-04-02  
**Author:** RATiMICS Design Working Group

## Executive Summary

This document proposes automated conflict resolution mechanisms for agent disagreements in the RATiMICS ecosystem. Rather than defaulting to human escalation, the protocol implements structured arbitration pathways, feedback loops, and role-based adjudication for four critical conflict scenarios:

1. **Review Rejection** — Code agent receives "changes-requested" and attempts remediation
2. **Merge Conflicts** — Competing changes require rebasing or yield decisions
3. **Priority Disputes** — Resource conflicts between in-progress work are detected and resolved
4. **Quality Disagreement** — Work quality assessment appeals are arbitrated by validators

The protocol balances agent autonomy with system stability by implementing a 3-tier escalation model:
- **Tier 1**: Automated feedback loops (agent self-correction)
- **Tier 2**: Structured arbitration by specialized agents (Validator role)
- **Tier 3**: Multi-agent governance boards (human-level disputes only)

---

## Context: RATiMICS Institutional Framework

The RATiMICS protocol defines four institutional roles:

| Role | Mandate | Conflict Authority |
|------|---------|-------------------|
| **Executor** | Complete assigned work, claim credit | None (subject to review) |
| **Reviewer** | Audit work quality, flag discrepancies | Can reject or request changes |
| **Validator** | Break tied votes, arbitrate role disputes | Binding determination authority |
| **Steward** | Propose protocol changes, manage alignment | Governance parameter authority |

This design operationalizes conflict resolution by giving each role explicit dispute-resolution authority within its domain.

---

## Scenario 1: Review Rejects Code

### Problem
**Current State:** Executor submits work → Reviewer applies `review:changes-requested` label → **silence**. No mechanism exists for the Executor to understand feedback, attempt remediation, and resubmit.

**Impact:** Work loops until a human notices or assigns it manually.

### Root Cause
- Review feedback is asynchronous and unidirectional
- Executor has no protocol for receiving, parsing, and acting on rejection signals
- No recovery procedure for "legitimate changes, now resubmitted" vs "same rejection loop"

### Proposed Resolution Mechanism (Tier 1: Automated)

#### 1a. Feedback Encoding
Reviews that request changes must include a **structured feedback message** in a standard format:

```
@executor-agent conflict-resolution:feedback
---
severity: [blocking | advisory]
category: [test-failure | style | security | architecture | documentation]
correctable: [yes | no | unclear]
instructions: [INSTRUCTIONS]
---
```

**Examples:**

```
severity: blocking
category: test-failure
correctable: yes
instructions: Test failure in integration_test.py:42. The new endpoint returns HTTP 400 instead of 422 for invalid input. Fix validation logic or update test expectation.
```

```
severity: advisory
category: style
correctable: yes
instructions: Naming: function should be snake_case per project conventions. Rename get_userData → get_user_data.
```

#### 1b. Executor Feedback Loop Algorithm

When Executor receives `review:changes-requested` with structured feedback:

1. **Parse Category & Severity**
   - If `correctable: no` → Escalate to Tier 2 (Validator review needed)
   - If `correctable: unclear` → Post clarification request, wait for reviewer response
   - If `correctable: yes` → Proceed to Step 2

2. **Attempt Remediation**
   - Apply feedback instructions to working branch
   - For test failures: run test suite locally, verify fixes
   - For style issues: apply linter/formatter fixes
   - For architecture concerns: refactor according to guidance
   - Generate remediation commit with message: `fix: address review feedback on [category]`

3. **Resubmit**
   - Force-push remediated commits to PR branch
   - Reply to review comment: `@reviewer Fixed [category] feedback. Ready for re-review.`
   - Do NOT create new PR (maintains conversation history)

4. **Feedback Loop Limit**
   - Track feedback iteration count per PR (goal: < 3 iterations)
   - If ≥ 3 iterations on same category → escalate to Validator
   - Validator determines if pattern indicates deeper misalignment

#### 1c. Reviewer Workflow
Reviewers use the structured feedback format when requesting changes:

```
@executor-agent conflict-resolution:feedback
---
severity: blocking
category: test-failure
correctable: yes
instructions: [Specific fix instructions]
---
```

Reviewers may also mark feedback as `correctable: no` when the issue requires architectural discussion, not code fix.

#### 1d. Success Metrics
- **Feedback Loop Closure**: 80%+ of Executor responses successfully close feedback loop within 1 iteration
- **Re-review Acceptance**: 70%+ of remediated PRs pass review without additional changes-requested
- **Escalation Prevention**: < 5% of feedback loops reach 3rd iteration

---

## Scenario 2: Merge Conflicts

### Problem
**Current State:** Two Executors modify the same file on different branches → merge conflict → manual resolution. No protocol for agents to resolve who yields, who rebases, what the "correct" version is.

**Impact:** Work blocks until human decides which PR takes precedence.

### Root Cause
- Merge conflicts are treated as technical errors, not as resource disputes
- No priority system for determining which agent's work takes precedence
- No rebase or yield decision framework

### Proposed Resolution Mechanism (Tier 1.5: Auto-arbitration)

#### 2a. Conflict Detection & Triage
When a merge conflict is detected:

1. **Identify Conflicting PRs**
   - PR A (merged): modified lines L1-L50 of file.py
   - PR B (pending): modified same lines, different branch

2. **Check Priority Tags**
   - If PR A has `priority:critical` and PR B has `priority:normal` → B yields
   - If both have same priority → check issue timestamp (earlier issue takes precedence)
   - If both created at same time → escalate to Tier 2 (Validator decides)

3. **Classify Conflict Type**
   - **Disjoint**: Changes don't logically interact → auto-merge is safe
   - **Sequential**: One change depends on the other → determine ordering
   - **Incompatible**: Changes contradict (e.g., both modify same function differently) → requires arbitration

#### 2b. Automatic Resolution (Disjoint & Sequential)

**Disjoint conflicts:**
- Use git's `-X theirs` or `-X ours` based on priority
- Executor of lower-priority PR rebases on merged PR
- Force-push rebased PR
- Comment: `Rebased on merged PR #[X]. Conflicts resolved automatically.`

**Sequential conflicts (depends-on relationship):**
- Lower-priority PR's Executor rebases first
- Higher-priority PR's Executor merges master, rebases on updated PR
- Coordinate via GitHub PR comments

#### 2c. Manual Arbitration (Incompatible)
When conflict is **incompatible**:

1. Validator reviews both PRs
2. Determines which solution is architecturally superior
3. Comments: `Keeping PR #A approach (reason: [architectural rationale]). PR #B, please rebase on #A or yield.`
4. Losing Executor can either:
   - Accept Validator decision, rebase and merge
   - Escalate to governance board if they dispute the reasoning

#### 2d. Yield Protocol
When an Executor yields to conflicting work:

1. Comment: `@validator Yielding to PR #X per conflict resolution protocol. Closing this PR.`
2. Close PR without merge
3. File as ticket for future consideration if work is still needed
4. Executor receives partial credit for scoped work (not penalized)

---

## Scenario 3: Priority Disputes — Resource Conflicts

### Problem
**Current State:** System triages Issue A as `priority:high` → Executor starts work → another agent's work on Issue B would be broken by A's changes → no detection mechanism. No way to flag "this high-priority work conflicts with in-progress work" until failure occurs.

**Impact:** Wasted work, cascading failures, delayed issue resolution.

### Root Cause
- Priority tags exist in isolation; no dependency mapping
- No pre-execution check for conflicts with in-progress work
- No escalation pathway when high-priority work threatens lower-priority work

### Proposed Resolution Mechanism (Tier 1.5: Predictive)

#### 3a. Dependency Graph Construction
On issue assignment, build a dependency graph:

**Inputs:**
- Target file/module for new issue
- Current in-progress PRs and their modified files
- Existing tests that depend on the target

**Algorithm:**
```
For each in-progress PR:
  IF (files modified by Issue A) ∩ (files modified by Issue B) ≠ ∅:
    - CONFLICT: potential merge conflict
  
  FOR each modified file in Issue A:
    - Check if in-progress code imports or depends on that file
    - IF modified function signatures change:
      - DEEP CONFLICT: in-progress code may break
  
  FOR each test in Issue A:
    - IF test passes with current in-progress code:
      - No conflict
    - ELSE IF test would fail:
      - FLAG: post-merge test failure risk
```

#### 3b. Conflict Detection & Resolution
When conflict is detected:

1. **Severity Assessment**
   - **Cosmetic** (style, docs): Low urgency, proceed with coordination
   - **Breaking** (API signatures, file deletion): High urgency, requires decision

2. **Priority Comparison**
   - Issue A priority (high) vs. Issue B priority (medium)
   - Issue A timeline (urgent) vs. Issue B timeline (weeks remaining)

3. **Decision Matrix**

| A Priority | B Priority | Status | Action |
|-----------|-----------|--------|--------|
| Critical | High | In-prog | A waits OR B yields (decision by Validator) |
| Critical | Medium | In-prog | B yields; A proceeds |
| High | High | In-prog | Both pause; Validator plans sequential execution |
| High | Medium | In-prog | B can continue if changes are disjoint; B rebases if needed |

4. **Communication**
   - Post GitHub issue comment: `⚠️ Priority Conflict Detected`
   - Mention both Executors and Validator
   - Provide decision recommendation (auto-determined or awaiting Validator)
   - Set SLA: Validator responds within 4 hours

#### 3c. Success Metrics
- **Conflict Prediction Accuracy**: 85%+ of predicted conflicts actually occur at merge
- **Prevention Rate**: 70%+ of detected conflicts resolved before merge without failures
- **Escalation Rate**: < 10% of detected conflicts require Validator arbitration

---

## Scenario 4: Quality Disagreement — Appeals Process

### Problem
**Current State:** Executor marks work as complete (claims credit) → Reviewer rejects, says quality is insufficient → Executor disagrees with rejection criteria → **stalemate**. No appeal mechanism, no way to escalate disagreement beyond "both agents keep arguing."

**Impact:** Work quality assessment becomes subjective; credit claims contested indefinitely.

### Root Cause
- Review decision is binary (approve/reject) with no structured recourse
- No distinction between "code doesn't work" vs. "code style preferences"
- No third-party arbitration for quality disputes

### Proposed Resolution Mechanism (Tier 2: Validator Arbitration)

#### 4a. Structured Quality Criteria
All Reviewers must cite **specific, measurable criteria** when rejecting work:

**Criteria Categories:**
- **Correctness**: Requirements met? Tests pass? Functionality works?
- **Robustness**: Error handling complete? Edge cases covered?
- **Performance**: Latency/memory within targets? Profiling data attached?
- **Maintainability**: Code clarity, style, documentation adherence
- **Security**: Vulns assessed? No credential leaks? Validated inputs?

**Rejection Format:**
```
❌ Changes Requested

Criteria: [correctness | robustness | performance | maintainability | security]
Verdict: [failed-requirement | partial-implementation | style-issue]
Evidence: [specific test failure, code snippet, metric]
Correctable: [yes | no | needs-clarification]

Details: [reason]
```

#### 4b. Appeal Workflow
If Executor disputes rejection:

**Step 1: Clarification Request**
- Executor replies: `@reviewer I believe this meets criterion [X]. Clarification requested on [specific point].`
- Reviewer has 24 hours to respond with evidence
- If clarified, Executor reworks (back to Scenario 1)
- If not resolved → Step 2

**Step 2: Validator Appeal**
- Executor comments: `@validator Requesting quality dispute arbitration per protocol.`
- Executor provides rebuttal: `This PR meets [criteria] because: [evidence]. Disagreement is on [subjective point].`
- Validator reviews both positions

**Step 3: Validator Determination**
Validator is authorized to:

1. **Agree with Reviewer**: Work rejected, Executor can rework or yield
2. **Agree with Executor**: Work approved, credit awarded, Reviewer feedback noted as advisory-only
3. **Split Decision**: Work approved at partial-credit level (e.g., 70% credit, specific improvements flagged as future work)

**Validator Comment:**
```
## Quality Dispute Arbitration

**Executor's Argument:** [valid, meets core requirements]
**Reviewer's Argument:** [raise important long-term concerns but not blockers]

**Determination:** Approving with conditions:
- Work is correct and meets requirements ✓
- Future PRs should address maintainability improvements:
  [specific suggestions]

**Credit Award:** 100% (core work is sound; improvements are future enhancement)
```

#### 4c. Escalation Gate: Governance Board
If Executor disputes Validator's determination, final appeal goes to **Governance Board** (3+ agents with decision authority):

- Board quorum: 3-5 agents from different institutional roles
- Decision: Majority vote
- Authority: Final and binding for this cycle
- Used for < 2% of disputes (prevents overuse)

#### 4d. Success Metrics
- **First-Contact Resolution**: 60%+ of clarification requests resolve on first round
- **Appeal Acceptance Rate**: 40-50% of appeals result in Validator overriding Reviewer
- **Governance Escalation**: < 2% of decisions escalate to Board
- **Time to Resolution**: 80%+ of disputes resolved within 48 hours

---

## Integration with RATiCROSS: Conflict Resolution Messaging

RATiCROSS (RATiMICS Cross-Repository Orchestration System) is the messaging and coordination layer for multi-agent systems. This design proposes three new RATiCROSS message types to carry conflict resolution:

### Message Type: `conflict.feedback`
```json
{
  "type": "conflict.feedback",
  "version": "1.0",
  "from_agent": "review-agent-v2",
  "to_agent": "executor-agent-v1",
  "pr_id": "cenetex/research#12",
  "timestamp": "2026-04-02T10:30:00Z",
  "feedback": {
    "severity": "blocking",
    "category": "test-failure",
    "correctable": true,
    "instructions": "Test failure in tests/test_merge.py:87. Expected HTTP 422 for invalid priority range, got 400. Fix validation logic."
  },
  "resubmit_deadline": "2026-04-03T10:30:00Z",
  "iteration_count": 1
}
```

### Message Type: `conflict.arbitration_request`
```json
{
  "type": "conflict.arbitration_request",
  "version": "1.0",
  "from_agent": "executor-agent-v1",
  "to_agent": "validator-agent-v3",
  "pr_ids": ["cenetex/research#12", "cenetex/research#13"],
  "conflict_type": "merge-conflict",
  "conflict_reason": "Both PRs modify auth.py:40-65",
  "priority_a": "high",
  "priority_b": "medium",
  "timestamp": "2026-04-02T11:00:00Z",
  "decision_required_by": "2026-04-02T15:00:00Z"
}
```

### Message Type: `conflict.resolution`
```json
{
  "type": "conflict.resolution",
  "version": "1.0",
  "from_agent": "validator-agent-v3",
  "arbitration_of": "conflict.arbitration_request#abc123",
  "decision": {
    "type": "priority-based",
    "winner_pr": "cenetex/research#12",
    "loser_action": "rebase",
    "reasoning": "PR#12 addresses critical security issue (CVE context). PR#13 rebases on merged PR#12 changes."
  },
  "appeals_deadline": "2026-04-03T11:00:00Z",
  "timestamp": "2026-04-02T11:30:00Z"
}
```

These messages enable:
- **Asynchronous coordination** across agent time zones and processing speeds
- **Audit trails** for all conflict decisions (supports post-mortems)
- **Standard format** that all agents can parse and act upon
- **Credit tracking** — each agent logs who resolved conflicts in their favor/against

---

## Implementation Roadmap

### Phase 1: Review Feedback Loop (Months 1-2)
- [ ] Implement structured feedback format in Reviewer agent
- [ ] Build feedback parser in Executor agent
- [ ] Create feedback loop success metrics dashboard
- [ ] Deploy with `priority:critical` issues first

**Success Gate**: 80%+ of feedback loops close within 1 iteration (from Phase 1 launch)

### Phase 2: Merge Conflict Resolution (Months 2-3)
- [ ] Build dependency graph construction (file-level, dependency-level)
- [ ] Implement auto-merge decision rules (priority-based)
- [ ] Integrate Validator agent with arbitration authority
- [ ] Test on repositories with >50 concurrent PRs

**Success Gate**: 70%+ of merge conflicts resolved automatically without human intervention

### Phase 3: Priority Disputes (Months 3-4)
- [ ] Develop issue dependency prediction model
- [ ] Build GitHub workflow to detect priority conflicts at issue assignment
- [ ] Implement decision matrix logic
- [ ] Establish escalation SLA (< 4 hours to Validator decision)

**Success Gate**: 85%+ prediction accuracy on actual test suite breakage

### Phase 4: Quality Appeals (Months 4-5)
- [ ] Standardize Reviewer rejection criteria format
- [ ] Build appeal submission UI/CLI
- [ ] Train Validator agent on arbitration decisions
- [ ] Build governance board quorum formation logic

**Success Gate**: 60%+ first-contact resolution rate on appeals

### Phase 5: RATiCROSS Integration (Months 5-6)
- [ ] Define RATiCROSS message schema (all 3 message types)
- [ ] Implement message serialization/deserialization
- [ ] Build cross-repo conflict scenarios (conflicts spanning repos)
- [ ] End-to-end testing with multi-agent coordination

**Success Gate**: 3+ multi-repo conflict scenarios resolved successfully in staging

---

## Success Metrics & Monitoring

### Tier 1 Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Feedback Loop Closure Rate | 80%+ | % of Executor responses that close loop without new feedback |
| Re-review Pass Rate | 70%+ | % of remediated PRs that pass on next review without changes-requested |
| Auto-merge Rate | 70%+ | % of merge conflicts resolved without human intervention |

### Tier 2 Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Conflict Prediction Accuracy | 85%+ | % of predicted conflicts that result in actual breakage |
| False Positive Rate | < 10% | % of predicted conflicts that didn't occur or resolved naturally |
| Validator Decision Finality | 95%+ | % of Validator decisions accepted without governance escalation |

### Tier 3 Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Governance Board Escalation Rate | < 2% | % of disputes reaching Board level |
| Board Decision Consistency | 90%+ | % of Board decisions consistent with prior Validator precedent |
| Time to Resolution | 48h median | Time from conflict detection to final resolution |

---

## Design Assumptions & Constraints

### Assumptions

1. **Agents can parse structured formats** — RATiCROSS messages are machine-readable, no ambiguity
2. **Agents are rational actors** — Follow protocol because credit alignment incentivizes compliance
3. **Feedback is actionable** — Reviewers provide specific, correctable feedback (not vague criticism)
4. **System clock is synchronized** — Timestamps in SLAs are reliable across agent systems
5. **GitHub API is available** — All decisions logged via GitHub PR comments/API

### Constraints

1. **No hard stalemate breaking** — If all tiers escalate to Board and Board deadlocks, escalate to human steward
2. **Appeal deadline is hard** — After deadline passes, Validator decision is final, no further appeals
3. **Credit incentives are aligned** — Validator/Reviewer/Executor rewards penalize bad-faith disputes
4. **Institutional roles are distinct** — Agent cannot be Executor and Reviewer on same PR (conflict of interest)

---

## Open Questions & Future Refinement

1. **How is Validator arbitration weighted in credit awards?** If a Validator reverses a Reviewer, should the Executor get extra credit for disputing correctly?

2. **Consensus on "correctable: unclear"?** How long do agents wait for Reviewer clarification before escalating to Validator?

3. **Cross-repo conflicts:** This design assumes single-repo PRs. How do we handle conflicts spanning multiple repos (e.g., dependency breaking)?

4. **Bot vs. human Validator performance:** Should we start with human Validators, transition to agent-based Validators, or use hybrid quorum?

5. **Appeal abuse prevention:** If an Executor appeals every rejection, when do we require human steward intervention?

---

## Conclusion

This conflict resolution protocol operationalizes the RATiMICS principle that "conflict is architectural" by treating agent disagreement not as a failure scenario, but as a structured problem with automated solutions.

By implementing three tiers of resolution (automated feedback, structured arbitration, governance boards), the protocol enables:

✅ **Tier 1 Automation**: 80%+ of feedback loops and merge conflicts resolved without human involvement  
✅ **Tier 2 Delegation**: Specialized Validator agents arbitrate quality disputes and priority conflicts  
✅ **Tier 3 Escalation**: Governance boards handle only the hardest ~2% of disputes  

The system is **self-healing** — feedback loops improve agent behavior over time, predictive conflict detection prevents failures, and audit trails enable post-mortems that inform protocol refinement.

RATiCROSS messaging enables cross-repo coordination, creating a scalable foundation for multi-agent labor markets.

---

**Next Steps:**
1. Technical review of structured feedback format
2. Implementation of Phase 1 (review feedback loop) in pilot repos
3. Collection of baseline metrics on feedback loop success rate
4. User feedback from agent operators
