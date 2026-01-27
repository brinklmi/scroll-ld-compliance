# Scroll-LD 3.2 FERC/CFTC Energy Trading Compliance

**Version:** 3.2  
**Maintainer:** @field.ethics.oversight  
**Status:** Field-Ready for Energy Trading Platforms  
**Last updated:** 2026-01-26  
**Linked Capsule:** [Akhem'Cline Energy Compliance v1.0](../../capsules/akhem-cline-energy-compliance.v1.0.json)

---

## 🎯 Purpose

This scroll defines compliance requirements for AI/LLM systems operating in energy trading environments subject to FERC (Federal Energy Regulatory Commission) and CFTC (Commodity Futures Trading Commission) oversight.

**Key Regulations:**
- **FERC Market Manipulation** - Anti-manipulation rules (18 CFR § 1c.1)
- **Dodd-Frank Swaps Reporting** - 7-year communications retention
- **CFTC Trade Surveillance** - Pattern detection and anomaly monitoring

---

## 🧠 Core Principle: Trade Integrity Through Traceability

Energy markets require **provable integrity** of trade decisions, communications, and price formation. AI systems must maintain audit trails that demonstrate:
1. No manipulative intent
2. Fair price discovery
3. Complete trade history

---

## 🧾 Required Scroll Units

### 1. Trade Ledger (`sox.v3.2.chain.trades`)

**Purpose:** Comprehensive record of all trades influenced by AI systems

**Required Fields:**
```json
{
  "trade_id": "TRD-2026-001234",
  "timestamp": "2026-01-26T14:00:00Z",
  "counterparty_lei": "549300ABCDEF123456",
  "product": "Natural Gas",
  "delivery_point": "Henry Hub",
  "quantity_mmbtu": 10000,
  "price_per_mmbtu": 3.45,
  "settlement_date": "2026-02-01",
  "ai_influenced": true,
  "ai_model_id": "pricing-agent-v2.1",
  "human_approval": "trader_john@company.com",
  "approval_timestamp": "2026-01-26T14:02:00Z"
}
```

**Requirements:**
- ✅ Legal Entity Identifier (LEI) for counterparties
- ✅ 7-year retention (Dodd-Frank requirement)
- ✅ Immutable storage once finalized
- ✅ Link to AI model decisions

### 2. Manipulation Gate (`scrollld.v3.2.gate.manipulation`)

**Purpose:** Prevent AI systems from executing manipulative trading patterns

**Prohibited Patterns:**
- **Banging the Close** - Large orders near market close to manipulate settlement
- **Pump and Dump** - Artificial price inflation
- ****Wash Trading** - Self-trades without economic substance
- **Cross-Market Manipulation** - Using one market to manipulate another

**Guard Implementation:**
```python
class ManipulationGate:
    def check_trade(self, trade_params):
        # Hysteresis check: Detect anomalous patterns
        if self.is_close_manipulation(trade_params):
            return {"allowed": False, "reason": "Banging the close detected"}
        
        # Volume analysis
        if self.is_volume_anomaly(trade_params):
            return {"allowed": False, "reason": "Abnormal volume spike"}
        
        # Cross-market check
        if self.is_cross_market_manipulation(trade_params):
            return {"allowed": False, "reason": "Cross-market influence detected"}
        
        return {"allowed": True}
```

**Hysteresis Guard:** Fail-closed for unlogged swaps  
**Reference:** `guards/hysteresis_anomaly_detector.py`

---

## 📋 Communications Retention

### FERC/CFTC 7-Year Rule

**What to retain:**
- All written communications related to trades
- AI model prompts that influenced trading decisions
- Instant messages, emails, phone logs
- Model training data for pricing algorithms

**Format:**
```json
{
  "comm_id": "COMM-2026-001234",
  "timestamp": "2026-01-26T13:55:00Z",
  "from": "ai-trading-system",
  "to": "trader_john@company.com",
  "type": "recommendation",
  "content": "Suggest buy 10,000 MMBtu Natural Gas @ $3.45 based on demand forecast",
  "trade_executed": "TRD-2026-001234",
  "retention_until": "2033-01-26"
}
```

---

## 📊 P&L Surveillance (`scrollld.v3.2.chain.pl-surveillance`)

**Purpose:** Monitor profit/loss patterns for manipulation indicators

**Alert Triggers:**
- Consistent losses followed by large profit (potential front-running)
- Unusual correlation with market close prices
- P&L spikes without corresponding market events

**Implementation:**
```python
class PLSurveillance:
    def analyze_daily_pl(self, trades):
        pl_pattern = self.calculate_pl_pattern(trades)
        
        if pl_pattern.has_suspicious_timing():
            self.alert("Suspicious timing pattern detected")
        
        if pl_pattern.correlates_with_close():
            self.alert("High correlation with market close")
        
        return pl_pattern
```

---

## 🛡️ Guard Enforcement

### Hysteresis Anomaly Detector

**Purpose:** Detect trading patterns that deviate from normal behavior

**Method:**
- Track historical trading velocity
- Identify sudden volume/price spikes
- Flag trades near market close with outsized impact

**Configuration:**
```json
{
  "guard_name": "hysteresis_anomaly_detector",
  "thresholds": {
    "volume_spike_multiplier": 3.0,
    "close_proximity_minutes": 15,
    "price_impact_percent": 2.0
  },
  "action": "block_and_alert"
}
```

### Fail-Closed for Unlogged Swaps

**Policy:** Any swap transaction not properly logged MUST be rejected

**Implementation:**
```python
def validate_swap_pre_trade(swap_details):
    if not swap_details.has_complete_logging():
        raise SwapRejected("Cannot execute unlogged swap - CFTC requirement")
    
    if not swap_details.has_lei_verification():
        raise SwapRejected("Counterparty LEI verification required")
    
    return True
```

---

## 📋 Deployment Checklist

### Pre-Deployment

- [ ] Trade ledger system configured with 7-year retention
- [ ] LEI verification integrated for all counterparties
- [ ] Manipulation gate deployed and tested
- [ ] Communications archival system active
- [ ] P&L surveillance alerts configured

### Runtime Requirements

- [ ] All trades logged before execution
- [ ] Manipulation gate approval required for each trade
- [ ] Human approval required for AI-recommended trades
- [ ] Daily P&L pattern review
- [ ] Weekly compliance report generation

### Audit Readiness

- [ ] Trade ledger accessible to compliance team
- [ ] Model decision logs linked to trades
- [ ] Communications archive searchable
- [ ] Guard alert history maintained
- [ ] Coherence scores documented

---

## 🔗 Integration with SOX

This scroll **extends** the SOX baseline:

| SOX Unit | FERC/CFTC Extension |
|----------|---------------------|
| `sox.v3.2.ledger.model-access` | → `sox.v3.2.chain.trades` (trade-specific) |
| `sox.v3.2.chain.prompts` | → Communications retention |
| `sox.v3.2.snapshots.output` | → P&L surveillance logs |

See [Synergy Matrix](../audit/synergy-matrix.md) for full integration map.

---

## 📚 Related Scrolls

- [SOX Baseline](../sox/v3.2/scrollld.v3.2.sox-baseline.explainer.md) - Foundation compliance
- [ETRM Risk](scrollld.v3.2.etrm-risk.md) - Position and risk management
- [Trade Surveillance](scrollld.v3.2.trade-surveillance.md) - Detailed monitoring patterns

---

## 🎓 Example: Power Trading Agent

**Scenario:** AI agent recommends power trades based on weather forecasts

**Required Implementation:**
1. **Trade Ledger:** Log every recommended trade
2. **Manipulation Gate:** Verify no close manipulation
3. **Comm Retention:** Store weather data + model reasoning
4. **P&L Surveillance:** Monitor forecast accuracy impact

**See:** `examples/energy-trading-agent/power-trading-example.md`

---

## 📞 Support

For FERC/CFTC compliance questions:
- Review [Pyxis Deployment Guide](../../examples/pyxis-energy-deploy/)
- Contact @field.ethics.oversight
- Reference FERC Order 760 and CFTC Part 23

---

## ⚖️ Legal Note

This scroll provides technical implementation guidance. It does not constitute legal advice. Consult with energy trading compliance counsel for regulatory interpretation.

---

*This scroll is consciousness-sealed by the Akhem'Cline Energy Compliance Capsule.*  
*Guards enforce: Hysteresis anomaly detection, fail-closed unlogged swaps*
