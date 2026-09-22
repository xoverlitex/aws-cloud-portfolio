# GuardDuty: Detection and Investigation

## Goal

Practice interpreting a GuardDuty finding without generating real malicious activity.

## Work completed

1. Enabled GuardDuty in `us-east-1`.
2. Generated AWS-provided sample findings.
3. Reviewed the Findings list and selected a sample `Recon:EC2/Portscan` finding.
4. Examined the finding details, including severity, direction, protocol, affected resource context, and evidence.

![GuardDuty sample findings](screenshots/guardduty-sample-findings.jpg)

![GuardDuty sample finding details](screenshots/guardduty-sample-finding-details.jpg)

## Finding interpretation

The selected `Recon:EC2/Portscan` sample finding is **simulated AWS test data**, not evidence that a real EC2 instance performed a port scan. Its purpose is to demonstrate the investigation workflow: review severity, identify the detection type, inspect the evidence, determine scope, and choose an appropriate response path.

## Security lesson

GuardDuty is a detective control. A finding should be triaged with context rather than treated as proof of compromise. For a real finding, the next steps would be to validate the affected resource, review CloudTrail and VPC Flow Logs, contain risk where appropriate, and document the investigation.

## Next step

Integrate GuardDuty findings with EventBridge and SNS to demonstrate alert routing without generating real attack traffic.
