# Security Policy

## Supported Versions

The following versions are currently supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| 2.x.x   | :white_check_mark: |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

> Notes:
> - The `main` branch is used for ongoing development toward upcoming releases.
> - Patch releases prioritize security fixes while preserving backward compatibility within the same supported version line.

## Security Guidelines

### 1. Dependency and Supply Chain Security
- Use dependencies only from trusted and official registries.
- Keep lockfiles committed and run vulnerability scans on a regular basis.
- Remove unused packages and unnecessary permissions.

### 2. Secret and Credential Management
- Never commit secrets such as API keys, tokens, passwords, certificates, or private keys.
- Inject sensitive values through environment variables or a dedicated secrets manager.
- Ensure logs and error messages do not expose sensitive data (mask when necessary).

### 3. Secure Development Practices
- Apply input validation and output encoding by default.
- Enforce authentication and authorization with the principle of least privilege.
- Keep security-relevant defaults in a secure-by-default state.

### 4. Reporting a Vulnerability
Please report vulnerabilities through a private channel, not through public issues.

- Contact: ``
- Initial response target: within 3 business days
- Please include:
  - Affected version(s) and environment
  - Reproduction steps or proof of concept
  - Potential impact and suggested mitigation (if available)

### 5. Disclosure Process
Our typical response process is:

1. Acknowledge and triage
2. Reproduce and assess impact
3. Develop and validate a fix
4. Prepare and release a patch
5. Publish a security advisory (as appropriate)

To protect users, detailed vulnerability information may be withheld until a fix is available.
