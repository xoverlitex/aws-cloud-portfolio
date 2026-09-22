# Evidence guide

Screenshots make a cloud portfolio credible, but AWS console views often contain identifiers that should not be published. Add only reviewed, redacted evidence.

## Recommended layout

```text
02-networking/01-vpc/
└── screenshots/
    ├── architecture-overview.png
    ├── route-tables.png
    ├── security-groups.png
    ├── private-connectivity.png
    └── flow-logs-configuration.png
```

Use lowercase kebab-case file names. Link each image from the relevant module section and include a one-sentence caption describing what it proves.

## Redact before committing

Remove or obscure:

- AWS account IDs, user names, email addresses, and S3 bucket names
- resource IDs, ENI IDs, VPC IDs, Elastic IPs, hostnames, and public IP addresses
- access keys, secret keys, session tokens, passwords, SSH keys, or QR codes
- raw CloudTrail and VPC Flow Log exports
- browser tabs, notifications, or other unrelated personal information

When in doubt, leave the image out. A short redacted screenshot with a clear caption is more valuable than a dense console capture.
