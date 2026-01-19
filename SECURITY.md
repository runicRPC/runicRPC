# Security Policy

## Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security issue in runicRPC, please report it responsibly.

### How to Report

**Do not create a public GitHub issue for security vulnerabilities.**

Instead, please use one of the following methods:

1. **GitHub Security Advisories** (preferred): Create a private security advisory on this repository
2. **Email**: Contact the maintainers directly through GitHub

Include the following information:

- Description of the vulnerability
- Steps to reproduce the issue
- Potential impact assessment
- Suggested remediation (if available)

### Response Timeline

| Stage | Timeline |
|-------|----------|
| Acknowledgment | Within 48 hours |
| Initial assessment | Within 5 business days |
| Resolution target (critical) | Within 7 days |
| Resolution target (high) | Within 14 days |
| Resolution target (medium/low) | Within 30 days |

### Disclosure Policy

- Security issues are addressed before public disclosure
- Security advisories are published after fixes are deployed
- Reporters receive credit in advisories upon request

---

## Security Best Practices

When integrating with runicRPC, follow these guidelines:

### API Key Management

| Practice | Description |
|----------|-------------|
| Environment variables | Store keys in environment variables, not source code |
| Secret management | Use a secrets manager (Vault, AWS Secrets Manager, etc.) for production |
| Key rotation | Rotate API keys periodically and after personnel changes |
| Separation | Use distinct keys for development, staging, and production |
| Least privilege | Request only necessary permissions from RPC providers |

### Network Security

- Use HTTPS/WSS endpoints exclusively
- Verify TLS certificate chains
- Consider network-level access controls for production deployments
- Monitor outbound connections to RPC providers

### Rate Limiting

- Configure rate limits appropriate to your provider quotas
- Monitor rate limit errors to detect misconfigurations
- Implement backpressure in your application layer

### Monitoring and Alerting

- Enable structured logging in production environments
- Monitor circuit breaker state transitions
- Track error rates and latency percentiles
- Configure alerts for anomalous patterns

---

## Security Considerations

### API Key Handling

runicRPC automatically masks API keys in logs and metrics output. However:

- Keys may appear in stack traces during initialization failures
- Ensure your logging infrastructure does not capture environment variables
- Review error reporting services for potential key exposure

### WebSocket Connections

- WebSocket connections maintain persistent state
- Reconnection behavior may be observable to network monitors
- Subscription data is transmitted without additional encryption beyond TLS

### Circuit Breaker State

- Circuit breaker transitions are observable through events and metrics
- State changes may reveal information about provider availability
- Consider this in threat models for sensitive applications

---

## Supported Versions

Security updates are provided for the current major version. Prior versions receive critical security fixes for 6 months after a new major release.

---

## Contact

For security concerns, use GitHub Security Advisories or contact maintainers through GitHub.

For general inquiries, open an issue on this repository.
