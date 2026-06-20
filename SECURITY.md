# Security Policy

PermitOps OS is a public, sanitized product architecture/demo repository.

It is based on a real, active business/project, but the public repository contains only sample, synthetic, generalized, or sanitized material. Do not treat examples, screenshots, counts, names, records, case details, or workflows in this repo as private production data.

## Public-repo rules

- Do not commit API keys, OAuth tokens, app passwords, private keys, `.env` files, client exports, applicant data, case files, real phone numbers, real personal email addresses, or private client notes.
- Use synthetic or sanitized examples only, and clearly label them as sample material when they could be mistaken for real operational data.
- Keep production credentials in the appropriate secret manager or local keychain, never in Git.
- Pull requests and pushes are scanned with Gitleaks.

## Reporting a vulnerability

Open a private GitHub security advisory or contact the repository owner through GitHub.
