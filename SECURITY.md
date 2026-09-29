# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in Simba, please report it responsibly. **Do not create a public GitHub issue for security vulnerabilities.**

### How to Report

Email **info@pymc-labs.com** with:

- A description of the vulnerability
- Steps to reproduce the issue
- The potential impact
- Any suggested remediation (if applicable)

### Response Timeline

- **Acknowledgement**: Within 48 hours of receiving your report
- **Initial assessment**: Within 5 business days
- **Resolution**: Depending on severity, typically within 30 days

### What to Expect

- We will acknowledge your report and keep you informed of our progress
- We will not take legal action against researchers who report vulnerabilities responsibly
- We will credit you in our security acknowledgements (unless you prefer to remain anonymous)

## Security Overview

Simba is Cyber Essentials certified and implements enterprise-grade security:

- **Encryption at rest**: AES-256 encryption for all stored data
- **Encryption in transit**: TLS 1.2 or later for all data transmission
- **Infrastructure**: data encrypted at rest and access-isolated per project, so a user only reaches the projects and models they own or that are shared with their team
- **Compliance**: Fully GDPR compliant with strict data minimization and zero-retention logging
- **Data sovereignty**: Standard Contractual Clauses for international data transfers

For full details, see our [Security Documentation](docs/security/README.md).
