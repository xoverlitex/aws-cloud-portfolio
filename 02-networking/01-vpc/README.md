# VPC Networking Lab

## Objective

Build and validate a two-AZ AWS network that keeps workloads private, provides controlled administrative access, and permits temporary outbound internet access for testing.

## Design

| Component | Implementation |
| --- | --- |
| VPC | Private RFC 1918 address space split across two Availability Zones |
| Public tier | Two public subnets; a bastion host provides the administrative entry point |
| Private tier | Two application subnets; instances do not receive public IP addresses |
| Egress | NAT Gateways were used for validation, then deleted to avoid charges |
| Access control | Stateful Security Groups restrict SSH to the bastion-to-application path |
| Visibility | CloudTrail records API activity; VPC Flow Logs support network investigation |

```text
Administrator
    │ SSH
    ▼
Bastion host (public subnet)
    │ SSH allowed by application Security Group
    ▼
Private application instances
    │ outbound-only during testing
    ▼
NAT Gateway → Internet Gateway → Internet
```

## Validation

| Test | Method | Result |
| --- | --- | --- |
| Private egress | `curl -I https://aws.amazon.com` from a private instance | Confirmed HTTPS egress through NAT during testing |
| Route inspection | `ip route` | Confirmed the expected subnet gateway default route |
| East-west traffic | ICMP between private instances | Confirmed after correcting the relevant network/security configuration |
| Audit visibility | CloudTrail and VPC Flow Logs | Enabled and reviewed |

## Security decisions

- Private instances have no public IP addresses.
- The bastion host centralizes administrative SSH access.
- Application SSH is restricted to the bastion Security Group rather than the public internet.
- IAM learning material follows least-privilege principles and distinguishes trust policies from permission policies.
- An `ACCEPT` VPC Flow Log record indicates network-layer acceptance; it does not prove that an application accepted a connection or that a host was compromised.

## Cost management

NAT Gateways were deleted after testing because they incur hourly and data-processing charges. EC2 instances were stopped after validation. Check for unused Elastic IPs or other temporary resources before considering a lab complete.

## Evidence

Screenshots are intentionally not embedded until they have been reviewed for account IDs, resource IDs, public IP addresses, bucket names, and credentials. See the repository [evidence guide](../../docs/evidence.md) before adding them.

## Next steps

1. Add a practical EC2 IAM-role exercise with least-privilege policy and validation.
2. Rebuild the network with Terraform.
3. Add CloudWatch alarms, GuardDuty, AWS Config, and Security Hub.
4. Add sanitized architecture and validation screenshots.
