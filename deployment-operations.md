# Deployment and Operations Security

Once a model is trained and validated, it needs to be deployed to production where it will serve predictions. This deployment phase introduces runtime security concerns analogous to deploying traditional applications but with AI-specific considerations.

## Model Serving Security

### API Security Fundamentals

When you expose an AI model through an API, you're creating an attack surface that needs the same protections as any web application.

**Authentication:**
```python
# Example: Securing model API with JWT
from flask import Flask, request, jsonify
from functools import wraps
import jwt

app = Flask(__name__)

def require_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token:
            return jsonify({'error': 'No token provided'}), 401
        
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
            request.user = payload
        except jwt.InvalidTokenError:
            return jsonify({'error': 'Invalid token'}), 401
        
        return f(*args, **kwargs)
    return decorated

@app.route('/predict', methods=['POST'])
@require_auth
def predict():
    data = request.json
    prediction = model.predict(data)
    return jsonify({'prediction': prediction})
```

**Rate Limiting:**
```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["100 per hour"]
)

@app.route('/predict', methods=['POST'])
@limiter.limit("10 per minute")
@require_auth
def predict():
    # Rate limited to prevent abuse
    pass
```

**Input Validation:**
```python
def validate_input(data):
    """Validate model input before processing"""
    # Check data type
    if not isinstance(data, dict):
        raise ValueError("Input must be JSON object")
    
    # Check required fields
    required_fields = ['text', 'user_id']
    for field in required_fields:
        if field not in data:
            raise ValueError(f"Missing required field: {field}")
    
    # Check data size (prevent DoS)
    text = data['text']
    if len(text) > 10000:
        raise ValueError("Text exceeds maximum length")
    
    # Check for prompt injection patterns
    suspicious_patterns = [
        'ignore previous instructions',
        'disregard all above',
        'system prompt override'
    ]
    for pattern in suspicious_patterns:
        if pattern.lower() in text.lower():
            log_security_event(f"Suspicious input detected: {pattern}")
            raise ValueError("Input rejected by security filter")
    
    return True
```

### AI-Specific API Considerations

Beyond traditional web security, LLM APIs face unique attacks:

**Adversarial Examples:**
- Crafted inputs designed to manipulate predictions
- Requires input validation and anomaly detection

**Data Extraction:**
- Attackers probing to extract training data
- Implement output filtering to catch leaked information

**Model Extraction:**
- Repeated queries to replicate your model
- Use rate limiting and query pattern detection

## Monitoring for Model Drift and Anomalies

### What is Model Drift?

AI models can degrade over time as real-world data drifts away from training distribution. This is both a performance issue and a security concern.

**Types of Drift:**

1. **Data Drift:** Input distributions change
2. **Concept Drift:** Relationships between inputs and outputs change
3. **Adversarial Drift:** Attacker deliberately triggers drift

### Monitoring Strategy

```python
import numpy as np
from scipy import stats

class ModelMonitor:
    def __init__(self, baseline_data):
        """Initialize with baseline statistics from training data"""
        self.baseline_mean = np.mean(baseline_data, axis=0)
        self.baseline_std = np.std(baseline_data, axis=0)
    
    def check_drift(self, new_data):
        """Detect statistical drift in incoming data"""
        new_mean = np.mean(new_data, axis=0)
        new_std = np.std(new_data, axis=0)
        
        # KS test for distribution change
        ks_statistic, p_value = stats.ks_2samp(
            self.baseline_data.flatten(),
            new_data.flatten()
        )
        
        if p_value < 0.05:
            alert("Significant data drift detected", {
                'ks_statistic': ks_statistic,
                'p_value': p_value
            })
        
        return p_value
    
    def check_anomalies(self, prediction_data):
        """Detect unusual prediction patterns"""
        # Check for sudden changes in prediction distribution
        recent_predictions = prediction_data[-1000:]
        
        # Alert if predictions become too uniform (possible attack)
        unique_ratio = len(np.unique(recent_predictions)) / len(recent_predictions)
        if unique_ratio < 0.1:
            alert("Unusually uniform predictions detected")
        
        # Alert if prediction confidence drops
        avg_confidence = np.mean([p['confidence'] for p in recent_predictions])
        if avg_confidence < 0.5:
            alert("Average prediction confidence dropped below threshold")
```

### Metrics to Monitor

**Model Performance:**
- Accuracy, precision, recall over time
- Prediction latency
- Error rates by input type

**Input Distribution:**
- Feature statistics (mean, std, quantiles)
- New/unseen values
- Encoding patterns (base64, special chars)

**Output Distribution:**
- Prediction confidence scores
- Output diversity
- Rate of refusals/errors

**Security Indicators:**
- Failed authentication attempts
- Rate limit violations
- Suspicious input patterns
- Unusual query sequences

## Secure MLOps Pipelines

### CI/CD for ML

Just as you automate security testing for code, automate it for models:

```yaml
# .github/workflows/model-security.yml
name: Model Security Checks

on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Scan dependencies
        run: |
          pip install safety
          safety check -r requirements.txt
      
      - name: Validate model inputs
        run: |
          python scripts/test_input_validation.py
      
      - name: Check for data leakage
        run: |
          python scripts/test_output_pii_detection.py
      
      - name: Test adversarial robustness
        run: |
          python scripts/test_adversarial_examples.py
      
      - name: Generate security report
        run: |
          python scripts/generate_security_report.py
```

### Deployment Gates

```python
def can_deploy_model(model, test_data):
    """Security gates before production deployment"""
    checks = {
        'accuracy_threshold': model.evaluate(test_data)['accuracy'] > 0.90,
        'no_pii_leakage': not detects_pii_in_outputs(model, test_data),
        'robustness': passes_adversarial_tests(model),
        'dependencies_clean': no_vulnerable_dependencies(),
        'model_signed': verify_model_signature(model)
    }
    
    for check, passed in checks.items():
        if not passed:
            log_error(f"Deployment gate failed: {check}")
            return False
    
    return True
```

## Connection to Enterprise Experience

*From my experience at Interac:*

**Runtime Monitoring:**
- Deployed Prisma Cloud for workload protection → Apply same monitoring to model serving
- Runtime anomaly detection for containers → Anomaly detection for predictions
- Alerting on unusual behavior → Alert on model drift or attack patterns

**CI/CD Automation:**
- Integrated Snyk into GitHub pipelines → Integrate model security tests
- Scan-on-commit reduced MTTD → Scan-on-train reduces model vulnerabilities
- Automated deployment gates → Automated model validation gates

## Best Practices Checklist

- [ ] Model APIs require authentication (OAuth 2.0, JWT)
- [ ] Rate limiting implemented per user/IP
- [ ] Input validation for size, format, content
- [ ] Output filtering to prevent data leakage
- [ ] Model drift monitoring configured
- [ ] Baseline metrics established
- [ ] Alerting on anomalies configured
- [ ] CI/CD pipeline includes security tests
- [ ] Deployment gates validate security
- [ ] Incident response plan documented

## Further Reading

- [MLOps Principles](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
- [Evidently AI - Drift Detection](https://www.evidentlyai.com/)
- [API Security Best Practices](https://owasp.org/www-project-api-security/)

---

[← Back: Model Development](model-development.md) | [Next Section: GenAI Security →](../02-genai-llm-security/README.md)
