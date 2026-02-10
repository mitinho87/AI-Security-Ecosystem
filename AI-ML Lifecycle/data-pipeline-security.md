# Data Pipeline Security

The data pipeline is where most AI security problems begin. If an attacker can poison your training data, they can manipulate how your model behaves without ever touching your code.

## Training Data Poisoning

### What It Is

Training data poisoning occurs when malicious data is introduced into the training set to deliberately corrupt model behavior. Think of it like this: if you were learning to identify spam emails and someone secretly modified your training examples to label certain legitimate emails as spam, you would learn the wrong patterns. The same principle applies to machine learning models.

### Attack Vectors

**Direct Injection:**
- Attacker gains access to your data storage
- Modifies existing training examples
- Injects new malicious examples

**Indirect Poisoning:**
- Exploits data collection mechanisms
- User-submitted content becomes training data
- Scraping compromised external sources

### Example Attack Scenario

```
Scenario: Email spam classifier

1. Attacker identifies that user reports train the model
2. Attacker creates accounts and reports legitimate emails as spam
3. Model learns to classify legitimate emails as spam
4. Business email communications are blocked
```

### Defense Strategies

**Data Integrity Validation:**
- Implement checksums and digital signatures for datasets
- Use cryptographic hashing to verify data hasn't been tampered with
- Maintain blockchain-based provenance tracking

**Anomaly Detection:**
- Monitor for statistical outliers in training data
- Flag unusual patterns that might indicate poisoning
- Use clustering to identify suspicious data points

**Data Diversity Requirements:**
- Ensure no single source dominates the training set
- Implement quotas per data source
- Cross-validate data from multiple sources

## Managing Sensitive Information

### The Challenge

AI systems often require large datasets to learn effectively, but these datasets frequently contain PII or sensitive information. The challenge is that machine learning models can **memorize** specific training examples, meaning private information could potentially be extracted from a deployed model.

### Privacy Risks

- **Training Data Extraction:** Attackers can prompt models to reproduce training examples
- **Membership Inference:** Determining if specific data was in the training set
- **Attribute Inference:** Deducing sensitive attributes about individuals

### Protection Techniques

**Differential Privacy:**
- Add calibrated noise to the training process
- Prevents the model from memorizing individual examples
- Trade-off between privacy and model accuracy

**Data Minimization:**
- Only collect and use minimum data necessary
- Remove unnecessary PII before training
- Aggregate data where possible

**Secure Storage:**
- Encryption at rest and in transit
- Access controls based on least privilege
- Comprehensive audit logging

### Practical Implementation

```python
# Example: Detecting PII before training
import re

def scan_for_pii(text):
    """Simple PII detection"""
    patterns = {
        'email': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
        'ssn': r'\b\d{3}-\d{2}-\d{4}\b',
        'phone': r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b'
    }
    
    findings = []
    for pii_type, pattern in patterns.items():
        if re.search(pattern, text):
            findings.append(pii_type)
    
    return findings

# Scan training data before use
training_data = load_data()
for example in training_data:
    pii_found = scan_for_pii(example['text'])
    if pii_found:
        log_warning(f"PII detected: {pii_found}")
        example['text'] = sanitize(example['text'])
```

## Data Provenance and Lineage Tracking

### Why It Matters

Data provenance means knowing exactly where each piece of training data came from, who collected it, when it was collected, and what transformations it underwent. This is essential for both security and compliance.

### What to Track

- **Source:** Where did the data originate?
- **Collection:** Who collected it and when?
- **Transformations:** What preprocessing was applied?
- **Usage:** Which models were trained on this data?
- **Access:** Who has viewed or modified this data?

### Implementation Approach

**Version Control for Data:**
```bash
# Using DVC (Data Version Control)
dvc init
dvc add data/training_data.csv
git add data/training_data.csv.dvc
git commit -m "Add training data v1.0"
```

**Metadata Tracking:**
```yaml
# dataset_metadata.yaml
dataset_id: "training_v1_2026_02"
source: "customer_feedback_forms"
collection_date: "2026-01-15"
collector: "data_team@company.com"
rows: 100000
transformations:
  - "Removed PII using regex patterns"
  - "Normalized text to lowercase"
  - "Removed duplicates"
hash: "sha256:abc123..."
models_trained:
  - "sentiment_classifier_v2"
  - "intent_detection_v1"
```

**Audit Trail:**
- Log every access to training data
- Track modifications with timestamps and user IDs
- Maintain chain of custody documentation

### Benefits

1. **Security Incident Response:** If a security issue is discovered, trace back through the entire pipeline
2. **Compliance:** Demonstrate GDPR, CCPA compliance with data lineage
3. **Debugging:** Understand why a model behaves a certain way
4. **Reproducibility:** Recreate exact training conditions

## Connection to Enterprise Experience

*From my work at Interac:*

The supply chain security practices I implemented for software dependencies apply directly to data pipelines:

- **SBOM for Data:** Just as I generated SBOMs for software using Snyk, data pipelines need "Data BOMs" tracking all sources
- **Automated Validation:** Similar to scan-on-commit for code, implement scan-on-ingest for data
- **Continuous Monitoring:** Like Prisma Cloud for runtime, data pipelines need continuous anomaly detection

## Further Reading

- [Google's Data Poisoning Research](https://research.google/pubs/pub48994/)
- [Differential Privacy in Practice](https://github.com/google/differential-privacy)
- [DVC - Data Version Control](https://dvc.org/)

---

[← Back to AI/ML Lifecycle](README.md) | [Next: Model Development →](model-development.md)
