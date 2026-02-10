# OWASP LLM Top 10 Quick Reference

The OWASP Top 10 for Large Language Model Applications - essential security risks.

## The Top 10 (2025)

1. **LLM01: Prompt Injection** - Manipulating LLM behavior via crafted inputs
2. **LLM02: Insecure Output Handling** - Insufficient validation of LLM outputs
3. **LLM03: Training Data Poisoning** - Corrupting model behavior through malicious training data
4. **LLM04: Model Denial of Service** - Resource exhaustion through expensive operations
5. **LLM05: Supply Chain Vulnerabilities** - Risks from third-party components
6. **LLM06: Sensitive Information Disclosure** - Leakage of confidential data
7. **LLM07: Insecure Plugin Design** - Vulnerable LLM extensions
8. **LLM08: Excessive Agency** - LLM given too much autonomy
9. **LLM09: Overreliance** - Trusting LLM outputs without verification
10. **LLM10: Model Theft** - Unauthorized access to proprietary models

For detailed explanations, see:
- [Official OWASP LLM Project](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [Prompt Injection Deep Dive](../../docs/02-genai-llm-security/prompt-injection.md)
- [Data Poisoning](../../docs/01-ai-ml-lifecycle/data-pipeline-security.md)
