# Security Policy

PermitOps OS is a public, sanitized product architecture/demo repository.

## Public-repo rules

- Do not commit API keys, OAuth tokens, app passwords, private keys, `.env` files, client exports, applicant data, case files, real phone numbers, real personal email addresses, or private client notes.
- Use synthetic examples only.
- Keep production credentials in the appropriate secret manager or local keychain, never in Git.
- Pull requests and pushes are scanned with Gitleaks.

## Reporting a vulnerability

Open a private GitHub security advisory or contact the repository owner through GitHub.
