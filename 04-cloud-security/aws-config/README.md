# AWS Config: Compliance and Remediation

## Goal

Validate that a Security Group complies with a restricted-SSH policy, then correct the configuration and confirm the resource returns to a compliant state.

## Work completed

1. Enabled AWS Config in `us-east-1`.
2. Evaluated a `restricted-ssh` rule against the Security Group configuration.
3. Observed a noncompliant result, showing that the rule was actively evaluating the resource.
4. Corrected the applicable SSH exposure.
5. Rechecked the AWS Config dashboard and confirmed there were no noncompliant rules or resources; four resources were reported compliant.

![AWS Config after remediation](screenshots/aws-config-compliance-remediated.jpg)

## Security lesson

Security Groups should permit SSH only from a justified administrative source, such as a bastion host or tightly controlled IP range. AWS Config turns that expectation into a continuously evaluated control instead of a one-time manual review.

## Portfolio outcome

This exercise demonstrates a full control loop: detect configuration drift, remediate the exposure, and verify the compliant result.
