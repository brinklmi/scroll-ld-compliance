# Model Access Ledger Template

**Scroll Unit:** `sox.v3.2.ledger.model-access`  
**Purpose:** Track who accessed AI models, when, and for what purpose  
**Retention:** 7 years (SOX requirement)

---

## JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ModelAccessLog",
  "type": "object",
  "required": [
    "timestamp",
    "user_identity",
    "model_id",
    "purpose",
    "session_id"
  ],
  "properties": {
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp of access"
    },
    "user_identity": {
      "type": "string",
      "description": "Email or unique identifier of accessor"
    },
    "model_id": {
      "type": "string",
      "description": "Unique identifier for the AI model"
    },
    "model_version": {
      "type": "string",
      "description": "Version of model accessed"
    },
    "purpose": {
      "type": "string",
      "description": "Business purpose of model access"
    },
    "access_method": {
      "type": "string",
      "enum": ["API", "UI", "CLI", "SDK"],
      "description": "How model was accessed"
    },
    "session_id": {
      "type": "string",
      "description": "Unique session identifier for linking related logs"
    },
    "ip_address": {
      "type": "string",
      "description": "IP address of accessor (optional)"
    },
    "department": {
      "type": "string",
      "description": "Department of user (optional)"
    },
    "financial_context": {
      "type": "boolean",
      "description": "Whether this access affects financial reporting"
    }
  }
}
```

---

## Example Log Entry

```json
{
  "timestamp": "2026-01-26T14:00:00Z",
  "user_identity": "rick@company.com",
  "model_id": "gpt-4-financial-commentary-v2",
  "model_version": "2.1.0",
  "purpose": "Generate Q4 2025 revenue commentary",
  "access_method": "API",
  "session_id": "sess_abc123xyz",
  "ip_address": "10.0.1.42",
  "department": "Finance",
  "financial_context": true
}
```

---

## Implementation Checklist

- [ ] Set up logging infrastructure (database or file system)
- [ ] Configure 7-year retention policy
- [ ] Integrate with authentication system to capture user_identity
- [ ] Generate unique session_id for each access
- [ ] Log before model invocation (not after)
- [ ] Ensure logs are immutable once written
- [ ] Set up backup and disaster recovery
- [ ] Configure access controls (only compliance team can read)
- [ ] Create search/query interface for audits

---

## Storage Recommendations

**Database Option:**
- PostgreSQL with time-series extension
- Indexed on timestamp, user_identity, model_id
- Partitioned by month for performance

**File Option:**
- JSON Lines format (.jsonl)
- One file per day: `model_access_2026-01-26.jsonl`
- Store in immutable S3 bucket with versioning

---

## Query Examples

### Find all accesses by specific user
```sql
SELECT * FROM model_access_log
WHERE user_identity = 'rick@company.com'
AND timestamp >= '2025-01-01'
ORDER BY timestamp DESC;
```

### Find all financial-context accesses
```sql
SELECT * FROM model_access_log
WHERE financial_context = true
AND timestamp >= '2025-01-01';
```

### Audit trail for specific model
```sql
SELECT user_identity, purpose, timestamp
FROM model_access_log
WHERE model_id = 'gpt-4-financial-commentary-v2'
ORDER BY timestamp DESC;
```

---

## Integration Example (Python)

```python
import json
from datetime import datetime
from uuid import uuid4

class ModelAccessLogger:
    def __init__(self, log_file_path):
        self.log_file = log_file_path
    
    def log_access(self, user_identity, model_id, purpose, 
                   access_method="API", model_version=None):
        log_entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "user_identity": user_identity,
            "model_id": model_id,
            "model_version": model_version,
            "purpose": purpose,
            "access_method": access_method,
            "session_id": f"sess_{uuid4().hex[:12]}",
            "financial_context": self.is_financial_model(model_id)
        }
        
        with open(self.log_file, 'a') as f:
            f.write(json.dumps(log_entry) + '\n')
        
        return log_entry["session_id"]
    
    def is_financial_model(self, model_id):
        # Implement logic to determine if model affects financials
        return "financial" in model_id.lower()

# Usage
logger = ModelAccessLogger("model_access.jsonl")
session_id = logger.log_access(
    user_identity="rick@company.com",
    model_id="gpt-4-financial-commentary-v2",
    purpose="Generate Q4 revenue commentary"
)
```

---

## Compliance Notes

- **SOX 404**: Internal controls over financial reporting
- **SOX 302**: Officer certification of financial statements
- **7-Year Retention**: Minimum retention period for audit trails
- **Access Control**: Logs must be protected from tampering
- **Audit Readiness**: Logs must be searchable within 24 hours

---

*This template is part of the Scroll-LD v3.2 SOX Compliance Framework*
