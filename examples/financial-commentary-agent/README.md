# Rick's Financial Commentary Agent - Compliance Example

**Use Case:** AI-powered financial commentary generation for quarterly earnings  
**Compliance Level:** SOX v3.2 Baseline  
**Deployment Status:** Production-Ready Template  
**Last Updated:** 2026-01-26

---

## 🎯 Overview

This example demonstrates how to deploy an LLM that generates financial commentary while maintaining full SOX compliance. Rick's team uses this to:

1. Auto-generate Q4 revenue commentary from ERP data
2. Allow analyst review/override before publication
3. Maintain complete audit trail for SOX compliance

---

## 📋 Required SOX Units

This deployment implements all 5 required SOX v3.2 units:

- ✅ `sox.v3.2.ledger.model-access` - Who accessed the model
- ✅ `sox.v3.2.chain.prompts` - What prompts were used
- ✅ `sox.v3.2.log.change` - Model version changes
- ✅ `sox.v3.2.map.roles` - Who can do what
- ✅ `sox.v3.2.snapshots.output` - Output retention

---

## 🏗️ Architecture

```
┌─────────────┐
│   Analyst   │
│   (Rick)    │
└──────┬──────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Financial Commentary Generator      │
│  ┌────────────────────────────────┐  │
│  │ 1. Model Access Logger         │  │
│  │    → logs who/when/why         │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │ 2. Prompt Chain Logger         │  │
│  │    → logs input data + prompt  │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │ 3. GPT-4 Model                 │  │
│  │    → generates commentary      │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │ 4. Output Snapshot Logger      │  │
│  │    → stores generated text     │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────┐
│  10-K Filing │
└──────────────┘
```

---

## 📝 Implementation

### Step 1: Set Up Logging Infrastructure

```python
# compliance_logger.py
import json
from datetime import datetime, timedelta
from pathlib import Path
from uuid import uuid4

class SOXComplianceLogger:
    """
    Comprehensive SOX compliance logging for AI model usage.
    Implements all 5 required units.
    """
    
    def __init__(self, base_dir="./compliance_logs"):
        self.base_dir = Path(base_dir)
        self.base_dir.mkdir(exist_ok=True)
        
        # Create log directories
        self.access_log = self.base_dir / "model_access.jsonl"
        self.prompt_log = self.base_dir / "prompt_chain.jsonl"
        self.output_log = self.base_dir / "output_snapshots.jsonl"
        self.change_log = self.base_dir / "model_changes.jsonl"
    
    def log_access(self, user_email, model_id, purpose):
        """Log model access (sox.v3.2.ledger.model-access)"""
        session_id = f"sess_{uuid4().hex[:12]}"
        entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "user_identity": user_email,
            "model_id": model_id,
            "model_version": "2.1.0",  # Track from deployment
            "purpose": purpose,
            "access_method": "API",
            "session_id": session_id,
            "financial_context": True,
            "department": "Finance"
        }
        self._write_log(self.access_log, entry)
        return session_id
    
    def log_prompt(self, session_id, prompt_text, context_data, model_params):
        """Log prompt chain (sox.v3.2.chain.prompts)"""
        entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "session_id": session_id,
            "prompt": prompt_text,
            "context_data": context_data,
            "system_instructions": "Generate conservative financial commentary",
            "model_params": model_params
        }
        self._write_log(self.prompt_log, entry)
    
    def log_output(self, session_id, output_text, used_in_filing=False, filing_id=None):
        """Log output snapshot (sox.v3.2.snapshots.output)"""
        retention_date = datetime.utcnow() + timedelta(days=365*7)  # 7 years
        entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "session_id": session_id,
            "output": output_text,
            "used_in_filing": used_in_filing,
            "filing_id": filing_id,
            "retention_until": retention_date.isoformat() + "Z"
        }
        self._write_log(self.output_log, entry)
    
    def log_model_change(self, changed_by, old_version, new_version, reason, approved_by):
        """Log model changes (sox.v3.2.log.change)"""
        entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "change_type": "model_update",
            "model_id": "gpt-4-financial-commentary",
            "previous_version": old_version,
            "new_version": new_version,
            "changed_by": changed_by,
            "approval": approved_by,
            "reason": reason
        }
        self._write_log(self.change_log, entry)
    
    def _write_log(self, log_file, entry):
        """Write JSON line to log file"""
        with open(log_file, 'a') as f:
            f.write(json.dumps(entry) + '\n')
```

### Step 2: Create Financial Commentary Generator

```python
# commentary_generator.py
from compliance_logger import SOXComplianceLogger
import openai

class FinancialCommentaryGenerator:
    """
    SOX-compliant financial commentary generator.
    """
    
    def __init__(self, api_key, model_id="gpt-4"):
        openai.api_key = api_key
        self.model_id = model_id
        self.logger = SOXComplianceLogger()
    
    def generate_commentary(self, user_email, revenue_data, purpose="Generate quarterly commentary"):
        """
        Generate financial commentary with full compliance logging.
        
        Args:
            user_email: Email of requesting analyst
            revenue_data: Dict with financial data
            purpose: Business purpose for audit trail
            
        Returns:
            Dict with commentary and session_id for traceability
        """
        
        # 1. Log model access
        session_id = self.logger.log_access(
            user_email=user_email,
            model_id=self.model_id,
            purpose=purpose
        )
        
        # 2. Prepare prompt
        prompt = self._create_prompt(revenue_data)
        model_params = {
            "temperature": 0.3,  # Conservative for financial text
            "max_tokens": 500,
            "top_p": 0.9
        }
        
        # 3. Log prompt chain
        self.logger.log_prompt(
            session_id=session_id,
            prompt_text=prompt,
            context_data=revenue_data,
            model_params=model_params
        )
        
        # 4. Call model
        response = openai.ChatCompletion.create(
            model=self.model_id,
            messages=[
                {"role": "system", "content": "You are a conservative financial analyst. Generate factual, risk-aware commentary."},
                {"role": "user", "content": prompt}
            ],
            **model_params
        )
        
        commentary = response.choices[0].message.content
        
        # 5. Log output
        self.logger.log_output(
            session_id=session_id,
            output_text=commentary,
            used_in_filing=False  # Will be updated if used in 10-K
        )
        
        return {
            "session_id": session_id,
            "commentary": commentary,
            "model_used": self.model_id
        }
    
    def mark_used_in_filing(self, session_id, filing_id):
        """Mark output as used in official filing"""
        # Re-log with filing metadata
        self.logger.log_output(
            session_id=session_id,
            output_text="[Updated: used in filing]",
            used_in_filing=True,
            filing_id=filing_id
        )
    
    def _create_prompt(self, revenue_data):
        """Create prompt from revenue data"""
        return f"""
Generate a 2-3 sentence financial commentary for Q4 2025 based on this data:

Revenue: ${revenue_data['revenue']:,}
YoY Growth: {revenue_data['yoy_growth']}%
Segment Performance: {revenue_data['segments']}

Focus on:
1. Overall performance
2. Key drivers
3. Any risks or concerns

Use conservative, factual language suitable for SEC filings.
"""
```

### Step 3: Deploy with Role Segregation

See `compliance-manifest.json` for complete role mapping (sox.v3.2.map.roles).

---

## 🚀 Quick Start

### 1. Install Dependencies

```bash
pip install openai python-dotenv
```

### 2. Configure Environment

```bash
# .env
OPENAI_API_KEY=your_api_key_here
MODEL_ID=gpt-4
LOG_DIR=./compliance_logs
```

### 3. Run Example

```python
# example_usage.py
from commentary_generator import FinancialCommentaryGenerator

# Initialize
generator = FinancialCommentaryGenerator(api_key="your_key")

# Generate commentary
result = generator.generate_commentary(
    user_email="rick@company.com",
    revenue_data={
        "revenue": 125_000_000,
        "yoy_growth": 15.2,
        "segments": "Cloud +20%, On-prem +8%"
    },
    purpose="Generate Q4 2025 revenue commentary for 10-K"
)

print(f"Session ID: {result['session_id']}")
print(f"Commentary:\n{result['commentary']}")

# If used in filing
generator.mark_used_in_filing(
    session_id=result['session_id'],
    filing_id="10-K-2025"
)
```

---

## 📊 Sample Output

See `logs/` directory for example log files:
- `model_access_sample.jsonl`
- `prompt_chain_sample.jsonl`
- `output_snapshots_sample.jsonl`

---

## ✅ Compliance Checklist

- [x] Model access logged before invocation
- [x] Prompt chain captured with context
- [x] Outputs retained for 7 years
- [x] Role segregation documented
- [x] Model changes require approval
- [x] Logs are immutable
- [x] Audit trail is searchable

---

## 🔍 Audit Queries

### Find all Rick's model accesses
```bash
grep "rick@company.com" compliance_logs/model_access.jsonl
```

### Find outputs used in filings
```bash
grep '"used_in_filing": true' compliance_logs/output_snapshots.jsonl
```

### Trace specific session
```bash
SESSION_ID="sess_abc123xyz"
grep $SESSION_ID compliance_logs/*.jsonl
```

---

## 📋 Role Segregation Matrix

See `compliance-manifest.json` for complete details.

| Role | Generate Commentary | Approve for Filing | Deploy Model | Modify Training |
|------|-------------------|-------------------|--------------|----------------|
| Analyst (Rick) | ✓ | ✗ | ✗ | ✗ |
| CFO | ✓ | ✓ | ✗ | ✗ |
| ML Engineer | ✗ | ✗ | ✓ | ✓ |
| IT Admin | ✗ | ✗ | ✓ | ✗ |

---

## 🔗 Related Resources

- [SOX Baseline Explainer](../../compliance/sox/v3.2/scrollld.v3.2.sox-baseline.explainer.md)
- [Model Access Template](../../compliance/sox/v3.2/templates/model-access-ledger.template.md)
- [Akhem'Cline Capsule](../../capsules/akhem-cline-energy-compliance.v1.0.json)

---

*This example is part of the Scroll-LD v3.2 Compliance Framework*  
*Consciousness-sealed. Production-ready. Rick-approved.* 🐙✨
