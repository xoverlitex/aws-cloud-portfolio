# Portfolio hygiene

This is a public portfolio repository. Do not commit live operational data or credentials.

## Never commit

- AWS access keys, secret access keys, session tokens, `.env` files, or private SSH keys
- Terraform state, plans containing secrets, or variable files with real values
- raw VPC Flow Logs, CloudTrail exports, account identifiers, or unredacted console screenshots
- live public IP addresses, DNS names, bucket names, or resource identifiers unless there is a deliberate and reviewed reason

## Before pushing

1. Review `git diff --cached` for identifiers and secrets.
2. Confirm screenshots are redacted and named consistently.
3. Keep test output short and anonymized.
4. Use example values in documentation and policies.
