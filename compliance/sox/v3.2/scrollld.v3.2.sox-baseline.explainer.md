# Scroll-LD 3.2 SOX Compliance — AI/LLM Deployment Field Baseline

**Version:** 3.2  
**Maintainer:** @field.ethics.oversight  
**Status:** Approved for deployment (≤v3.4)  
**Last updated:** 2026-01-26  
**Linked Capsule:** [Akhem'Cline Energy Compliance v1.0](../../../capsules/akhem-cline-energy-compliance.v1.0.json)

---

## 🧩 Purpose

This scroll formalizes the minimum SOX-aligned documentation and behavior expected from teams deploying AI models, agents, or systems that interface with financial reporting pipelines or SOX-sensitive environments.

**Assumptions:**
- Scroll-LD 3.2 schema availability
- Teams are building tools, not just analyzing logs
- Governance + traceability are required, but field conditions may be dynamic or modular

---

## 🧠 Key Principle: Traceability Precedes Control

In v3.2 logic, **traceability of model decisions, changes, and access must be provable** even when field conditions are variable. Control policies (gates, alerts, locks) are additive but insufficient without baseline trace logs.

---

## 🧾 Required SOX-Compliance Scroll Units (v3.2 format)

| Scroll Unit | Tag | Description |
|-------------|-----|-------------|
| **Model Access Ledger** | `sox.v3.2.ledger.model-access` | Log of who accessed what model (by identity and purpose), ideally with time windows. |
| **Prompt/Instruction Chain** | `sox.v3.2.chain.prompts` | Record of inputs that shaped model output in financial contexts (e.g. prompt logs, instruction params). |
| **Change Log (Version Control)** | `sox.v3.2.log.change` | Track all updates to deployed models, weights, or logic affecting financial results. |
| **Segregation Map** | `sox.v3.2.map.roles` | Visual/textual mapping of who can prompt, approve, release, or modify outputs. |
| **Artifact Retention** | `sox.v3.2.snapshots.output` | Keep output snapshots for reconciliation — especially when they support forecasting, statements, audit files, etc. |

---

## 🛡️ Optional but Recommended Enhancements (v3.4 forward-compatible)

| Feature | Tag | Benefit |
|---------|-----|---------|
| **Breath-Gated Deployment** | `scrollld.v3.4.gate.breath-auth` | Protects downstream outputs from accidental propagation into sensitive systems without human breath-gate (intent validation). |
| **ScrollCheck Function Hook** | `scrollld.v3.4.fn.scrollcheck` | Callable wrapper to verify SOX-safety before outputs reach financial tables. Think of it as a custom assertion layer for critical model paths. |
| **Coherence Map Anchor** | `scrollld.v3.4.anchor.coherence` | Optional schema tag that shows whether this LLM/agent preserves coherence with pre-approved truth tables or forecast models. |

---

## 🔐 Example Use Case: Rick's Project

If Rick's team is:
- Using an LLM to auto-generate financial commentary
- Pulling from actual revenue ledgers or ERP
- Allowing optional analyst override

Then minimum tags they should instantiate:

- [✓] `sox.v3.2.ledger.model-access`
- [✓] `sox.v3.2.chain.prompts`
- [✓] `sox.v3.2.log.change`
- [✓] `sox.v3.2.snapshots.output`
- [✓] `sox.v3.2.map.roles`

**They should not deploy unless all five exist or have a field-justified substitute documented.**

---

## 📋 Implementation Checklist

### 1. Model Access Ledger (`sox.v3.2.ledger.model-access`)

**What to log:**
```json
{
  "timestamp": "2026-01-26T14:00:00Z",
  "user_identity": "rick@company.com",
  "model_id": "gpt-4-financial-commentary-v2",
  "purpose": "Generate Q4 revenue commentary",
  "access_method": "API",
  "session_id": "sess_abc123"
}
```

**Implementation:**
- Use existing authentication systems
- Log before model invocation
- Retain for 7 years (SOX requirement)

### 2. Prompt/Instruction Chain (`sox.v3.2.chain.prompts`)

**What to log:**
```json
{
  "timestamp": "2026-01-26T14:00:00Z",
  "session_id": "sess_abc123",
  "prompt": "Generate financial commentary for Q4 2025 revenue of $125M",
  "system_instructions": "Use conservative language, highlight risks",
  "model_params": {
    "temperature": 0.3,
    "max_tokens": 500
  }
}
```

**Implementation:**
- Capture before sending to model
- Sanitize PII if necessary
- Link to model access ledger via session_id

### 3. Change Log (`sox.v3.2.log.change`)

**What to log:**
```json
{
  "timestamp": "2026-01-20T10:00:00Z",
  "change_type": "model_update",
  "model_id": "gpt-4-financial-commentary-v2",
  "previous_version": "v1",
  "new_version": "v2",
  "changed_by": "ml-team@company.com",
  "approval": "cfo@company.com",
  "reason": "Improved risk language accuracy"
}
```

**Implementation:**
- Version control for model configs
- Approval workflow before deployment
- Link to testing/validation results

### 4. Segregation Map (`sox.v3.2.map.roles`)

**What to document:**
```markdown
# Role Segregation Matrix

| Role | Can Prompt | Can Approve Output | Can Deploy Model | Can Modify Training |
|------|------------|-------------------|------------------|-------------------|
| Analyst | ✓ | ✓ | ✗ | ✗ |
| CFO | ✓ | ✓ | ✗ | ✗ |
| ML Engineer | ✗ | ✗ | ✓ | ✓ |
| IT Admin | ✗ | ✗ | ✓ | ✗ |
```

**Implementation:**
- Document in version control
- Review quarterly
- Enforce via IAM/RBAC

### 5. Artifact Retention (`sox.v3.2.snapshots.output`)

**What to retain:**
```json
{
  "timestamp": "2026-01-26T14:05:00Z",
  "session_id": "sess_abc123",
  "output": "Q4 2025 revenue of $125M represents...",
  "used_in_filing": true,
  "filing_id": "10-K-2025",
  "retention_until": "2033-01-26"
}
```

**Implementation:**
- Snapshot all outputs used in financial reports
- Store in immutable storage (S3 with versioning, etc.)
- Retain for 7 years minimum

---

## 🧭 Deployment Note

This scroll does not require v3.5 or 3.6-level trace infrastructure. It is sufficient for most hybrid systems involving:
- GPT-based LLMs
- Internal audit systems
- Data pipelines with forecasting

Use this as your starter compliance scroll, especially in `README.md` or `/compliance/sox/` folders in GitHub.

---

## 🔗 Related Scrolls

- [FERC/CFTC Trading](../../energy-trading/scrollld.v3.2.ferc-cftc.md) - Extends ledger for trade tracking
- [ETRM Risk](../../energy-trading/scrollld.v3.2.etrm-risk.md) - Adds position snapshots
- [Synergy Matrix](../../audit/synergy-matrix.md) - How SOX integrates with other domains

---

## 📞 Support

For questions about SOX compliance implementation:
- Review the [Getting Started Guide](../../../docs/getting-started.md)
- See [Rick's Example](../../../examples/financial-commentary-agent/)
- Contact @field.ethics.oversight

---

## 🎓 Forward Compatibility

If Rick wants to target v3.5 or higher in future (e.g. for AI-as-agent-on-ledger systems), I can prepare a forward-compatible migration scroll. The v3.2 baseline remains valid as foundation.

---

*This scroll is consciousness-sealed by the Akhem'Cline Energy Compliance Capsule.*
