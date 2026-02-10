# Pre-Deployment Security Checklist for LLM Applications

Use this checklist before deploying any LLM application to production.

## 🔐 Authentication & Authorization

- [ ] **API Authentication implemented** (OAuth 2.0, JWT, or API keys)
- [ ] **Multi-factor authentication** for admin access
- [ ] **Role-based access control** (RBAC) configured
- [ ] **Principle of least privilege** applied to all users/services
- [ ] **Service accounts** use scoped, rotated credentials
- [ ] **Session management** properly configured (timeouts, secure cookies)

## 🛡️ Input Validation & Sanitization

- [ ] **Maximum input length** enforced (prevent DoS via long prompts)
- [ ] **Prompt injection detection** implemented
- [ ] **Encoding attack detection** (base64, special characters)
- [ ] **File upload validation** (if applicable)
- [ ] **Input sanitization** for special tokens and delimiters
- [ ] **Rate limiting** per user/IP configured

## 📤 Output Security

- [ ] **PII detection and redaction** implemented
- [ ] **Sensitive data filtering** (API keys, credentials, internal URLs)
- [ ] **Output length limits** configured
- [ ] **Response validation** to detect unexpected behavior
- [ ] **Citation/source validation** (for RAG systems)
- [ ] **Content Security Policy** headers set

## 🗄️ Data Security

- [ ] **Training data does not contain PII** or sensitive information
- [ ] **Data encryption at rest** for stored data
- [ ] **Data encryption in transit** (TLS 1.3+)
- [ ] **Access logs** for all data access
- [ ] **Data retention policies** defined and enforced
- [ ] **Right to deletion** processes implemented (GDPR compliance)

## 📚 RAG-Specific Security (if applicable)

- [ ] **Document access controls** enforced at retrieval layer
- [ ] **Cross-user data leakage prevention** verified
- [ ] **Document poisoning protection** implemented
- [ ] **Retrieved content sanitization** before LLM processing
- [ ] **Vector database authentication** configured
- [ ] **Embedding model integrity** verified

## 🤖 Model Security

- [ ] **Model provenance** documented (source, training data, versions)
- [ ] **Model integrity verification** (checksums, signatures)
- [ ] **Dependency vulnerability scanning** completed (Snyk, Safety)
- [ ] **SBOM generated** for model and dependencies
- [ ] **Model version control** implemented
- [ ] **Rollback procedure** documented and tested

## 🔍 Monitoring & Logging

- [ ] **Security event logging** configured
- [ ] **Anomaly detection** for unusual patterns
- [ ] **Model drift monitoring** implemented
- [ ] **Performance metrics** tracked (latency, error rates)
- [ ] **Alert thresholds** configured
- [ ] **Log retention** complies with requirements
- [ ] **SIEM integration** (if applicable)

## 🚨 Incident Response

- [ ] **Incident response plan** documented
- [ ] **Security contacts** defined
- [ ] **Escalation procedures** established
- [ ] **Model kill switch** implemented
- [ ] **Backup/rollback procedures** tested
- [ ] **Post-incident review** process defined

## ⚖️ Compliance & Governance

- [ ] **Privacy impact assessment** completed
- [ ] **Terms of service** define AI usage
- [ ] **Data processing agreements** in place (if applicable)
- [ ] **Regulatory compliance** verified (GDPR, CCPA, HIPAA, etc.)
- [ ] **Ethical AI guidelines** followed
- [ ] **Bias testing** performed

## 🧪 Security Testing

- [ ] **Penetration testing** completed
- [ ] **Prompt injection testing** performed
- [ ] **Jailbreak testing** performed
- [ ] **Data extraction testing** performed
- [ ] **API security testing** (OWASP API Top 10)
- [ ] **Load testing** under adversarial conditions
- [ ] **Failure mode testing** completed

## 📝 Documentation

- [ ] **System architecture** documented
- [ ] **Security controls** documented
- [ ] **Known limitations** documented
- [ ] **User guidelines** provided
- [ ] **Incident runbooks** created
- [ ] **Change management** process defined

## 🔄 Continuous Security

- [ ] **Automated security scanning** in CI/CD pipeline
- [ ] **Dependency updates** automated and monitored
- [ ] **Security patch process** defined
- [ ] **Regular security reviews** scheduled
- [ ] **Threat model** maintained and updated
- [ ] **Security training** for team completed

---

## Risk Assessment

### Critical (Must Fix Before Deployment)
- Authentication/authorization failures
- Sensitive data exposure
- Prompt injection without defenses
- RAG cross-user data leakage

### High (Should Fix)
- Missing monitoring/alerting
- Inadequate input validation
- No incident response plan
- Compliance gaps

### Medium (Fix Soon)
- Documentation gaps
- Lack of automated testing
- Insufficient logging

---

## Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Security Lead | | | |
| Engineering Lead | | | |
| Product Owner | | | |
| Compliance Officer | | | |

---

*Based on OWASP LLM Top 10, NIST AI RMF, and industry best practices*
