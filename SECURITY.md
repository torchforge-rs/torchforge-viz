# Security Policy

## Supported Versions

We support only the latest `v0.x` release series. Backports are not provided for older versions.

- **Latest version**: v0.x series
- **Security updates**: Applied only to the current release series
- **Older versions**: Not supported for security updates

## Reporting a Vulnerability

### Private Reporting

For security vulnerabilities, please **do not** open a public issue.

Instead, report vulnerabilities privately through:
- **GitHub Security Advisories**: Use the "Report a vulnerability" button on our [GitHub page](https://github.com/torchforge-rs/torchforge-viz/security/advisories)
- **Email**: security@torchforge-rs.org (for sensitive issues requiring encrypted communication) (coming soon)

### What to Include

When reporting a vulnerability, please include:
- **Description**: Clear description of the vulnerability
- **Impact**: Potential impact on users and systems
- **Reproduction**: Steps to reproduce the issue
- **Environment**: Version information and system details
- **Proof of Concept**: Code or examples demonstrating the vulnerability

## Response Timeline

Our security response process:

- **Acknowledgment**: Within 72 hours of receiving a report
- **Triage**: Within 7 days to assess severity and impact
- **Resolution**: As soon as possible based on severity
- **Disclosure**: Coordinated disclosure with reporter

### Severity Levels

- **Critical**: Immediate resolution required (within 7 days)
- **High**: Resolution in next release (within 30 days)
- **Medium**: Resolution in next scheduled release
- **Low**: Resolution in future release

## Security Scope

### In Scope

We consider the following security issues in scope:

- **Soundness issues**: Memory safety violations, undefined behavior
- **Event file corruption**: Incorrect CRC/framing producing silently corrupt event files
- **Supply chain issues**: Vulnerabilities in dependencies detected by `cargo audit`
- **Data exposure**: Unintended disclosure of training data or metrics
- **Code execution**: Arbitrary code execution through event file parsing

### Out of Scope

The following are considered out of scope:

- **TensorBoard bugs**: Issues in TensorBoard itself (report to TensorBoard project)
- **Theoretical-only issues**: Vulnerabilities without practical exploit paths
- **Denial of service**: Standard DoS attacks not specific to our implementation
- **User configuration**: Security issues resulting from user misconfiguration

## Security Best Practices

### For Users

- **Keep updated**: Always use the latest version
- **Verify integrity**: Check event files from untrusted sources
- **Limit access**: Restrict access to event files containing sensitive data
- **Monitor**: Watch for unusual behavior in training metrics

### For Developers

- **Input validation**: Validate all external inputs
- **Memory safety**: Use Rust's safety features and avoid unsafe code
- **Error handling**: Handle errors gracefully without exposing sensitive information
- **Dependencies**: Regularly audit dependencies for vulnerabilities

## Supply Chain Security

We maintain supply chain security through:

- **Dependency auditing**: Regular `cargo audit` checks
- **License compliance**: `cargo deny` for license policy enforcement
- **Code review**: All changes reviewed by maintainers
- **CI/CD security**: Secure build and release processes

## Security Advisories

When we fix a security vulnerability:

1. **Private fix**: Develop and test the fix privately
2. **Coordinated disclosure**: Work with reporter on disclosure timeline
3. **Security advisory**: Publish GitHub Security Advisory
4. **Security release**: Release fixed version
5. **Public disclosure**: Announce after grace period

## Security Team

The torchforge-rs security team:
- **Primary**: security@torchforge-rs.org
- **GitHub**: @torchforge-rs/security team
- **Response**: Security team members handle vulnerability reports

## Acknowledgments

We thank security researchers for:
- Responsible disclosure practices
- Detailed vulnerability reports
- Collaboration on fixes
- Patience during the disclosure process

## Security Questions

For security-related questions:
- **Vulnerability reports**: Use private reporting methods
- **Security best practices**: Open an issue with "security" label
- **General questions**: Use GitHub Discussions

## Related Resources

- [Rust Security Working Group](https://www.rust-lang.org/security)
- [GitHub Security Advisories](https://docs.github.com/en/code-security)
- [CVE Database](https://cve.mitre.org/)
- [OWASP Rust Security Guide](https://cheatsheetseries.owasp.org/cheatsheets/Rust_Security_Cheat_Sheet.html)
