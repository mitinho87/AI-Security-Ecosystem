# Model Development and Training Security

The model development phase is where data scientists experiment with different architectures, hyperparameters, and training techniques. This phase introduces security considerations that parallel traditional software development but require AI-specific approaches.

## Securing Training Infrastructure

### The Challenge

Training large AI models requires significant computational resources, often running on cloud infrastructure with GPU clusters. These environments must be secured like any computing infrastructure, but with additional considerations:

- **Long Attack Windows:** Training jobs run for hours or days
- **High Resource Costs:** Prime targets for cryptojacking or resource theft  
- **Sensitive Data Access:** Training code often accesses confidential datasets
- **Valuable Artifacts:** Model weights represent significant intellectual property

### Security Measures

**Authentication & Authorization:**
```bash
# Example: Securing access to training cluster
- Implement SSO with MFA for cluster access
- Use service accounts with scoped permissions
- Rotate credentials regularly
- Implement just-in-time access for training jobs
```

**Network Segmentation:**
- Isolate training environments from production systems
- Use private VPCs or VNETs
- Implement firewall rules restricting outbound connections
- Monitor network traffic for anomalies

**Resource Quotas & Monitoring:**
```python
# Example: Detecting unauthorized resource usage
def monitor_training_jobs():
    """Alert on suspicious training patterns"""
    for job in get_active_jobs():
        # Check if job owner is authorized
        if not is_authorized_user(job.owner):
            alert("Unauthorized training job detected", job)
        
        # Check resource consumption
        if job.gpu_hours > THRESHOLD:
            alert("Excessive GPU usage", job)
        
        # Check for unusual patterns
        if job.start_time.hour not in BUSINESS_HOURS:
            alert("Off-hours training job", job)
```

**Secrets Management:**
- Never hardcode API keys or credentials in training code
- Use secrets management services (AWS Secrets Manager, HashiCorp Vault)
- Rotate secrets automatically
- Audit secret access

## Version Control for Models

### Why It's Complex

ML version control is more complex than code version control because you need to track:

- **Code:** Training scripts, model definitions
- **Data:** Training datasets (often gigabytes)
- **Model Weights:** Trained model artifacts (can be very large)
- **Hyperparameters:** Configuration used for training
- **Metrics:** Performance measurements
- **Environment:** Framework versions, dependencies

### Tools & Approaches

**MLflow for Experiment Tracking:**
```python
import mlflow

# Track experiments
with mlflow.start_run():
    # Log parameters
    mlflow.log_param("learning_rate", 0.001)
    mlflow.log_param("batch_size", 32)
    
    # Train model
    model = train_model(data, lr=0.001, batch_size=32)
    
    # Log metrics
    mlflow.log_metric("accuracy", 0.95)
    mlflow.log_metric("loss", 0.05)
    
    # Log model with signature
    mlflow.sklearn.log_model(model, "model")
```

**DVC for Data Versioning:**
```bash
# Track large datasets
dvc add data/train.csv
dvc add data/validation.csv

# Push to remote storage
dvc push

# Version control the .dvc files (small)
git add data/train.csv.dvc data/validation.csv.dvc
git commit -m "Add training data v2.0"
```

### Security Benefits

1. **Tamper Detection:** Cryptographic hashes detect unauthorized modifications
2. **Audit Trail:** Complete history of who trained what, when
3. **Incident Response:** Trace back from vulnerable model to source data/code
4. **Reproducibility:** Recreate exact model for forensic analysis

## Supply Chain Security for ML Frameworks

### The Risk

Modern ML relies heavily on open-source frameworks:
- PyTorch, TensorFlow (deep learning frameworks)
- Hugging Face Transformers (pre-trained models)
- NumPy, Pandas, Scikit-learn (data processing)
- Hundreds of supporting libraries

Each dependency is a potential attack vector.

### Attack Scenarios

**Compromised Package:**
```python
# Malicious code in popular ML library
import torch  # Attacker compromised PyPI package

# Backdoor executes during import
# Steals model weights, training data, or credentials
```

**Model Poisoning via Pre-trained Weights:**
```python
# Downloading pre-trained model from untrusted source
from transformers import AutoModel

# Model may contain backdoors or trojans
model = AutoModel.from_pretrained("untrusted-source/bert-base")
```

### Defense Strategies

**Dependency Scanning:**
```bash
# Scan ML dependencies for vulnerabilities
pip install safety
safety check

# Or use Snyk
snyk test

# Generate SBOM
pip install cyclonedx-bom
cyclonedx-py -o sbom.json
```

**Pin Dependency Versions:**
```txt
# requirements.txt - Never use 'latest'
torch==2.1.0
transformers==4.35.0
numpy==1.24.3
```

**Verify Package Integrity:**
```python
import hashlib

def verify_package_hash(package_path, expected_hash):
    """Verify downloaded package matches expected hash"""
    sha256 = hashlib.sha256()
    with open(package_path, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b""):
            sha256.update(chunk)
    
    if sha256.hexdigest() != expected_hash:
        raise SecurityError("Package hash mismatch!")
```

**Use Private Package Repositories:**
- Mirror approved packages to internal repository
- Scan packages before adding to mirror
- Enforce use of internal mirror only

## Connection to Enterprise Experience

*From my work at Interac implementing supply chain security:*

The same practices I used for traditional software apply to ML:

**SBOM Generation:**
- Generated SBOMs for software dependencies → Generate SBOMs for ML models
- Snyk scanning for vulnerabilities → Scan ML frameworks with same tools
- Tracked software provenance → Track model/data provenance

**CI/CD Integration:**
- Scan-on-commit for code → Scan-on-train for models
- Automated security gates → Automated model validation gates
- Reduced MTTD from weeks to hours → Apply same automation to ML pipelines

## Best Practices Checklist

- [ ] All training infrastructure requires MFA
- [ ] Training environments are network-isolated
- [ ] Resource quotas prevent abuse
- [ ] Secrets are managed, never hardcoded
- [ ] All experiments logged with MLflow/similar
- [ ] Data versioned with DVC/similar
- [ ] Model artifacts cryptographically signed
- [ ] Dependencies pinned to specific versions
- [ ] SBOM generated for all models
- [ ] Regular vulnerability scanning of ML frameworks
- [ ] Pre-trained models verified before use

## Further Reading

- [MLflow Documentation](https://mlflow.org/)
- [DVC - Data Version Control](https://dvc.org/)
- [OWASP ML Security Top 10](https://owasp.org/www-project-machine-learning-security-top-10/)
- [Google's Secure ML Infrastructure](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)

---

[← Back: Data Pipeline Security](data-pipeline-security.md) | [Next: Deployment & Operations →](deployment-operations.md)
